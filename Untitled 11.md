```
dùng agent_team thay cho agent_pool dùng agent_name thay cho agent_type

Và vẽ full kiến trúc (agent_team: automation_team, simple_team), agent_name: drive_agent, notion_agent, ..., simple_agent có math_agent, date_agent (call đến MCP: ...)

Các agent này call đến MCP

-- vẫn giữ kiến trúc message bus

```


### Architecture Pattern được Khuyến Nghị

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR AGENT PLATFORM                      │
│                                                             │
│  ┌──────────────┐         ┌─────────────────┐             │
│  │  FastAPI     │────────▶│  Session Store  │             │
│  │  Backend     │◀────────│  (Redis/DB)     │             │
│  └──────────────┘         └─────────────────┘             │
│         │                          │                        │
│         │                          │ user_id → token       │
│         ▼                          ▼                        │
│  ┌──────────────────────────────────────────┐             │
│  │     MCP Server Manager                   │             │
│  │  - Spawn per-user MCP servers            │             │
│  │  - Track active servers                  │             │
│  │  - Cleanup on logout/timeout             │             │
│  └──────────────────────────────────────────┘             │
│         │                                                   │
└─────────┼───────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────┐
│              PER-USER MCP SERVERS (Isolated)                │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐               │
│  │ MCP Server      │    │ MCP Server      │               │
│  │ (User A)        │    │ (User B)        │               │
│  │                 │    │                 │               │
│  │ OAuth Token A   │    │ OAuth Token B   │               │
│  │ Port: 8001      │    │ Port: 8002      │               │
│  └─────────────────┘    └─────────────────┘               │
│         │                       │                          │
│         ▼                       ▼                          │
│  ┌─────────────────┐    ┌─────────────────┐               │
│  │ Google Drive A  │    │ Google Drive B  │               │
│  └─────────────────┘    └─────────────────┘               │
└─────────────────────────────────────────────────────────────┘
```

## 2. Implementation Chi Tiết


```bash

1. Orchestrator (Entry Point)
   ├─ Khởi tạo MessageBus + tất cả agents
   ├─ Init MCP adapter (nếu có)
   └─ Start run_loop cho tất cả agents

2. Planner Agent (Planning)
   ├─ Subscribe "task_available"
   ├─ Phân tích query (LLM hoặc heuristic)
   ├─ Tạo plan: list[PlanStep] với agent_pool, agent_type, arguments
   └─ Publish trực tiếp từng step đến special agents:
      ├─ "math_task" (nếu có math step)
      ├─ "date_task" (nếu có date step)
      ├─ "web_browsertask" (nếu có youtube step)
      └─ "{pool}_task" (cho các pool khác)

3. Agent Teams (Execution)
   ├─ Math Team: DemoMathAgentSum, DemoMathAgentSubtract
   │  ├─ Subscribe "math_task"
   │  ├─ Match agent_type (sum/subtract)
   │  ├─ Call MCP tool (mcp_sum/mcp_subtract)
   │  └─ Publish "math_result" → MessageBus
   │
   ├─ Date Team: DateAgentToday
   │  ├─ Subscribe "date_task"
   │  └─ Publish "date_result" → MessageBus
   │
   └─ Automation Team: SheetAgent/DriveAgent/NotionAgent...
      ├─ Subscribe ""
      └─ Publish "" → MessageBus

4. Tester Agent (Aggregation)
   ├─ Subscribe: "", "", ""
   ├─ Collect tất cả results
   ├─ Generate final report (format markdown)
   └─ Publish "final_report" → MessageBus



```



```mermaid
graph TB
    %% Layer 1: Orchestrator
    O[Orchestrator]
    
    %% Layer 2: Message Bus
    MB[Message Bus<br/>Redis/In-Memory]
    
    %% Layer 3: Planner
    P[Planner]
    
    %% Layer 4: Agent Teams
    subgraph AT[" "]
        direction LR
        DA[drive_agent]
        NA[notion_agent]
    end
    
    subgraph ST[" "]
        direction LR
        MA[math_agent]
        DTA[date_agent]
    end
    
    %% Layer 5: MCP Servers
    subgraph MCP[" "]
        direction LR
        DriveMCP[drive_mcp]
        NotionMCP[notion_mcp]
        MathMCP[math_mcp]
        DateMCP[date_mcp]
    end
    
    %% Layer 6: Synthesizer
    S[Synthesizer]
    
    %% Flow từ trên xuống dưới
    O -->|task_available| MB
    MB -->|task_available| P
    P -->|plan_ready| MB
    
    MB -->|automation_team_task| DA
    MB -->|automation_team_task| NA
    MB -->|simple_team_task| MA
    MB -->|simple_team_task| DTA
    
    DA -->|call_tool| DriveMCP
    NA -->|call_tool| NotionMCP
    MA -->|call_tool| MathMCP
    DTA -->|call_tool| DateMCP
    
    DA -->|automation_result| MB
    NA -->|automation_result| MB
    MA -->|simple_result| MB
    DTA -->|simple_result| MB
    
    MB -->|automation_result| S
    MB -->|simple_result| S
    S -->|final_report| O
    
    %% Styling
    classDef orchestratorClass fill:#2c5282,stroke:#1a365d,color:#fff
    classDef busClass fill:#2d3748,stroke:#1a202c,color:#fff
    classDef agentClass fill:#234e52,stroke:#1a365d,color:#fff
    classDef mcpClass fill:#553c9a,stroke:#322659,color:#fff
    classDef synthClass fill:#742a2a,stroke:#5a1e1e,color:#fff
    
    class O orchestratorClass
    class MB busClass
    class P,DA,NA,MA,DTA agentClass
    class DriveMCP,NotionMCP,MathMCP,DateMCP mcpClass
    class S synthClass


```



# Hướng Dẫn Chi Tiết: Triển Khai Hệ Thống AI Agent Multi-Tenant với Drive MCP

Dựa trên nghiên cứu từ các nguồn chuyên sâu, đây là architecture guide đầy đủ để xây dựng hệ thống cho phép nhiều người dùng truy cập Drive riêng của họ thông qua AI agent.

## 1. Tổng Quan Kiến Trúc

### Architecture Pattern được Khuyến Nghị

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR AGENT PLATFORM                      │
│                                                             │
│  ┌──────────────┐         ┌─────────────────┐             │
│  │  FastAPI     │────────▶│  Session Store  │             │
│  │  Backend     │◀────────│  (Redis/DB)     │             │
│  └──────────────┘         └─────────────────┘             │
│         │                          │                        │
│         │                          │ user_id → token       │
│         ▼                          ▼                        │
│  ┌──────────────────────────────────────────┐             │
│  │     MCP Server Manager                   │             │
│  │  - Spawn per-user MCP servers            │             │
│  │  - Track active servers                  │             │
│  │  - Cleanup on logout/timeout             │             │
│  └──────────────────────────────────────────┘             │
│         │                                                   │
└─────────┼───────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────┐
│              PER-USER MCP SERVERS (Isolated)                │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐               │
│  │ MCP Server      │    │ MCP Server      │               │
│  │ (User A)        │    │ (User B)        │               │
│  │                 │    │                 │               │
│  │ OAuth Token A   │    │ OAuth Token B   │               │
│  │ Port: 8001      │    │ Port: 8002      │               │
│  └─────────────────┘    └─────────────────┘               │
│         │                       │                          │
│         ▼                       ▼                          │
│  ┌─────────────────┐    ┌─────────────────┐               │
│  │ Google Drive A  │    │ Google Drive B  │               │
│  └─────────────────┘    └─────────────────┘               │
└─────────────────────────────────────────────────────────────┘
```


---


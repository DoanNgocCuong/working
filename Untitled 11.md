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
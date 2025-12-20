# SDD - HLD P3: High Architecture, Folder Structure & Data Flow - Updated

**Version:** 3.0
**Date:** December 2025
**Status:** ✅ Updated với MCP Integration

---

## 📋 Mục Lục

1. [Tổng Quan](#tổng-quan)
2. [High Level Architecture](#high-level-architecture)
3. [Folder Structure Chi Tiết](#folder-structure-chi-tiết)
4. [Data Flow Diagrams](#data-flow-diagrams)
5. [MCP Integration](#mcp-integration)
6. [Component Interactions](#component-interactions)
7. [So Sánh với P2](#so-sánh-với-p2)
8. [Migration Notes](#migration-notes)

---

## Tổng Quan

### Thay Đổi Chính từ P2

**SDD P2** đã chuyển từ **ReAct Pattern (LangGraph Sequential)** sang **Multi-Agent Pattern (MessageBus)**.

**SDD P3** bổ sung:

- ✅ **MCP (Model Context Protocol) Integration** - Chuẩn hóa tool interface
- ✅ **Folder Structure đầy đủ** - Bao gồm MCP servers và adapters
- ✅ **Data Flow chi tiết** - Từ input đến output với MCP flow
- ✅ **Component Interactions** - Cách các components tương tác

### Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│ Layer 0: Governance (Input/Output Gates, Guards)       │
├─────────────────────────────────────────────────────────┤
│ Layer 1: Perception (Query Processing, Entity Extract) │
├─────────────────────────────────────────────────────────┤
│ Layer 2: Cognition (Multi-Agent System + MCP)          │
│  ├── Agents (Chief, Planner, Navigator, Extractor...)  │
│  ├── MessageBus (Redis PubSub)                         │
│  └── MCPToolAdapter → ToolRegistry → MCP Servers       │
├─────────────────────────────────────────────────────────┤
│ Layer 3: Action (Browser Tools, Data Extractors)       │
│  ├── PlaywrightController                              │
│  ├── FinancialDataExtractor                            │
│  └── MCP Servers (BrowserMCPServer, FinancialMCPServer)│
└─────────────────────────────────────────────────────────┘
```

---

## High Level Architecture

### 1. Architecture Layers

#### Layer 0: Governance

- **Input Gate**: Rate limiting, safety checks, validation
- **In-Flight Guards**: Browser guards, navigation loop detection
- **Output Gate**: Result validation, quality checks

#### Layer 1: Perception

- **Input Processor**: Text normalization, classification
- **Entity Extractor**: LLM-based entity extraction
- **Context Builder**: User history, market context

#### Layer 2: Cognition (Multi-Agent + MCP)

**Multi-Agent System:**

```
ChiefAgent (Coordinator)
    ↓ broadcast("task_available")
PlannerAgent
    ↓ publish("plan_ready")
NavigatorAgent ──┐
    ↓ publish("page_ready")  │
ExtractorAgent ──┼──→ MCPToolAdapter
    ↓ publish("data_extracted") │
VerifierAgent ───┘      ↓
    ↓ publish("verification_done") ToolRegistry
SynthesizerAgent              ↓
    ↓ publish("final_report") MCP Servers
                                ↓
                         Underlying Tools
```

**MCP Integration:**

- **MCPToolAdapter**: Bridge giữa agents và MCP servers
- **ToolRegistry**: Centralized tool discovery và management
- **MCPServers**: BrowserMCPServer, FinancialMCPServer (TODO)

#### Layer 3: Action

- **Browser Tools**: PlaywrightController, BrowserPool
- **Extractors**: FinancialDataExtractor
- **MCP Servers**: Wrap underlying tools với MCP interface

---

## Folder Structure Chi Tiết

### Complete Folder Tree

```
app/module/finai_agent/
├── __init__.py                          # Module exports
├── agent_entrypoint.py                  # Main entry point (legacy LangGraph)
├── IMPLEMENTATION_STATUS.md             # Implementation status doc
│
├── core/                                # Core Abstractions
│   ├── __init__.py
│   ├── agent_config.py                  # AgentConfig
│   ├── base_extractor.py                # BaseDataExtractor
│   ├── config.py                        # Settings
│   ├── constants.py                     # Constants
│   ├── exceptions.py                    # Custom exceptions
│   └── schemas/
│       └── memory.py                    # ConversationMemory
│
├── interfaces/                          # Interfaces (SOLID)
│   ├── __init__.py
│   ├── financial_interface.py          # IFinancialExtractor
│   └── generic_browser_interface.py    # IBrowserController
│
├── layer_0_governance/                  # Governance Layer
│   ├── __init__.py
│   ├── phase_1_input_gate/
│   │   ├── __init__.py
│   │   └── input_gate.py               # FinancialInputGate
│   ├── phase_2_in_flight_guards/
│   │   ├── __init__.py
│   │   └── browser_guards.py           # BrowserGuards
│   └── phase_3_output_gate/
│       ├── __init__.py
│       └── result_validator.py         # FinancialOutputValidator
│
├── layer_1_perception/                  # Perception Layer
│   ├── __init__.py
│   ├── input_processor.py              # FinancialQueryProcessor
│   ├── entity_extractor.py             # FinancialEntityExtractor
│   ├── context_builder.py              # FinancialContextBuilder
│   ├── query_processor.py              # QueryProcessor
│   ├── models.py                       # FinancialQuery model
│   └── phase_1_input_processing/       # Stage 1 Pipeline
│       ├── __init__.py
│       ├── env.py                      # buildEnv
│       ├── envelope.py                 # initEnvelope
│       ├── classifier.py               # runInputClassifier
│       ├── normalizer.py               # runTextNormalizer
│       ├── page_context.py             # attachPageContext
│       ├── context_manager.py          # fetchConversationMemory
│       └── telemetry.py                # buildTelemetry
│
├── layer_2_cognition/                   # ⭐ Cognition Layer (Multi-Agent + MCP)
│   ├── __init__.py
│   │
│   ├── message_bus.py                  # MessageBus (Redis PubSub)
│   ├── base_agent.py                   # P2PAgent base class
│   ├── multi_agent_orchestrator.py     # Orchestrator cho multi-agent
│   │
│   ├── agent_chat_run.py               # LLM helper với Langfuse
│   ├── agent_prompt_config.py          # Prompt config từ Langfuse
│   ├── agent_prompt_service.py         # Prompt service
│   │
│   ├── mcp_tool_adapter.py             # ⭐ NEW: MCPToolAdapter
│   ├── mcp_init.py                     # ⭐ NEW: init_mcp()
│   │
│   ├── state.py                        # FinAIState (legacy LangGraph)
│   ├── graph.py                        # build_finai_graph() (legacy)
│   │
│   ├── chief_agent/                    # Chief Agent
│   │   ├── __init__.py
│   │   └── chief.py                    # ChiefAgent
│   │
│   ├── planner_agent/                  # Planner Agent
│   │   ├── __init__.py
│   │   └── planner.py                  # PlannerAgent
│   │
│   ├── navigator_agent/                # Navigator Agent
│   │   ├── __init__.py
│   │   └── navigator.py                # NavigatorAgent (MCP support)
│   │
│   ├── extractor_agent/                # Extractor Agent
│   │   ├── __init__.py
│   │   └── extractor.py                # ExtractorAgent
│   │
│   ├── verifier_agent/                 # Verifier Agent
│   │   ├── __init__.py
│   │   └── verifier.py                 # VerifierAgent
│   │
│   ├── synthesizer_agent/              # Synthesizer Agent
│   │   ├── __init__.py
│   │   └── synthesizer.py              # SynthesizerAgent
│   │
│   └── nodes/                          # Legacy LangGraph nodes (deprecated)
│       ├── __init__.py
│       ├── perceive_node.py
│       ├── plan_navigation_node.py
│       ├── navigate_browser_node.py
│       ├── extract_data_node.py
│       ├── verify_data_node.py
│       ├── renavigate_node.py
│       └── synthesize_node.py
│
├── layer_3_action/                      # ⭐ Action Layer (với MCP)
│   ├── __init__.py
│   │
│   ├── tool_registry.py                # ⭐ Extended: MCP-aware ToolRegistry
│   │
│   ├── browser/                        # Browser Tools
│   │   ├── __init__.py
│   │   ├── playwright_controller.py   # PlaywrightController
│   │   ├── browser_pool.py            # BrowserPool
│   │   └── mock_browser_controller.py # Mock for testing
│   │
│   ├── tools/                          # Legacy Tools (vẫn dùng được)
│   │   ├── __init__.py
│   │   ├── base_tool.py                # BaseTool
│   │   ├── browser_tools.py            # NavigateTool, ClickTool, etc.
│   │   └── financial_data_extractor.py # FinancialDataExtractor
│   │
│   └── mcp_servers/                    # ⭐ NEW: MCP Servers
│       ├── __init__.py
│       ├── mcp_server.py               # MCPServer base class
│       ├── browser_mcp.py              # BrowserMCPServer
│       └── finance_mcp.py              # FinancialMCPServer (TODO)
│
├── repositories/                        # Data Layer
│   ├── __init__.py
│   └── query_repository.py             # QueryRepository
│
└── migrations/                          # Database Migrations
    └── 001_create_tables.sql           # Database schema
```

### Key Changes from P2

#### ✅ Added (MCP Integration)

1. **`layer_2_cognition/mcp_tool_adapter.py`**

   - MCPToolAdapter class
   - MCPToolRequest/Response models
2. **`layer_2_cognition/mcp_init.py`**

   - `init_mcp()` function
   - MCP infrastructure initialization
3. **`layer_3_action/mcp_servers/`** (NEW FOLDER)

   - `mcp_server.py` - Base MCPServer class
   - `browser_mcp.py` - BrowserMCPServer implementation
4. **`layer_3_action/tool_registry.py`** (EXTENDED)

   - MCP server registration methods
   - Tool discovery và validation

#### ✅ Modified

1. **`layer_2_cognition/navigator_agent/navigator.py`**

   - Added `mcp_adapter` parameter (optional)
   - Support cả MCP mode và legacy mode
2. **`layer_2_cognition/multi_agent_orchestrator.py`**

   - Calls `init_mcp()` during setup
   - Passes `mcp_adapter` to NavigatorAgent

#### 🔄 Unchanged (Backward Compatible)

- All legacy code still works
- LangGraph nodes folder (deprecated but not removed)
- All Layer 0, Layer 1, Layer 3 underlying tools

---

## Data Flow Diagrams

### 1. End-to-End Data Flow (with MCP)

```
[User Request]
    ↓
┌─────────────────────────────────────────────────┐
│ Layer 0: Input Gate                            │
│  - Rate limiting                                │
│  - Safety checks                                │
│  - Validation                                   │
└──────────────────┬──────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────┐
│ Layer 1: Perception                             │
│  - Input Processing                             │
│  - Entity Extraction                            │
│  - Context Building                             │
│  → FinancialQuery + Context                     │
└──────────────────┬──────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────┐
│ Layer 2: Multi-Agent Orchestration              │
│                                                  │
│  ChiefAgent                                     │
│    ↓ broadcast("task_available")                │
│  PlannerAgent                                   │
│    ↓ publish("plan_ready")                      │
│  NavigatorAgent                                 │
│    │  ┌──────────────────────┐                  │
│    ├─→│ MCPToolAdapter       │                  │
│    │  │  - validate          │                  │
│    │  │  - publish events    │                  │
│    │  └──┬───────────────────┘                  │
│    │     ↓                                      │
│    │  ToolRegistry.get_server()                 │
│    │     ↓                                      │
│    │  BrowserMCPServer.call_tool()              │
│    │     ↓                                      │
│    │  PlaywrightController.navigate()           │
│    ↓ publish("page_ready")                      │
│  ExtractorAgent                                 │
│    ↓ publish("data_extracted")                  │
│  VerifierAgent                                  │
│    ↓ publish("verification_done")               │
│  SynthesizerAgent                               │
│    ↓ publish("final_report")                    │
└──────────────────┬──────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────┐
│ Layer 0: Output Gate                            │
│  - Result validation                            │
│  - Quality checks                               │
└──────────────────┬──────────────────────────────┘
                   ↓
[Final Response to User]
```

### 2. MCP Tool Call Flow (Detailed)

```
NavigatorAgent
    │
    │ req = MCPToolRequest(
    │     agent_name="FinAI_Navigator",
    │     tool_server="browser",
    │     tool_name="navigate",
    │     arguments={"url": "https://..."}
    │ )
    │
    ↓ await self.mcp.execute(req)
    │
┌─────────────────────────────────────────┐
│ MCPToolAdapter.execute()                │
│                                          │
│  1. Validate tool exists                │
│     registry.tool_exists("browser",     │
│                          "navigate")    │
│                                          │
│  2. Publish "tool_request" event        │
│     bus.publish("tool_request", {...})  │
│                                          │
│  3. Get server                           │
│     server = registry.get_server(       │
│                 "browser")               │
│                                          │
│  4. Call tool                            │
│     result = await server.call_tool(    │
│                  "navigate", args)       │
│                                          │
│  5. Publish "tool_response" event       │
│     bus.publish("tool_response", {...}) │
│                                          │
│  6. Return MCPToolResponse               │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│ BrowserMCPServer.call_tool()            │
│                                          │
│  if tool_name == "navigate":            │
│      await controller.navigate(url)     │
│      return {"status": "success", ...}  │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│ PlaywrightController.navigate()         │
│                                          │
│  await self.page.goto(url, ...)         │
│  # Actual browser operation             │
└─────────────────────────────────────────┘
```

### 3. Multi-Agent Message Flow

```
┌─────────────────────────────────────────────────────────┐
│                    MessageBus (Redis PubSub)            │
│                                                          │
│  Topics:                                                 │
│  - task_available                                        │
│  - plan_ready                                            │
│  - page_ready                                            │
│  - data_extracted                                        │
│  - verification_done                                     │
│  - renavigate_request                                    │
│  - final_report                                          │
│  - tool_request (MCP)                                    │
│  - tool_response (MCP)                                   │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ↓                 ↓                 ↓
   ChiefAgent      PlannerAgent     NavigatorAgent
        │                 │                 │
        │                 │                 │
        │                 ↓                 │
        │          publish("plan_ready")    │
        │                 │                 │
        │                 └─────→ subscribe │
        │                                 │
        │                                 ↓
        │                          execute_mcp()
        │                                 │
        │                                 ↓
        │                          publish("page_ready")
        │                                 │
        │                                 │
        ↓                                 │
   ExtractorAgent ←─── subscribe ────────┘
        │
        ↓
   publish("data_extracted")
        │
        ↓
   VerifierAgent (subscribe)
        │
        ↓
   publish("verification_done")
        │
        ↓
   SynthesizerAgent (subscribe)
        │
        ↓
   publish("final_report")
        │
        ↓
   ChiefAgent (subscribe)
        │
        ↓
   Return to orchestrator
```

---

## MCP Integration

### 1. MCP Architecture Components

#### MCPToolAdapter

**Location:** `layer_2_cognition/mcp_tool_adapter.py`

**Responsibilities:**

- Validate tool requests
- Execute tools via ToolRegistry
- Publish events to MessageBus
- Error handling và logging
- Execution time tracking

**Key Methods:**

```python
async def execute(req: MCPToolRequest) -> MCPToolResponse:
    # 1. Validate tool exists
    # 2. Get server từ registry
    # 3. Publish tool_request event
    # 4. Call server.call_tool()
    # 5. Publish tool_response event
    # 6. Return response
```

#### ToolRegistry (MCP-aware)

**Location:** `layer_3_action/tool_registry.py`

**New Methods:**

- `register_mcp_server(name, server)`
- `refresh_tools(server_name)`
- `tool_exists(server_name, tool_name)`
- `get_server(server_name)`
- `list_mcp_servers()`
- `list_mcp_tools(server_name)`

#### MCPServer (Base Class)

**Location:** `layer_3_action/mcp_servers/mcp_server.py`

**Abstract Interface:**

```python
class MCPServer(ABC):
    @abstractmethod
    async def discover_tools() -> List[ToolDefinition]
  
    @abstractmethod
    def get_tool_definition(tool_name: str) -> Optional[ToolDefinition]
  
    @abstractmethod
    async def call_tool(tool_name: str, arguments: Dict) -> Any
```

#### BrowserMCPServer

**Location:** `layer_3_action/mcp_servers/browser_mcp.py`

**Tools Exposed:**

- `navigate` - Navigate to URL
- `get_html` - Get page HTML
- `screenshot` - Capture screenshot
- `get_current_url` - Get current URL
- `get_a11y_tree` - Get accessibility tree

**Wraps:** `PlaywrightController`

### 2. MCP Initialization

**Location:** `layer_2_cognition/mcp_init.py`

```python
async def init_mcp(bus: MessageBus) -> Optional[MCPToolAdapter]:
    # 1. Create ToolRegistry
    # 2. Initialize PlaywrightController
    # 3. Create BrowserMCPServer
    # 4. Register server
    # 5. Discover tools
    # 6. Create MCPToolAdapter
    # 7. Return adapter (or None if failed)
```

**Called in:** `multi_agent_orchestrator.py`

```python
mcp_adapter = await init_mcp(bus)
navigator = NavigatorAgent(bus=bus, mcp_adapter=mcp_adapter)
```

### 3. Agent Integration (Opt-in Pattern)

**NavigatorAgent** hỗ trợ cả MCP mode và legacy mode:

```python
class NavigatorAgent(P2PAgent):
    def __init__(self, bus: MessageBus, 
                 mcp_adapter: Optional[MCPToolAdapter] = None):
        self.mcp = mcp_adapter  # Optional
        self.browser = None     # Legacy fallback
  
    async def execute_navigation_plan(self, msg: Message):
        if self.mcp:
            # MCP mode
            req = MCPToolRequest(...)
            res = await self.mcp.execute(req)
        else:
            # Legacy mode
            await self.browser.navigate(url)
```

**Benefits:**

- ✅ Backward compatible
- ✅ Gradual migration
- ✅ Fallback to legacy if MCP fails

---

## Component Interactions

### 1. Agent-to-Tool Interaction (via MCP)

```
Agent
  │
  │ MCPToolRequest(
  │   tool_server="browser",
  │   tool_name="navigate",
  │   arguments={"url": "..."}
  │ )
  │
  ↓ await mcp_adapter.execute(req)
  │
MCPToolAdapter
  │
  │ 1. Validate
  │ 2. Get server từ registry
  │ 3. Publish tool_request
  │ 4. Call server.call_tool()
  │ 5. Publish tool_response
  │
  ↓
ToolRegistry
  │
  │ server = mcp_servers["browser"]
  │
  ↓
BrowserMCPServer
  │
  │ await controller.navigate(url)
  │
  ↓
PlaywrightController
  │
  │ await page.goto(url)
  │
  ↓
Browser (Playwright)
```

### 2. Agent-to-Agent Communication (via MessageBus)

```
Agent A
  │
  │ await self.send(
  │     to_agent="broadcast",
  │     topic="plan_ready",
  │     payload={...}
  │ )
  │
  ↓
MessageBus (Redis PubSub)
  │
  │ publish("plan_ready", payload)
  │
  ↓ (to all subscribers)
  │
Agent B (subscribed to "plan_ready")
  │
  │ async def on_plan_ready(msg: Message):
  │     await self.handle_plan(msg)
  │
  ↓
Agent B processes message
```

### 3. Orchestrator Flow

```
multi_agent_orchestrator.run_finai_multi_agent()
  │
  │ 1. Init MCP
  │    mcp_adapter = await init_mcp(bus)
  │
  │ 2. Create agents
  │    chief = ChiefAgent(bus)
  │    navigator = NavigatorAgent(bus, mcp_adapter)
  │    ...
  │
  │ 3. Start agent run_loops
  │    tasks = [agent.run_loop() for agent in agents]
  │
  │ 4. Broadcast task
  │    await chief.broadcast_task(...)
  │
  │ 5. Wait for final_report
  │    final_msg = await _wait_for_final_report()
  │
  │ 6. Stop agents
  │    for agent in agents: agent.stop()
  │
  │ 7. Return results
  ↓
Formatted response
```

---

## So Sánh với P2

### P2 Architecture (Multi-Agent Only)

```
Layer 2: Cognition
  ├── Agents (Chief, Planner, Navigator, ...)
  ├── MessageBus
  └── Direct Tool Calls
        ↓
  PlaywrightController (direct)
```

### P3 Architecture (Multi-Agent + MCP)

```
Layer 2: Cognition
  ├── Agents (Chief, Planner, Navigator, ...)
  ├── MessageBus
  └── MCPToolAdapter
        ↓
      ToolRegistry
        ↓
      MCP Servers
        ↓
  PlaywrightController (via MCP)
```

### Key Differences

| Aspect                    | P2               | P3                 |
| ------------------------- | ---------------- | ------------------ |
| **Tool Interface**  | Direct calls     | MCP (standardized) |
| **Tool Discovery**  | Hardcoded        | Dynamic discovery  |
| **Testability**     | Mock controllers | Mock MCP servers   |
| **Observability**   | Manual logging   | MessageBus events  |
| **Flexibility**     | Tight coupling   | Loose coupling     |
| **Backward Compat** | N/A              | ✅ Full support    |

---

## Migration Notes

### For Developers

#### Using MCP in New Code

```python
# 1. Get MCP adapter từ orchestrator
mcp_adapter = await init_mcp(bus)

# 2. Pass to agent
agent = NavigatorAgent(bus=bus, mcp_adapter=mcp_adapter)

# 3. Agent tự động dùng MCP nếu có adapter
```

#### Adding New MCP Server

```python
# 1. Create MCPServer subclass
class NewMCPServer(MCPServer):
    async def discover_tools(self) -> List[ToolDefinition]:
        return [ToolDefinition(...)]
  
    async def call_tool(self, tool_name: str, arguments: Dict) -> Any:
        # Implementation

# 2. Register trong init_mcp()
registry.register_mcp_server("new_server", NewMCPServer(...))
await registry.refresh_tools("new_server")
```

#### Using Legacy Mode

```python
# Simply don't pass mcp_adapter
agent = NavigatorAgent(bus=bus)  # Uses legacy mode
```

### Backward Compatibility

- ✅ All existing code continues to work
- ✅ Agents fallback to legacy if MCP not available
- ✅ No breaking changes
- ✅ Opt-in pattern (only use MCP when explicitly passed)

---

## Summary

### Architecture Evolution

1. **P1**: ReAct Pattern (LangGraph Sequential)
2. **P2**: Multi-Agent Pattern (MessageBus)
3. **P3**: Multi-Agent + MCP Integration ✅

### Key Features P3

- ✅ **MCP Integration**: Standardized tool interface
- ✅ **Backward Compatible**: Legacy code still works
- ✅ **Opt-in Pattern**: Agents use MCP only when adapter provided
- ✅ **Observability**: All tool calls via MessageBus
- ✅ **Testability**: Easy to mock MCP servers
- ✅ **Extensibility**: Easy to add new tools via MCP

### Folder Structure Highlights

- ✅ **MCP code isolated** trong `layer_3_action/mcp_servers/`
- ✅ **MCP adapters** trong `layer_2_cognition/`
- ✅ **Legacy code preserved** (nodes folder, direct tool calls)
- ✅ **Clear separation** of concerns

### Data Flow Highlights

- ✅ **End-to-end flow** từ input gate → agents → MCP → tools → output gate
- ✅ **Message-driven** communication between agents
- ✅ **Event-driven** tool calls via MessageBus
- ✅ **Standardized** tool interface via MCP

---

**Status**: ✅ **SDD P3 Complete** - Architecture Updated với MCP Integration

**Next Steps**:

- Implement FinancialMCPServer
- Add MCP support to ExtractorAgent
- Tool argument validation
- Integration tests

---

**References:**

- SDD P2: `SDD_P2_Update_ArchitectureMulAgents_for_Layer2.md`
- MCP Docs: `docs1.2_MCP_CONCEPT_AND_ARCHITECTURE.md`, `docs1.2_MCP_IMPLEMENTATION_GUIDE.md`
- Implementation Status: `app/module/finai_agent/IMPLEMENTATION_STATUS.md`
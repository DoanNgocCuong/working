# SDD Multi-Agent Coding System - UPDATE NOTES

## Tài Liệu Cập Nhật Để Mapping Với Source Code FinAI

**Version**: 1.0  
**Date**: 2025-12-17  
**Status**: Required Updates Identified

---

## 📋 TÓM TẮT THAY ĐỔI

| # | Hạng Mục | SDD Hiện Tại | Cần Update Thành | Priority |
|---|----------|--------------|------------------|----------|
| 1 | File Structure | `app/module/finai_agent/` | `app/module/agent/multi_agent_coding/` | P0 |
| 2 | Base Class | Custom `P2PAgent` | Extend `BaseAgent` + Mixin | P0 |
| 3 | Agent Registration | Không có | Dùng `@agent` decorator | P0 |
| 4 | Dependency Injection | Không có | Dùng `DependencyResolver` | P0 |
| 5 | State Model | Custom state | Extend `BaseState` | P0 |
| 6 | Kafka Integration | Manual | Tự động từ `BaseAgent` | P1 |
| 7 | LangFuse Tracing | Custom setup | Đã có sẵn trong `BaseAgent` | P1 |
| 8 | Auto-discovery | Không đề cập | Factory auto-discover | P1 |
| 9 | Message Bus | In-memory only | Tích hợp `graph_pub_sub.py` | P1 |
| 10 | Input Validation | Custom | Dùng Pydantic + `input_model` | P2 |

---

## 1. CẬP NHẬT FILE STRUCTURE

### ❌ SDD Hiện Tại

```
app/module/finai_agent/
├── layer_0_governance/
├── layer_1_perception/
├── layer_2_cognition/
│   ├── message_bus.py
│   ├── base_agent.py          # Custom P2PAgent
│   ├── chief_agent/
│   ├── coder_agent/
│   └── ...
└── layer_3_action/
```

### ✅ Nên Update Thành

```
app/module/agent/multi_agent_coding/
├── __init__.py
├── agent.py                    # Main entry + @agent decorator (REQUIRED for auto-discovery)
├── models.py                   # Input models + State models
├── config.py                   # Configuration
│
├── message_bus/
│   ├── __init__.py
│   ├── bus.py                  # MessageBus class (extend from graph_pub_sub pattern)
│   ├── topics.py               # MessageTopic enum
│   └── models.py               # Message dataclass
│
├── agents/                     # Individual agents (NOT separate folders)
│   ├── __init__.py
│   ├── base_p2p_agent.py       # P2PMixin class
│   ├── chief.py
│   ├── coder.py
│   ├── tester.py
│   └── reviewer.py
│
├── tools/
│   ├── __init__.py
│   ├── file_tools.py
│   └── execution_tools.py
│
├── governance/                 # Optional: Keep governance layer
│   ├── input_gate.py
│   └── output_gate.py
│
└── tests/
    ├── __init__.py
    ├── conftest.py
    └── test_multi_agent.py
```

### 📝 Lý Do

- `AgentFactory` auto-discover từ `app.module.agent` package
- Cần có file `agent.py` để trigger registration
- Flatten structure để dễ maintain

---

## 2. CẬP NHẬT BASE AGENT PATTERN

### ❌ SDD Hiện Tại - Custom P2PAgent

```python
# SDD định nghĩa class mới hoàn toàn
class P2PAgent(ABC):
    def __init__(self, name: str, subscribed_topics: List[str], bus: MessageBus = None):
        self.name = name
        self.inbox = asyncio.Queue()
        self.bus = bus or get_message_bus()
        # ...
```

### ✅ Nên Update - Extend BaseAgent + Mixin

```python
# app/module/agent/multi_agent_coding/agents/base_p2p_agent.py

from typing import List, Dict, Any
from abc import abstractmethod
import asyncio

from app.common.agent.base import BaseAgent
from app.common.agent.models import BaseState
from app.common.log import setup_logger

from ..message_bus.bus import MessageBus, get_message_bus
from ..message_bus.models import Message

logger = setup_logger(__name__)


class P2PMixin:
    """
    Mixin để thêm P2P communication capabilities vào BaseAgent.
    
    Mixin này KHÔNG replace BaseAgent mà BỔ SUNG thêm:
    - Subscribe/Publish qua Message Bus
    - Autonomous run loop
    - P2P messaging
    
    Usage:
        class MyAgent(P2PMixin, BaseAgent):
            subscribed_topics = ["task_available", "fix_request"]
            # ...
    """
    
    # Class attributes - override trong subclass
    subscribed_topics: List[str] = []
    agent_name: str = "unnamed"
    
    def __init__(self, **kwargs):
        # Call BaseAgent.__init__ first
        super().__init__(**kwargs)
        
        # P2P specific initialization
        self.inbox: asyncio.Queue = asyncio.Queue()
        self._bus: MessageBus = None
        self._running = False
        
    @property
    def bus(self) -> MessageBus:
        """Lazy load message bus."""
        if self._bus is None:
            self._bus = get_message_bus()
            # Subscribe to topics
            for topic in self.subscribed_topics:
                self._bus.subscribe(topic, self._on_message)
        return self._bus
        
    async def _on_message(self, msg: Message):
        """Callback khi nhận message từ bus."""
        if msg.to_agent in ["broadcast", self.agent_name]:
            await self.inbox.put(msg)
            logger.debug(f"[{self.agent_name}] 📬 Received: {msg.topic}")
            
    async def send_p2p(self, to_agent: str, topic: str, payload: Dict[str, Any]):
        """Send P2P message qua Message Bus."""
        msg = Message(
            from_agent=self.agent_name,
            to_agent=to_agent,
            topic=topic,
            payload=payload,
        )
        await self.bus.publish(msg)
        
    @abstractmethod
    async def decide_next_action(self, msg: Message) -> str:
        """Agent tự quyết định action tiếp theo."""
        pass
        
    @abstractmethod
    async def execute_action(self, action: str, msg: Message):
        """Execute action đã quyết định."""
        pass
        
    async def p2p_run_loop(self, timeout: float = 300):
        """
        Autonomous P2P run loop.
        
        Note: Đây là THÊM vào, không thay thế BaseAgent.run_async()
        """
        self._running = True
        logger.info(f"[{self.agent_name}] 🚀 P2P loop started")
        
        start_time = asyncio.get_event_loop().time()
        
        try:
            while self._running:
                if asyncio.get_event_loop().time() - start_time > timeout:
                    logger.warning(f"[{self.agent_name}] ⏰ Timeout")
                    break
                    
                try:
                    msg = await asyncio.wait_for(self.inbox.get(), timeout=5.0)
                    action = await self.decide_next_action(msg)
                    
                    if action == "STOP":
                        break
                        
                    await self.execute_action(action, msg)
                    
                except asyncio.TimeoutError:
                    continue
                    
        finally:
            self._running = False
            logger.info(f"[{self.agent_name}] Stopped")
            
    def stop_p2p(self):
        """Signal agent to stop P2P loop."""
        self._running = False
```

### 📝 Key Changes

| Aspect | SDD Hiện Tại | Sau Update |
|--------|--------------|------------|
| Inheritance | `P2PAgent(ABC)` | `P2PMixin + BaseAgent` |
| LangGraph | Không có | Kế thừa từ `BaseAgent.graph` |
| Kafka | Không có | Kế thừa từ `BaseAgent.send_kafka_one()` |
| Tracing | Không có | Kế thừa `@observe` decorator |

---

## 3. CẬP NHẬT AGENT REGISTRATION

### ❌ SDD Hiện Tại - Không Có Registration

```python
# SDD không sử dụng registry
class ChiefAgent(P2PAgent):
    def __init__(self, llm=None, **kwargs):
        super().__init__(
            name="Chief",
            subscribed_topics=["code_ready", "test_result", "review_done"],
            **kwargs
        )
```

### ✅ Nên Update - Dùng @agent Decorator

```python
# app/module/agent/multi_agent_coding/agent.py
"""
Multi-Agent Coding System - Main Entry Point.

File này PHẢI có để AgentFactory auto-discover.
"""

from pydantic import BaseModel, Field
from typing import Optional, List, Literal

from app.common.agent.base import BaseAgent
from app.common.agent.decorators import agent
from app.common.agent.models import BaseState
from app.common.log import setup_logger

from .agents.chief import ChiefAgentLogic
from .agents.base_p2p_agent import P2PMixin
from .message_bus.bus import get_message_bus

logger = setup_logger(__name__)


# ══════════════════════════════════════════════════════════════
# INPUT MODELS (API Input)
# ══════════════════════════════════════════════════════════════

class MultiAgentInput(BaseModel):
    """Input model cho API endpoint."""
    goal: str = Field(..., min_length=5, max_length=1000, description="Task goal")
    repo_path: str = Field(default=".", description="Repository path")
    timeout: int = Field(default=120, ge=10, le=600, description="Timeout in seconds")
    

# ══════════════════════════════════════════════════════════════
# STATE MODEL (Internal State)
# ══════════════════════════════════════════════════════════════

class MultiAgentState(BaseState):
    """State model cho LangGraph."""
    # Input
    goal: str = ""
    repo_path: str = "."
    timeout: int = 120
    
    # Progress
    current_phase: str = "init"
    agents_status: dict = {}
    messages_processed: int = 0
    
    # Results
    final_report: Optional[str] = None
    error: Optional[str] = None


def convert_input_to_state(input_model: MultiAgentInput) -> MultiAgentState:
    """Convert API input to internal state."""
    return MultiAgentState(
        goal=input_model.goal,
        repo_path=input_model.repo_path,
        timeout=input_model.timeout,
    )


# ══════════════════════════════════════════════════════════════
# MAIN AGENT (Registered with @agent decorator)
# ══════════════════════════════════════════════════════════════

@agent(
    agent_id="multi_agent_coding",
    name="Multi-Agent Coding System",
    description="Q4 Hierarchical Choreography system with Chief, Coder, Tester, Reviewer agents",
    input_model=MultiAgentInput,
    state_model=MultiAgentState,
    input_to_state=convert_input_to_state,
    kafka_topic="multi_agent_coding_events",
    dependency_specs={
        "kafka": "kafka_producer",      # Inject Kafka producer
        "redis": "redis_client",        # Inject Redis client
        # Thêm dependencies khác nếu cần
    }
)
class MultiAgentCodingSystem(P2PMixin, BaseAgent[MultiAgentState]):
    """
    Multi-Agent Coding System - Cursor Demo.
    
    Đây là ORCHESTRATOR agent, quản lý các sub-agents:
    - ChiefAgent
    - CoderAgent
    - TesterAgent
    - ReviewerAgent
    
    Flow:
    1. API call → MultiAgentCodingSystem.run_async()
    2. Spawn sub-agents
    3. Chief broadcasts task
    4. Sub-agents communicate via Message Bus
    5. Collect results → Return
    """
    
    agent_name = "Orchestrator"
    subscribed_topics = ["final_report"]  # Only listen for final results
    
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        
        # Build LangGraph for orchestration
        from langgraph.graph import StateGraph, START, END
        
        builder = StateGraph(MultiAgentState)
        
        # Nodes
        builder.add_node("spawn_agents", self._spawn_agents)
        builder.add_node("wait_for_completion", self._wait_for_completion)
        builder.add_node("collect_results", self._collect_results)
        
        # Edges
        builder.add_edge(START, "spawn_agents")
        builder.add_edge("spawn_agents", "wait_for_completion")
        builder.add_edge("wait_for_completion", "collect_results")
        builder.add_edge("collect_results", END)
        
        self.graph = builder.compile()
        
    async def _spawn_agents(self, state: MultiAgentState) -> dict:
        """Spawn all sub-agents."""
        logger.info("[Orchestrator] Spawning sub-agents...")
        
        # Import and create sub-agents
        from .agents.chief import ChiefAgent
        from .agents.coder import CoderAgent
        from .agents.tester import TesterAgent
        from .agents.reviewer import ReviewerAgent
        
        bus = get_message_bus()
        
        self.chief = ChiefAgent(bus=bus)
        self.coder = CoderAgent(bus=bus)
        self.tester = TesterAgent(bus=bus)
        self.reviewer = ReviewerAgent(bus=bus)
        
        return {"current_phase": "agents_spawned"}
        
    async def _wait_for_completion(self, state: MultiAgentState) -> dict:
        """Run agents and wait for completion."""
        import asyncio
        
        # Start all agent loops
        tasks = [
            asyncio.create_task(self.chief.p2p_run_loop(state.timeout)),
            asyncio.create_task(self.coder.p2p_run_loop(state.timeout)),
            asyncio.create_task(self.tester.p2p_run_loop(state.timeout)),
            asyncio.create_task(self.reviewer.p2p_run_loop(state.timeout)),
        ]
        
        # Wait a bit for agents to initialize
        await asyncio.sleep(1)
        
        # Chief kicks off task
        await self.chief.broadcast_task(state.goal, state.repo_path)
        
        # Wait for completion
        try:
            await asyncio.wait(tasks, timeout=state.timeout)
        except asyncio.TimeoutError:
            logger.warning("[Orchestrator] Timeout reached")
            
        # Stop all agents
        self.chief.stop_p2p()
        self.coder.stop_p2p()
        self.tester.stop_p2p()
        self.reviewer.stop_p2p()
        
        return {"current_phase": "completed"}
        
    async def _collect_results(self, state: MultiAgentState) -> dict:
        """Collect final results from message bus."""
        history = self.bus.get_history(topic="final_report")
        
        if history:
            final_msg = history[-1]
            return {
                "final_report": final_msg.payload.get("report", ""),
                "status": "success",
            }
        else:
            return {
                "final_report": "No report generated",
                "status": "error",
                "error": "Agents did not produce final report",
            }
```

### 📝 Key Points

1. **File `agent.py` là BẮT BUỘC** - Factory auto-discover từ đây
2. **Dùng `@agent` decorator** thay vì manual registration
3. **`input_model`** cho API validation
4. **`state_model`** extend từ `BaseState`
5. **`dependency_specs`** để inject dependencies

---

## 4. CẬP NHẬT SUB-AGENTS

### ❌ SDD Hiện Tại

```python
class CoderAgent(P2PAgent):
    def __init__(self, llm=None, **kwargs):
        super().__init__(
            name="Coder",
            subscribed_topics=["task_available", "fix_request"],
            **kwargs
        )
        self.llm = llm
```

### ✅ Nên Update

```python
# app/module/agent/multi_agent_coding/agents/coder.py

from typing import Dict, Any

from app.common.log import setup_logger
from .base_p2p_agent import P2PMixin
from ..message_bus.models import Message
from ..tools.file_tools import read_file, write_file, list_files

logger = setup_logger(__name__)


class CoderAgent(P2PMixin):
    """
    Coder Agent - Chuyên viết và sửa code.
    
    Note: KHÔNG cần @agent decorator vì đây là sub-agent,
    được quản lý bởi MultiAgentCodingSystem.
    """
    
    agent_name = "Coder"
    subscribed_topics = ["task_available", "fix_request"]
    
    def __init__(self, bus=None, llm=None, **kwargs):
        self._bus = bus
        self.llm = llm
        self.inbox = __import__('asyncio').Queue()
        self._running = False
        self.current_file = None
        
    async def decide_next_action(self, msg: Message) -> str:
        if msg.topic == "task_available":
            return "analyze_and_code"
        elif msg.topic == "fix_request":
            return "fix_bug"
        return "idle"
        
    async def execute_action(self, action: str, msg: Message):
        if action == "analyze_and_code":
            await self._analyze_and_code(msg)
        elif action == "fix_bug":
            await self._fix_bug(msg)
            
    async def _analyze_and_code(self, msg: Message):
        """Phân tích và sửa code."""
        repo_path = msg.payload.get("repo_path", ".")
        goal = msg.payload.get("goal", "")
        
        logger.info(f"[{self.agent_name}] 🔍 Analyzing: {goal}")
        
        files = list_files(repo_path)
        logger.info(f"[{self.agent_name}] Files: {files}")
        
        # Find and read main file
        target_file = f"{repo_path}/math_utils.py"
        try:
            code = read_file(target_file)
            self.current_file = target_file
            
            await self.send_p2p(
                to_agent="Tester",
                topic="code_ready",
                payload={"file": target_file, "code": code, "action": "initial_read"}
            )
        except FileNotFoundError:
            logger.error(f"[{self.agent_name}] File not found: {target_file}")
            
    async def _fix_bug(self, msg: Message):
        """Fix bug dựa trên error report."""
        error = msg.payload.get("error", "")
        logger.info(f"[{self.agent_name}] 🔧 Fixing: {error}")
        
        fixed_code = '''def add(a, b):
    """Add two numbers."""
    return a + b  # Fixed by CoderAgent
'''
        
        if self.current_file:
            write_file(self.current_file, fixed_code)
            logger.info(f"[{self.agent_name}] ✅ Fixed")
            
            await self.send_p2p(
                to_agent="Tester",
                topic="code_ready",
                payload={"file": self.current_file, "code": fixed_code, "action": "fix_applied"}
            )
```

---

## 5. CẬP NHẬT MESSAGE BUS

### ❌ SDD Hiện Tại

```python
# In-memory only
class MessageBus:
    def __init__(self):
        self.subscribers = {}
        self.message_history = []
```

### ✅ Nên Update - Tích Hợp Redis PubSub

```python
# app/module/agent/multi_agent_coding/message_bus/bus.py

"""
Message Bus với Redis PubSub support.

Tham khảo: app/common/redis/graph_pub_sub.py
"""

import asyncio
import json
from typing import Dict, List, Callable, Optional
from dataclasses import dataclass, field
from datetime import datetime

import redis.asyncio as redis

from app.common.log import setup_logger
from app.common.config import settings

logger = setup_logger(__name__)


@dataclass
class Message:
    """Message schema cho inter-agent communication."""
    from_agent: str
    to_agent: str
    topic: str
    payload: Dict
    timestamp: float = field(default_factory=lambda: datetime.now().timestamp())
    message_id: str = field(default_factory=lambda: __import__('uuid').uuid4().hex[:8])


class MessageBus:
    """
    Message Bus với dual-mode:
    - In-memory: Cho development/testing
    - Redis PubSub: Cho production (multi-worker support)
    """
    
    def __init__(self, redis_url: Optional[str] = None):
        self.subscribers: Dict[str, List[Callable]] = {}
        self.message_history: List[Message] = []
        self._lock = asyncio.Lock()
        
        # Redis support
        self.redis_url = redis_url or getattr(settings, 'REDIS_URL', None)
        self.redis_client: Optional[redis.Redis] = None
        self._pubsub = None
        self._listener_task = None
        
    async def connect_redis(self):
        """Connect to Redis for production mode."""
        if self.redis_url:
            self.redis_client = redis.from_url(self.redis_url, decode_responses=True)
            self._pubsub = self.redis_client.pubsub()
            logger.info("[MessageBus] Connected to Redis")
            
    async def disconnect_redis(self):
        """Disconnect from Redis."""
        if self._listener_task:
            self._listener_task.cancel()
        if self._pubsub:
            await self._pubsub.close()
        if self.redis_client:
            await self.redis_client.close()
            
    def subscribe(self, topic: str, callback: Callable):
        """Subscribe to topic."""
        if topic not in self.subscribers:
            self.subscribers[topic] = []
        self.subscribers[topic].append(callback)
        logger.debug(f"[MessageBus] Subscribed to '{topic}'")
        
    async def publish(self, msg: Message):
        """Publish message to all subscribers."""
        async with self._lock:
            self.message_history.append(msg)
            
        logger.info(f"[MessageBus] 📨 {msg.from_agent} → {msg.to_agent} | {msg.topic}")
        
        # Local subscribers
        tasks = []
        for callback in self.subscribers.get(msg.topic, []):
            tasks.append(asyncio.create_task(callback(msg)))
            
        if tasks:
            await asyncio.gather(*tasks, return_exceptions=True)
            
        # Redis publish (for multi-worker)
        if self.redis_client:
            await self.redis_client.publish(
                f"multi_agent:{msg.topic}",
                json.dumps({
                    "from_agent": msg.from_agent,
                    "to_agent": msg.to_agent,
                    "topic": msg.topic,
                    "payload": msg.payload,
                    "timestamp": msg.timestamp,
                    "message_id": msg.message_id,
                })
            )
            
    def get_history(self, topic: str = None, limit: int = 100) -> List[Message]:
        """Get message history."""
        if topic:
            return [m for m in self.message_history if m.topic == topic][-limit:]
        return self.message_history[-limit:]
        
    def clear_history(self):
        """Clear message history."""
        self.message_history.clear()


# Singleton instance
_bus_instance: Optional[MessageBus] = None


def get_message_bus() -> MessageBus:
    """Get or create MessageBus singleton."""
    global _bus_instance
    if _bus_instance is None:
        _bus_instance = MessageBus()
    return _bus_instance


def reset_message_bus():
    """Reset message bus (for testing)."""
    global _bus_instance
    if _bus_instance:
        _bus_instance.clear_history()
    _bus_instance = None
```

---

## 6. CẬP NHẬT API ENDPOINT

### ❌ SDD Hiện Tại

```python
@router.post("/agent/run")
async def run_agent(request: RunAgentRequest):
    result = await run_multi_agent_system(request.goal, request.repo_path)
    return result
```

### ✅ Nên Update - Dùng Factory Pattern

```python
# app/api/routes/multi_agent.py

from fastapi import APIRouter, Depends, HTTPException
from typing import Dict, Any

from app.common.agent.factory import AgentFactory
from app.common.dependency_resolver import DependencyResolver
from app.common.log import setup_logger

logger = setup_logger(__name__)

router = APIRouter(prefix="/multi-agent", tags=["Multi-Agent Coding"])


def get_agent_factory() -> AgentFactory:
    """Dependency injection for AgentFactory."""
    resolver = DependencyResolver()
    # Register dependencies
    # resolver.register("kafka_producer", singleton=kafka_producer)
    # resolver.register("redis_client", singleton=redis_client)
    return AgentFactory(resolver)


@router.post("/run")
async def run_multi_agent_system(
    goal: str,
    repo_path: str = ".",
    timeout: int = 120,
    factory: AgentFactory = Depends(get_agent_factory)
) -> Dict[str, Any]:
    """
    Chạy Multi-Agent Coding System.
    
    Flow:
    1. Factory.get_agent() → Get registered agent
    2. agent.run_async() → Execute LangGraph
    3. Return results
    """
    try:
        # Get agent from factory (auto-discovered and cached)
        agent = factory.get_agent("multi_agent_coding")
        
        # Get config for input conversion
        config = factory.get_agent_config("multi_agent_coding")
        
        # Create input model
        input_data = config.input_model(
            goal=goal,
            repo_path=repo_path,
            timeout=timeout
        )
        
        # Convert to state
        state = config.input_to_state(input_data)
        
        # Run agent
        result = await agent.run_async(state)
        
        return result
        
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
    except Exception as e:
        logger.error(f"Multi-agent system error: {e}")
        raise HTTPException(status_code=500, detail=str(e))
```

---

## 7. CẬP NHẬT DEPENDENCIES

### ❌ SDD Hiện Tại

```python
# Không đề cập dependency injection
class ChiefAgent:
    def __init__(self, llm=None):
        self.llm = llm
```

### ✅ Nên Update

```python
# Trong @agent decorator
@agent(
    agent_id="multi_agent_coding",
    # ...
    dependency_specs={
        "kafka": "kafka_producer",
        "redis": "redis_client",
        "llm": "openai_client",
        "db": "database_manager",
    }
)

# Dependencies được inject tự động qua **kwargs
class MultiAgentCodingSystem(BaseAgent):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        # self.kafka, self.redis, self.llm, self.db đã được set
```

### 📝 Cách Register Dependencies

```python
# app/common/lifespans.py hoặc app/container.py

from app.common.dependency_resolver import DependencyResolver

resolver = DependencyResolver()

# Register singletons
resolver.register("kafka_producer", singleton=kafka_producer_instance)
resolver.register("redis_client", singleton=redis_client_instance)

# Register factories (lazy creation)
resolver.register("database_manager", factory=lambda: DatabaseManager())
resolver.register("openai_client", factory=lambda: OpenAIClient(api_key=settings.OPENAI_API_KEY))
```

---

## 8. CẬP NHẬT TESTING

### ❌ SDD Hiện Tại

```python
# Test không sử dụng fixtures của project
def test_coder_agent():
    coder = CoderAgent()
    # ...
```

### ✅ Nên Update

```python
# app/module/agent/multi_agent_coding/tests/conftest.py

import pytest
from unittest.mock import MagicMock, AsyncMock

from app.common.agent.registry import AgentRegistry


@pytest.fixture(autouse=True)
def clear_registry():
    """Clear registry before each test."""
    AgentRegistry.clear_registry()
    yield
    AgentRegistry.clear_registry()


@pytest.fixture
def mock_bus():
    """Mock MessageBus."""
    bus = MagicMock()
    bus.publish = AsyncMock()
    bus.subscribe = MagicMock()
    bus.get_history = MagicMock(return_value=[])
    return bus


@pytest.fixture
def mock_dependencies():
    """Mock dependencies for agents."""
    return {
        "kafka": MagicMock(),
        "redis": MagicMock(),
        "llm": MagicMock(),
    }


# app/module/agent/multi_agent_coding/tests/test_coder.py

import pytest
from ..agents.coder import CoderAgent
from ..message_bus.models import Message


class TestCoderAgent:
    @pytest.fixture
    def coder(self, mock_bus):
        return CoderAgent(bus=mock_bus)
        
    @pytest.mark.asyncio
    async def test_decide_action_task_available(self, coder):
        msg = Message(
            from_agent="Chief",
            to_agent="broadcast",
            topic="task_available",
            payload={"goal": "Fix tests"}
        )
        
        action = await coder.decide_next_action(msg)
        assert action == "analyze_and_code"
        
    @pytest.mark.asyncio
    async def test_decide_action_fix_request(self, coder):
        msg = Message(
            from_agent="Reviewer",
            to_agent="Coder",
            topic="fix_request",
            payload={"error": "Test failed"}
        )
        
        action = await coder.decide_next_action(msg)
        assert action == "fix_bug"
```

---

## 9. CHECKLIST IMPLEMENTATION

### Phase 1: Setup (Day 1-2)

- [ ] Tạo folder structure mới: `app/module/agent/multi_agent_coding/`
- [ ] Tạo `agent.py` với `@agent` decorator
- [ ] Tạo `models.py` với Input/State models
- [ ] Tạo `message_bus/` package

### Phase 2: Agents (Day 3-4)

- [ ] Implement `P2PMixin` trong `base_p2p_agent.py`
- [ ] Migrate `ChiefAgent` sang pattern mới
- [ ] Migrate `CoderAgent`
- [ ] Migrate `TesterAgent`
- [ ] Migrate `ReviewerAgent`

### Phase 3: Integration (Day 5)

- [ ] Implement `MultiAgentCodingSystem` orchestrator
- [ ] Setup API route với Factory pattern
- [ ] Test auto-discovery
- [ ] Integration tests

### Phase 4: Polish (Day 6-7)

- [ ] Add Redis PubSub support
- [ ] Add Kafka event logging
- [ ] Documentation
- [ ] Demo preparation

---

## 10. TÓM TẮT THAY ĐỔI QUAN TRỌNG

| Component | Thay Đổi | Lý Do |
|-----------|----------|-------|
| **File Structure** | Đặt trong `app/module/agent/` | Auto-discovery |
| **Base Class** | Dùng `P2PMixin + BaseAgent` | Tái sử dụng framework |
| **Registration** | Dùng `@agent` decorator | Consistency |
| **Dependencies** | Dùng `dependency_specs` | DI pattern |
| **State** | Extend `BaseState` | Compatibility |
| **Message Bus** | Thêm Redis support | Multi-worker |
| **API** | Dùng `AgentFactory` | Factory pattern |

---

## 📎 REFERENCES

- `app/common/agent/base.py` - BaseAgent class
- `app/common/agent/registry.py` - AgentRegistry
- `app/common/agent/factory.py` - AgentFactory
- `app/common/agent/decorators.py` - @agent decorator
- `app/common/dependency_resolver.py` - DependencyResolver
- `app/common/redis/graph_pub_sub.py` - Redis PubSub pattern
- `app/module/agent/talk_agent/agent.py` - Reference implementation

---

*Document Version: 1.0*  
*Last Updated: 2025-12-17*

# 🎯 Best Practices Triển Khai Langfuse Python SDK

## 1. Client Initialization Pattern

### ✅ **Recommended: `get_client()` Pattern**

```python
# app/core/langfuse_client.py
from langfuse import get_client

# Singleton instance - initialized once, reused everywhere
langfuse_client = get_client()

# Export for use across the application
__all__ = ["langfuse_client"]
```

**Tại sao?**

- Clear intent: “Get existing client” thay vì “Create new instance”
- Tránh confusion với “new arguments are ignored” behavior
- Thread-safe và context-aware
- [Source](https://langfuse.com/docs/observability/sdk/overview)

---

## 2. Project Structure Best Practice

```
your_project/
├── app/
│   ├── core/
│   │   ├── __init__.py
│   │   └── langfuse_client.py      # Single initialization point
│   ├── services/
│   │   └── llm_service.py          # Import from core
│   ├── api/
│   │   └── routes.py               # Import from core
│   └── utils/
│       └── tracing_utils.py        # Helper functions
├── main.py                          # Application entry
└── .env                            # Environment variables
```

### ✅ **Single Import Pattern**

```python
# app/core/langfuse_client.py
from langfuse import get_client, observe
from langfuse._client import Langfuse

# Initialize once at startup
langfuse_client: Langfuse = get_client()

def get_langfuse():
    """Get the singleton Langfuse client."""
    return langfuse_client

__all__ = ["langfuse_client", "get_langfuse", "observe"]
```

---

## 3. Environment Configuration

### ✅ **Environment Variables (Best Practice)**

```bash
# .env
LANGFUSE_PUBLIC_KEY=pk-lf-...
LANGFUSE_SECRET_KEY=sk-lf-...
LANGFUSE_BASE_URL=https://cloud.langfuse.com  # or your self-hosted URL
LANGFUSE_TRACING_ENVIRONMENT=production
LANGFUSE_SAMPLE_RATE=1.0  # 1.0 = 100%, 0.1 = 10%
LANGFUSE_TRACING_ENABLED=true
LANGFUSE_FLUSH_INTERVAL=10.0  # seconds
```

**Ưu điểm:**

- Không cần hardcode credentials
- Dễ dàng chuyển đổi giữa environments
- Consistent với [Langfuse docs](https://langfuse.com/docs/observability/features/environments)

---

## 4. Instrumentation Patterns

### ✅ **Pattern 1: `@observe` Decorator (Recommended)**

```python
from app.core.langfuse_client import observe

@observe(as_type="generation")
async def call_llm(prompt: str, model: str = "gpt-4"):
    """Automatically traced LLM call."""
    response = await openai_client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content

# Update metadata if needed
@observe(as_type="generation")
async def call_llm_with_metadata(prompt: str):
    response = await call_llm(prompt)
    
    # Get client to update current span
    from app.core.langfuse_client import get_langfuse
    langfuse = get_langfuse()
    
    langfuse.update_current_generation(
        metadata={"custom_key": "value"},
        usage={
            "input": 100,
            "output": 50,
            "total": 150,
            "unit": "TOKENS",
            "input_cost": 0.001,
            "output_cost": 0.002
        }
    )
    return response
```

### ✅ **Pattern 2: Context Manager**

```python
from app.core.langfuse_client import get_langfuse

async def process_workflow(user_input: str):
    langfuse = get_langfuse()
    
    # Create trace context
    with langfuse.start_as_current_observation(
        as_type="span",
        name="workflow_processing"
    ) as span:
        # All nested calls automatically become children
        result = await step1(user_input)
        result = await step2(result)
        
        # Update span before exiting
        span.update(
            metadata={"workflow_version": "1.0.0"},
            output=result
        )
        return result
```

### ✅ **Pattern 3: Manual Observations (Advanced)**

```python
from app.core.langfuse_client import get_langfuse

async def background_task(data: dict):
    """For non-blocking, parallel operations."""
    langfuse = get_langfuse()
    
    # Create manual observation (no context shift)
    observation = langfuse.start_observation(
        as_type="span",
        name="background_processing"
    )
    
    try:
        result = await process_data(data)
        observation.end(
            output=result,
            status="success"
        )
    except Exception as e:
        observation.end(
            error=str(e),
            status="error"
        )
        raise
```

---

## 5. Async/Await Best Practices

### ✅ **Trong async code, rely on Langfuse helpers**

```python
# ✅ Good: Use decorator
@observe(as_type="generation")
async def async_llm_call():
    return await openai_client.chat.completions.create(...)

# ✅ Good: Use context manager with async
async def async_workflow():
    langfuse = get_langfuse()
    
    with langfuse.start_as_current_observation(as_type="span") as span:
        # await boundaries are handled correctly
        result1 = await step1()
        result2 = await step2(result1)
        return result2

# ❌ Avoid: Manual context management across await boundaries
async def bad_pattern():
    trace = langfuse.start_trace(name="bad")
    await step1()  # Context might be lost here!
    trace.end()  # Might not work correctly
```

[Source](https://langfuse.com/docs/observability/sdk/troubleshooting-and-faq)

---

## 6. Multi-Project Setup (Advanced)

### ✅ **Contextvars for Multi-Project**

```python
from langfuse._client.get_client import _set_current_public_key
from app.core.langfuse_client import get_langfuse

async def handle_multi_project_request(project_id: str):
    # Set context for this request
    with _set_current_public_key(project_id):
        langfuse = get_langfuse()
        # All traces go to correct project
        await process_request()
```

**Lưu ý:** Multi-project support là experimental. [Source](https://langfuse.com/docs/observability/sdk/advanced-features)

---

## 7. Error Handling & Resilience

### ✅ **Graceful Degradation**

```python
from app.core.langfuse_client import get_langfuse

async def safe_llm_call(prompt: str):
    try:
        langfuse = get_langfuse()
        
        with langfuse.start_as_current_observation(
            as_type="generation",
            name="llm_call"
        ) as span:
            response = await openai_client.chat.completions.create(...)
            span.update(output=response.choices[0].message.content)
            return response
            
    except Exception as e:
        # Langfuse won't break your app
        # Log error but don't crash
        logger.error(f"LLM call failed: {e}")
        # Return fallback or re-raise
        raise
```

---

## 8. Short-Lived Processes (Scripts/Serverless)

### ✅ **Always Flush/Shutdown**

```python
# scripts/batch_processing.py
from app.core.langfuse_client import get_langfuse

async def main():
    langfuse = get_langfuse()
    
    try:
        # Process data
        for item in dataset:
            await process_item(item)
    finally:
        # Critical: Flush before exit
        langfuse.flush()  # or langfuse.shutdown()
        # In async: await langfuse.async_shutdown()

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

**Tại sao?** SDK là asynchronous và buffers spans trong background. [Source](https://langfuse.com/docs/sdk/python/decorators)

---

## 9. Sampling Strategy

### ✅ **Production Setup**

```python
# .env.production
LANGFUSE_SAMPLE_RATE=0.1  # 10% sampling in production

# .env.development
LANGFUSE_SAMPLE_RATE=1.0   # 100% in development
```

```python
# For critical paths, force sampling
@observe(as_type="generation")
async def critical_path():
    # This will respect global sample_rate
    pass

# Or use manual sampling override (if supported)
```

**Lưu ý:** Với SDK v3, sampling được quản lý ở global OTEL TracerProvider level, không thể có different sampling rates cho cùng public_key. [Source](https://github.com/orgs/langfuse/discussions/7571)

---

## 10. Testing & Development

### ✅ **Mock Langfuse for Testing**

```python
# tests/conftest.py
import pytest
from unittest.mock import MagicMock

@pytest.fixture
def mock_langfuse():
    """Mock Langfuse client for unit tests."""
    mock = MagicMock()
    mock.start_as_current_observation.return_value.__enter__ = MagicMock(
        return_value=MagicMock()
    )
    mock.start_as_current_observation.return_value.__exit__ = MagicMock(
        return_value=False
    )
    return mock

# tests/test_service.py
async def test_llm_service(mock_langfuse, monkeypatch):
    monkeypatch.setattr(
        "app.services.llm_service.get_langfuse",
        lambda: mock_langfuse
    )
    
    result = await llm_service.call_llm("test prompt")
    assert result is not None
```

---

## 11. Monitoring & Debugging

### ✅ **Enable Debug Logging (Development Only)**

```python
# .env.development
LANGFUSE_DEBUG=True
```

```python
# Programmatic check
from app.core.langfuse_client import get_langfuse

langfuse = get_langfuse()

# Verify connectivity (don't use in production)
try:
    langfuse.auth_check()
    print("✅ Langfuse connected")
except Exception as e:
    print(f"❌ Langfuse connection failed: {e}")
```

---

## 12. Summary Checklist

|Aspect|Best Practice|Priority|
|---|---|---|
|**Client Init**|`get_client()` in single module|🔴 High|
|**Import Pattern**|Import from centralized module|🔴 High|
|**Config**|Environment variables|🔴 High|
|**Tracing**|`@observe` decorator|🟡 Medium|
|**Async**|Use decorators/context managers|🟡 Medium|
|**Shutdown**|`flush()`/`shutdown()` in scripts|🔴 High|
|**Sampling**|Environment-based rates|🟢 Low|
|**Multi-project**|Contextvars with caution|🟢 Low|
|**Testing**|Mock client|🟡 Medium|

---

## 13. Common Anti-Patterns to Avoid

### ❌ **Don’t: Create multiple Langfuse() instances expecting different configs**

```python
# ❌ Bad: This won't work as expected!
client1 = Langfuse(sample_rate=0.1)   # First call "wins"
client2 = Langfuse(sample_rate=1.0)   # Ignored! Same public_key
```

### ❌ **Don’t: Mix `get_client()` and `Langfuse()` without understanding**

```python
# ❌ Confusing and potentially problematic
from langfuse import Langfuse, get_client

# In one file
client_a = Langfuse()

# In another file
client_b = get_client()  # Might return different instance!
```

### ❌ **Don’t: Forget to flush in short-lived processes**

```python
# ❌ Data might be lost!
def lambda_handler(event, context):
    langfuse = get_langfuse()
    process(event)
    # Missing flush() or shutdown()!
```

---

## 14. Recommended Project Template

```python
# app/core/langfuse_client.py
"""
Centralized Langfuse client configuration.
All Langfuse interactions should go through this module.
"""
from langfuse import get_client, observe
from langfuse._client import Langfuse

# Initialize singleton client
_langfuse_client: Langfuse = get_client()

def get_langfuse() -> Langfuse:
    """Get the singleton Langfuse client."""
    return _langfuse_client

__all__ = ["get_langfuse", "observe"]
```

```python
# app/services/llm_service.py
from app.core.langfuse_client import get_langfuse, observe

@observe(as_type="generation")
async def call_llm(prompt: str, model: str = "gpt-4"):
    """LLM call with automatic tracing."""
    langfuse = get_langfuse()
    
    # Update metadata if needed
    langfuse.update_current_generation(
        metadata={"model": model, "prompt_version": "1.0"}
    )
    
    # Your LLM call here
    response = await openai_client.chat.completions.create(...)
    return response.choices[0].message.content
```

```python
# main.py
from app.core.langfuse_client import get_langfuse

async def main():
    langfuse = get_langfuse()
    
    try:
        # Your application logic
        pass
    finally:
        # Critical for clean shutdown
        await langfuse.async_shutdown()
```

---

Bạn muốn tôi đi sâu vào phần nào cụ thể hơn không? Ví dụ:

- Multi-project setup chi tiết
- Self-hosted Langfuse configuration
- Integration với FastAPI/FastAPI
- Prompt management best practices
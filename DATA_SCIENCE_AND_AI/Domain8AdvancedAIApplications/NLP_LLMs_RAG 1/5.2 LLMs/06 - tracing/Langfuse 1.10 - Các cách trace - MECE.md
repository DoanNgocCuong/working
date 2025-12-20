<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

## Các cách log vào Langfuse

Langfuse cung cấp **4 phương pháp chính** để instrument và log traces vào hệ thống observability:[^1][^2]

### 1. **@observe Decorator** (Tự động capture)

Wrapper function để tự động capture inputs, outputs, timings và errors mà không cần sửa logic bên trong function.[^2]

```python
from langfuse import observe

@observe(name="llm-call", as_type="generation")
async def my_async_llm_call(prompt_text):
    return "LLM response"

@observe()  # Tự động tạo TRACE ở top-level
def main_function(data, parameter):
    # Tự động tạo SPAN ở nested function
    return my_data_processing_function(data, parameter)
```

**Ưu điểm**:

- Đơn giản nhất, không xâm phạm code[^3]
- Tự động maintain call stack hierarchy[^3]
- Async-safe với Python Contextvars[^3]

**Tùy chỉnh**: Có thể tắt capture I/O để giảm overhead:[^2]

```python
@observe(capture_input=False, capture_output=False)
def sensitive_function():
    pass
```


### 2. **Context Manager** (start_as_current_observation)

Tạo span và set làm active observation trong OpenTelemetry context.[^2]

```python
from langfuse import get_client, propagate_attributes

langfuse = get_client()

with langfuse.start_as_current_observation(
    as_type="span",
    name="user-request-pipeline",
    input={"user_query": "Tell me a joke"},
) as root_span:
    # Nested spans tự động inherit parent
    with langfuse.start_as_current_observation(
        as_type="generation",
        name="joke-generation",
        model="gpt-4o",
    ) as generation:
        generation.update(output="Why did the span cross the road?")
    
    root_span.update(output={"final_joke": "..."})
```

**Ưu điểm**:

- Kiểm soát lifecycle của observation[^2]
- Tự động quản lý `.end()` khi exit context[^2]
- Hỗ trợ nesting tự nhiên qua OpenTelemetry context propagation[^2]


### 3. **Manual Observations** (start_observation)

Tạo observations thủ công không thay đổi active context.[^2]

```python
from langfuse import get_client

langfuse = get_client()

# Manual span - không thay đổi active context
span = langfuse.start_observation(name="manual-span")
span.update(input="Data for side task")

# Tạo child spans
child = span.start_observation(name="child-span", as_type="generation")
child.end()

span.end()  # ⚠️ PHẢI gọi .end() thủ công
```

**Khi nào dùng**:[^2]

- Background tasks song song với main execution
- Lifecycle được xác định bởi non-contiguous events
- Cần reference đến observation object trước khi tie vào context

**Lưu ý**: Phải tự gọi `.end()`, nếu không sẽ mất data.[^2]

### 4. **Native Integrations** (OpenAI, LangChain, ...)

Tự động tạo observations cho popular frameworks:[^2]

```python
# OpenAI Integration
from langfuse.openai import openai

client = openai.OpenAI()
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}],
    langfuse_public_key="pk-lf-..."  # Multi-project routing
)

# LangChain Integration
from langfuse.langchain import CallbackHandler

handler = CallbackHandler(public_key="pk-lf-...")
chain.invoke({"topic": "ML"}, config={"callbacks": [handler]})
```


***

## Advanced Techniques

### **Propagate Attributes** (Session ID, User ID, Metadata)

```python
from langfuse import propagate_attributes

with langfuse.start_as_current_observation(name="workflow"):
    with propagate_attributes(
        user_id="user_123",
        session_id="session_abc",
        metadata={"experiment": "variant_a"},
        version="1.0",
    ):
        # Tất cả nested observations tự động inherit attributes
        with langfuse.start_as_current_observation(name="llm-call"):
            pass
```

**Cross-service propagation** qua HTTP headers:[^1]

```python
with propagate_attributes(
    user_id="user_123",
    as_baggage=True,  # ⚠️ Add vào HTTP headers
):
    requests.get("https://service-b.example.com/api")
```


### **Update Trace-Level Input/Output**

Mặc định trace input/output mirror root observation, nhưng có thể override:[^2]

```python
with langfuse.start_as_current_observation(name="pipeline") as root:
    root.update(input="Step 1 data", output="Step 1 result")
    
    # Override trace-level (useful cho LLM-as-a-Judge)
    root.update_trace(
        input={"original_query": "User's question"},
        output={"final_answer": "Complete response", "confidence": 0.95}
    )
```


### **Mask Sensitive Data**

```python
from langfuse import Langfuse
import re

def pii_masker(data: any, **kwargs) -> any:
    if isinstance(data, str):
        return re.sub(r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+", 
                      "[EMAIL_REDACTED]", data)
    return data

langfuse = Langfuse(mask=pii_masker)
```


### **Sampling** (Giảm cost trong high-volume apps)

```python
# Sample 20% traces
langfuse = Langfuse(sample_rate=0.2)
```


### **Multi-Project Setup** (Experimental)

Route traces đến nhiều projects:[^1]

```python
# Top-most decorated function cần langfuse_public_key
@observe
def process_for_project_a(data):
    return result

result = process_for_project_a(
    data="input",
    langfuse_public_key="pk-lf-project-a-..."
)
```


***

## Flushing \& Lifecycle

**Quan trọng cho serverless/scripts**:[^2]

```python
from langfuse import get_client

langfuse = get_client()

# ... create traces ...

langfuse.flush()     # Đảm bảo send all buffered data
langfuse.shutdown()  # Graceful shutdown với cleanup
```


***

## So sánh các phương pháp

| Phương pháp | Ease of Use | Control Level | Auto Context | Use Case |
| :-- | :-- | :-- | :-- | :-- |
| **@observe** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ✅ | Simple functions, auto-tracing [^3] |
| **Context Manager** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ✅ | Complex workflows, explicit spans [^2] |
| **Manual** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ❌ | Background tasks, custom lifecycle [^2] |
| **Native Integrations** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ✅ | OpenAI, LangChain, Vercel AI SDK [^2] |


***

## Best Practices

1. **@observe cho entry point** để tạo TRACE tự động[^3]
2. **Context manager cho nested spans** với explicit control[^2]
3. **Propagate attributes sớm** trong trace để đảm bảo tất cả observations có metadata[^2]
4. **Flush() trong serverless** để tránh mất data[^2]
5. **Update trace-level I/O** cho LLM-as-a-Judge workflows[^2]

Bạn muốn tôi demo áp dụng cách nào cho `finai_agent`?
<span style="display:none">[^10][^4][^5][^6][^7][^8][^9]</span>

<div align="center">⁂</div>

[^1]: https://langfuse.com/docs/observability/sdk/python/advanced-usage

[^2]: https://langfuse.com/docs/observability/sdk/overview

[^3]: https://langfuse.com/blog/2024-04-python-decorator

[^4]: https://langfuse.com/docs/observability/features/log-levels

[^5]: https://langfuse.com/docs/observability/sdk/advanced-features

[^6]: https://langfuse.com/docs/sdk/python/decorators

[^7]: https://www.paulmduvall.com/llm-observability-with-langfuse-a-complete-guide/

[^8]: https://github.com/orgs/langfuse/discussions/8305

[^9]: https://github.com/langfuse/langfuse/issues/2238

[^10]: https://www.flowhunt.io/knowledge-base/how-to-trace-flowhunt-ai-agents-with-langfuse/



---

# Tại sao phải dùng cách nested spans này nhỉ, sao ko dùng observe vào đầu mỗi hàm là được mà nhỉ

---

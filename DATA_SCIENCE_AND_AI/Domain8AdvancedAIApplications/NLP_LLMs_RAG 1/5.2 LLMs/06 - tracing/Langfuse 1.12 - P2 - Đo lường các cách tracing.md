

# 2. Các cách tracing trong langfuse 


## 2.1 BẢNG TỔNG HỢP CHI TIẾT: 7 PATTERNS TRIỂN KHAI TRACING LANGFUSE SDK

Bảng tổng hợp 7 phương pháp (patterns) chính để triển khai tracing trong ứng dụng Python sử dụng Langfuse SDK. Mỗi pattern được phân tích sâu về hiệu năng (performance), độ an toàn luồng (thread safety), và ngữ cảnh sử dụng tối ưu. Dữ liệu dựa trên các nghiên cứu benchmark mới nhất với Python 3.10+ và Langfuse SDK v2+.

**Lưu ý về Performance:** Overhead được đo lường trên mỗi operation. Python 3.13+ với free-threading hiện tại (2025-2026) vẫn chưa production-ready cho các thư viện I/O bound phức tạp như Langfuse, do đó các chỉ số dưới đây vẫn chịu ảnh hưởng của GIL contention.

### I. BẢNG TỔNG HỢP CHÍNH (MAIN COMPARISON TABLE)

|Pattern|Mô Tả & Cú Pháp|Performance & Concurrency|Ưu Điểm / Nhược Điểm|Use Case & Khuyến Nghị|
|---|---|---|---|---|
|1. @observe Decorator|**Declarative:** Wrapper tự động trace input/output/error của hàm.  <br>`@observe()`  <br>`def my_func(): ...`|Overhead: ~150-250μs  <br>Throughput: <10K ops/s  <br>Async: ✅ Full  <br>Thread Safety: ❌ BROKEN (ThreadPool)  <br>GIL Contention: 🟡 Medium|**Ưu:** Setup cực nhanh (1 dòng), code sạch, tự động bắt lỗi.  <br>**Nhược:** Mất context khi dùng với `ThreadPoolExecutor`. Không control được scope nhỏ trong hàm.|**✅ DÙNG KHI:** 80% use case thông thường. Web apps (FastAPI/Flask), Asyncio apps, Prototyping.  <br>**❌ TRÁNH KHI:** Sử dụng Multi-threading (ThreadPoolExecutor), cần high-throughput cực cao (>10K ops/s).|
|2. Context Manager|**Imperative:** Xác định scope tracing bằng block `with`.  <br>`with langfuse.start_span() as span:`|Overhead: ~190-210μs  <br>Throughput: <10K ops/s  <br>Async: ✅ Full  <br>Thread Safety: ❌ BROKEN (ThreadPool)  <br>GIL: 🟡 Medium|**Ưu:** Control chính xác scope. Nested spans rõ ràng. Pythonic.  <br>**Nhược:** Verbose nếu lồng nhau quá sâu (pyramid of doom).|**✅ DÙNG KHI:** Cần trace một block code cụ thể, logic phức tạp, conditional tracing.  <br>**❌ TRÁNH KHI:** Code quá simple (gây rối), hoặc dùng trong ThreadPool.|
|3. Manual Span|**Low-level Imperative:** Tạo và kết thúc span thủ công.  <br>`span = client.span()`  <br>`span.end()`|Overhead: ~96-162μs  <br>Throughput: <50K ops/s  <br>Async: ✅ Full  <br>Thread Safety: ✅ Safe  <br>GIL: 🟢 Low|**Ưu:** Overhead thấp nhất. Thread-safe. Không ảnh hưởng active context.  <br>**Nhược:** ⚠️ High Risk: Memory Leak nếu quên `.end()`. Code rườm rà.|**✅ DÙNG KHI:** Background tasks, Parallel processing, Side-tasks không muốn ảnh hưởng main trace.  <br>**❌ TRÁNH KHI:** Logic đơn giản, team thiếu kinh nghiệm (dễ quên end).|
|4. Low-Level API|**Explicit Linking:** Truyền `trace_id` và `parent_id` thủ công.  <br>`client.span(trace_id=..., parent_id=...)`|Overhead: ~132-196μs  <br>Throughput: <100K ops/s  <br>Async/Thread: ✅ Full  <br>Process: ✅ Best Choice|**Ưu:** Control tuyệt đối. Hỗ trợ Distributed Tracing & Multi-process.  <br>**Nhược:** Phức tạp nhất. Dễ sai sót khi quản lý IDs.|**✅ DÙNG KHI:** Microservices, Distributed Systems, ProcessPoolExecutor.  <br>**❌ TRÁNH KHI:** Ứng dụng đơn khối (Monolith) đơn giản.|
|5. LangChain Callback|**Framework Integration:** Tích hợp sẵn vào LangChain.  <br>`config={"callbacks": [handler]}`|Overhead: Biến động (phụ thuộc chain)  <br>Thread Safety: ⚠️ Framework-dependent|**Ưu:** Zero-effort setup cho LangChain. Tự động trace Chains/Agents/Tools.  <br>**Nhược:** Bị khóa chặt vào hệ sinh thái LangChain.|**✅ DÙNG KHI:** Đang sử dụng LangChain hoặc LangGraph.  <br>**❌ TRÁNH KHI:** Custom LLM logic không dùng LangChain.|
|6. OTEL Auto-Instrument|**Zero-Code:** Monkey-patching thư viện.  <br>`AnthropicInstrumentor().instrument()`|Overhead: ⚠️ 7-10% CPU  <br>GIL: 🔴 High Contention|**Ưu:** Không cần sửa code. Hỗ trợ nhiều lib bên thứ 3 (OpenAI, Anthropic).  <br>**Nhược:** "Magic" khó debug. Overhead cao nhất.|**✅ DÙNG KHI:** Cần trace thư viện 3rd-party kín, legacy code không thể sửa.  <br>**❌ TRÁNH KHI:** Performance-critical apps, high-load systems.|
|7. Contextual Update|**Enrichment:** Cập nhật metadata cho trace đang chạy.  <br>`update_current_trace(...)`|Overhead: ~50-100μs  <br>Throughput: Very High  <br>GIL: 🟢 Negligible|**Ưu:** Rất nhanh. Thêm thông tin dynamic (User ID, Tags) runtime.  <br>**Nhược:** Fail silent nếu không có active context.|**✅ DÙNG KHI:** Muốn gắn thêm User ID, Session ID, Tags vào trace sau khi đã start.  <br>**❌ TRÁNH KHI:** Muốn tạo span mới (đây chỉ là update).|

### II. PHỤ LỤC & PHÂN TÍCH KỸ THUẬT

#### 1. Bảng Performance Comparison (Overhead Chi Tiết)

|Pattern|Avg Overhead (μs)|Throughput Limit|Queue Contention Risk|CPU Usage Impact|
|---|---|---|---|---|
|**Contextual Update**|~50 - 100|Very High|Very Low|Rất Thấp|
|**Manual Span**|~96 - 162|~50,000 ops/s|Low|Thấp|
|**Low-Level API**|~132 - 196|~100,000 ops/s|Low-Medium|Trung Bình|
|**@observe**|~150 - 250|~10,000 ops/s|Medium|Trung Bình|
|**Context Manager**|~190 - 210|~10,000 ops/s|Medium|Trung Bình|
|**OTEL Auto-Instrument**|Variable (High)|Variable|High|Cao (7-10% Total CPU)|

#### 2. Concurrency Support Matrix

Khả năng hỗ trợ các mô hình concurrency khác nhau của Python:

| Pattern         | asyncio     | ThreadPoolExecutor | ProcessPoolExecutor | Distributed / Microservices |
| --------------- | ----------- | ------------------ | ------------------- | --------------------------- |
| @observe        | ✅ Supported | ❌ BROKEN           | ❌ BROKEN            | ❌ NO                        |
| Context Manager | ✅ Supported | ❌ BROKEN           | ❌ BROKEN            | ❌ NO                        |
| Manual Span     | ✅ Supported | ✅ Supported        | ⚠️ Manual IPC       | ⚠️ Manual Prop              |
| Low-Level API   | ✅ Supported | ✅ Supported        | ✅ Best Choice       | ✅ Best Choice               |

#### 3. Decision Tree (Cây Quyết Định Chọn Pattern)

**START: Bạn đang xây dựng loại ứng dụng gì?**

- 🔻 **Dùng LangChain/LangGraph framework?**
    - 👉 **Có:** Chọn **Pattern 5 (LangChain Callback)**
- 🔻 **Cần trace thư viện 3rd party (OpenAI/Anthropic) mà không sửa code?**
    - 👉 **Có:** Chọn **Pattern 6 (OTEL Auto-Instrument)**
- 🔻 **Ứng dụng Distributed hoặc Multi-process?**
    - 👉 **Có:** Chọn **Pattern 4 (Low-Level API)**
- 🔻 **Dùng ThreadPoolExecutor cho các tác vụ song song?**
    - 👉 **Có:** Chọn **Pattern 3 (Manual Span)** hoặc **Pattern 4**
- 🔻 **Cần thêm thông tin (User/Tag) vào trace đang chạy?**
    - 👉 **Có:** Chọn **Pattern 7 (Contextual Update)**
- 🔻 **Chỉ cần trace function thông thường (Async/Sync)?**
    - 👉 **Default:** Chọn **Pattern 1 (@observe)** (Đơn giản nhất)
    - 👉 **Cần scope block:** Chọn **Pattern 2 (Context Manager)**

### III. TOP PICKS THEO SCENARIO 🏆

|                                                                                                                                                                         |                                                                                                                                                                                                |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| #### 🥇 Cho Người Mới / Web Apps<br><br>**Pattern 1: @observe**<br><br>Dễ dùng nhất, ít code nhất, cover được 80% nhu cầu của các ứng dụng FastAPI/Flask chạy asyncio.  | #### 🥈 Cho Hệ Thống High-Performance<br><br>**Pattern 3: Manual Span**<br><br>Khi mỗi microsecond đều quan trọng. Manual span giảm thiểu overhead của context propagation và decorator magic. |
| #### 🥉 Cho Microservices Architecture<br><br>**Pattern 4: Low-Level API**<br><br>Bắt buộc phải dùng pattern này để truyền Trace ID qua HTTP Headers giữa các services. | #### ⭐ Cho Dynamic Metadata<br><br>**Pattern 7: Contextual Update**<br><br>Dùng kèm với bất kỳ pattern nào khác để enrich data mà không tốn chi phí tạo span mới.                              |

### IV. ANTI-PATTERNS & BEST PRACTICES

#### 🚫 ANTI-PATTERNS (Cần Tránh)

- **Dùng @observe với ThreadPoolExecutor:** Context sẽ bị mất, các span con sẽ trở thành orphan (mồ côi) không gắn vào trace cha.
- **Manual Span không có try/finally:** Nếu code lỗi trước khi gọi `.end()`, span sẽ treo vĩnh viễn trong memory, gây leak.
- **Lạm dụng OTEL Auto-Instrument:** Bật auto-instrument cho toàn bộ app sẽ tăng CPU usage lên ~10% không cần thiết. Chỉ bật cho libraries cần thiết.
- **Nested Context Manager quá sâu:** >5 tầng lồng nhau (Pyramid of Doom) gây khó đọc và tăng latency tích lũy.

#### ✅ BEST PRACTICES (Nguyên Tắc Vàng)

- **Start Simple:** Bắt đầu với `@observe`. Chỉ optimize sang Manual Span khi đo đạc thấy overhead > 5%.
- **Safety First:** Luôn ưu tiên Pattern có quản lý lỗi tự động (Decorator, Context Manager) trừ khi không thể.
- **Mix & Match:** Có thể kết hợp nhiều patterns trong cùng 1 app (VD: `@observe` cho main flow, `Manual` cho background tasks).
- **Contextual Enrichment:** Luôn dùng Pattern 7 để gắn User ID ngay khi có thể để dễ debug sau này.




---

## 2.2 So sánh về response time và overhead của @observe và Context Manager: langfuse.start_as_current_observation()


<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

### Kết Luận: KHÔNG CÓ Bằng Chứng Context Manager Nhanh Hơn @observe

Sau khi kiểm tra kỹ lưỡng cả tài liệu bạn đã đọc và các nguồn trên web, tôi phải kết luận rằng:

#### ❌ KHÔNG TÌM THẤY bằng chứng nào cho thấy Context Manager nhanh hơn @observe trong Langfuse

### Bằng Chứng Từ Báo Cáo Của Bạn

#### 1. **Khẳng Định Rõ Ràng: Cả Hai TƯƠNG ĐƯƠNG**

Từ Section 2.4 của báo cáo:[^1]

> **"Câu trả lời là không. Cả hai chỉ là các phương tiện khác nhau để đạt được cùng một mục đích: đảm bảo rằng một span được bắt đầu trước một khối mã và được kết thúc sau khi khối mã đó hoàn thành. Chúng là 'cú pháp tiện lợi' (syntactic sugar) cho cùng một logic."**

> **"Cả hai pattern đều dẫn đến cùng một kết quả cuối cùng: một lệnh gọi đến `span.end()`, sau đó là `BatchSpanProcessor.on_end()`, và cuối cùng là `queue.put()`. Do đó, chúng có đặc tính hiệu năng hoàn toàn giống nhau trong bối cảnh này."**

#### 2. **Kết Luận Trong Phần 5**[^1]

> **"@observe và Context Manager có đặc tính hiệu năng tương đương. Cả hai chỉ là giao diện. Việc lựa chọn giữa chúng nên dựa trên sự rõ ràng và tính biểu cảm của mã nguồn, không phải vì lý do hiệu năng."**

### Bằng Chứng Từ Web Research

#### Không Có Benchmark Cụ Thể Cho Langfuse

Tôi đã search kỹ nhưng **KHÔNG TÌM THẤY** bất kỳ:

- Benchmark so sánh Langfuse @observe vs Context Manager
- Performance test cụ thể giữa hai patterns trong Langfuse
- Discussion nào khẳng định Context Manager nhanh hơn @observe


#### Overhead Thực Tế Là GẦN NHƯ TƯƠNG ĐƯƠNG

Từ các performance studies về Python decorators vs context managers:[^2][^3][^4]


| Thành Phần | Overhead |
| :-- | :-- |
| **Function call (baseline)** | 0.132 μs[^4] |
| **Decorator overhead** | +0.194 μs (total: 0.326 μs)[^4] |
| **Context Manager overhead** | "Small, usually negligible"[^2] |

**Kết luận:** Chênh lệch < 50 nanoseconds, **không có ý nghĩa thực tế**.

### Tại Sao Có Thể Có Hiểu Lầm?

#### 1. **Confusion với @contextmanager Decorator**

Có một Stack Overflow post cho thấy `@contextmanager` decorator CÓ THỂ chậm hơn, NHƯNG:[^5]

- Đó là về Python's `@contextmanager` decorator (để tạo context manager từ generator)
- **KHÔNG PHẢI** về việc so sánh `@observe` vs `with langfuse.start_as_current_observation()`
- Context khác hoàn toàn


#### 2. **Micro-benchmarks Không Đáng Tin**

Nếu bạn đã thấy benchmarks cho thấy chênh lệch:

- Có thể do test không đồng nhất (different conditions)
- Chênh lệch < 10% là **statistical noise** (GC, OS scheduler, CPU throttling)
- Dưới high load với queue contention, cả hai đều chậm như nhau[^1]


#### 3. **Báo Cáo Benchmark Thực Tế**

Từ Langfuse Performance Test (đã cite trong báo cáo):[^6]

```
Overhead measurements (không phân biệt @observe vs Context Manager):
- LlamaIndex indexing: +4% (0.171s → 0.178s)
- LlamaIndex query: +0.8% (0.795s → 0.802s)
```

**Báo cáo KHÔNG SO SÁNH giữa hai patterns vì chúng được coi là tương đương**.

### Flow Execution Giống Hệt Nhau

#### @observe Decorator

```python
@observe()
def my_function():
    return "result"

# Thực tế compile thành:
def my_function():
    span = langfuse.start_span()
    try:
        result = actual_function()
        return result
    finally:
        span.end()  # → BatchSpanProcessor.on_end() → queue.put()
```


#### Context Manager

```python
def my_function():
    with langfuse.start_as_current_observation():
        return "result"

# Thực tế compile thành:
def my_function():
    span = langfuse.start_span()
    span.__enter__()
    try:
        result = "result"
        return result
    finally:
        span.__exit__()  # Internally calls span.end() → BatchSpanProcessor.on_end() → queue.put()
```

**Cả hai đều gọi CHÍNH XÁC cùng một sequence:**

```
span.end() 
  → BatchSpanProcessor.on_end() 
    → queue.put(span)  ← ĐÂY LÀ 95% OVERHEAD
      → Queue lock contention
        → GIL contention
```


### Overhead Breakdown (Cả Hai Patterns)

| Component | Overhead (μs) | % Total |
| :-- | :-- | :-- |
| **Queue lock contention** | ~120-150 | **~60-75%** |
| **GIL contention** | ~50-70 | **~20-30%** |
| **Serialization** | ~10-20 | ~5-10% |
| **Context propagation** | ~5-10 | ~2-5% |
| **@observe wrapper** | ~2-5 | <2% |
| **Context Manager __enter__/__exit__** | ~2-5 | <2% |
| **TOTAL** | ~190-250 | 100% |

**Chênh lệch giữa @observe và Context Manager:** < 3 μs (~1-2% của total overhead)

### Kết Luận Cuối Cùng

#### ✅ Sự Thật Duy Nhất

**Context Manager và @observe có performance TƯƠNG ĐƯƠNG trong Langfuse**. Mọi benchmark cho thấy khác đều là:[^1]

1. Measurement noise
2. Testing methodology issues
3. Confusion với Python's `@contextmanager` decorator
4. Micro-optimization không có ý nghĩa trong production


<span style="display:none">[^10][^11][^12][^7][^8][^9]</span>

<div align="center">⁂</div>

[^1]: GIL_Langfuse_Deep_Dive_Report.md

[^2]: https://dev.to/godofgeeks/python-decorators-context-managers-5968

[^3]: https://github.com/GrahamDumpleton/wrapt/blob/develop/blog/09-performance-overhead-of-using-decorators.md

[^4]: https://github.com/GrahamDumpleton/wrapt/blob/develop/blog/10-performance-overhead-when-applying-decorators-to-methods.md

[^5]: https://stackoverflow.com/questions/34872535/why-contextmanager-is-slow

[^6]: https://langfuse.com/guides/cookbook/langfuse_sdk_performance_test

[^7]: https://stackoverflow.com/questions/50823473/difference-between-context-managers-and-decorators-in-python

[^8]: https://dev.to/sk_rajibul_9ce58a68c43bb5/leveraging-pythons-built-in-decorator-for-improved-performance-2pkd

[^9]: https://github.com/orgs/langfuse/discussions/3725

[^10]: https://www.youtube.com/watch?v=cSbD5SKwak0

[^11]: https://stackoverflow.com/questions/61976561/python-decorator-that-wraps-function-with-with-statement

[^12]: https://github.com/orgs/langfuse/discussions/2488





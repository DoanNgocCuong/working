<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# PHÂN LOẠI MECE: CÁC PHƯƠNG PHÁP LẬP TRÌNH CONCURRENT/PARALLEL TRONG PYTHON

KHUNG PHÂN LOẠI MECE (MUTUALLY EXCLUSIVE, COLLECTIVELY EXHAUSTIVE)

```bash
Python Concurrency/Parallelism Models
│
├─ 1. SEQUENTIAL (Baseline - Không có concurrency)
│   └─ Dùng: Tasks đơn giản, debugging, baseline benchmark
│
├─ 2. CONCURRENCY (Đồng thời - KHÔNG song song thực sự)
│   │
│   ├─ 2A. Asyncio (Event Loop - Single Thread)
│   │   ├─ ✅ Dùng: I/O-bound (API, DB, file)
│   │   ├─ Đặc điểm: Cooperative multitasking, 1 thread
│   │   └─ Hiệu quả: ⭐⭐⭐⭐⭐ cho I/O, ❌ cho CPU
│   │
│   └─ 2B. Threading (OS Threads - Bị GIL giới hạn)
│       ├─ ✅ Dùng: I/O-bound (legacy sync code không async được)
│       ├─ ❌ KHÔNG dùng: CPU-bound (GIL block, không cải thiện)
│       ├─ Đặc điểm: Preemptive, shared memory, nhiều threads
│       └─ Hiệu quả: ⭐⭐⭐ cho I/O, ❌ cho CPU
│
├─ 3. PARALLELISM (Song song THỰC SỰ - Bypass GIL)
│   │
│   ├─ 3A. Multiprocessing (Separate Processes)
│   │   ├─ ✅ Dùng: CPU-bound (tính toán nặng)
│   │   ├─ ❌ KHÔNG dùng: I/O-bound (overhead cao)
│   │   ├─ Đặc điểm: True parallelism, separate memory
│   │   └─ Hiệu quả: ⭐⭐⭐⭐⭐ cho CPU, ⭐ cho I/O
│   │
│   └─ 3B. Hybrid (Mix Asyncio/Threading + Multiprocessing)
│       ├─ ✅ Dùng: Mix I/O + CPU trong cùng workflow
│       ├─ VD: asyncio.run_in_executor() với ProcessPoolExecutor
│       └─ Hiệu quả: ⭐⭐⭐⭐ cho mixed workloads
│
└─ 4. SPECIALIZED (High-level wrappers - Dễ dùng hơn)
    │
    ├─ 4A. concurrent.futures.ThreadPoolExecutor
    │   ├─ ✅ Dùng: I/O-bound batch jobs (sync code)
    │   ├─ Bản chất: Threading wrapper với API đơn giản
    │   └─ Hiệu quả: = Threading (⭐⭐⭐ cho I/O)
    │
    ├─ 4B. concurrent.futures.ProcessPoolExecutor
    │   ├─ ✅ Dùng: CPU-bound batch jobs
    │   ├─ Bản chất: Multiprocessing wrapper với API đơn giản
    │   └─ Hiệu quả: = Multiprocessing (⭐⭐⭐⭐⭐ cho CPU)
    │
    └─ 4C. asyncio.run_in_executor()
        ├─ ✅ Dùng: Chạy sync/CPU code từ async context
        ├─ Bản chất: Bridge giữa asyncio và ThreadPool/ProcessPool
        └─ Hiệu quả: ⭐⭐⭐⭐ (best of both worlds)

```

***

**Sources:**,,,,,[^5][^2][^4][^6][^1][^3]
<span style="display:none">[^10][^11][^8][^9]</span>

<div align="center">⁂</div>

[^1]: https://stackoverflow.com/questions/27435284/multiprocessing-vs-multithreading-vs-asyncio/52498068

[^2]: https://www.geeksforgeeks.org/python/asyncio-vs-threading-in-python/

[^3]: https://github.com/Minibrams/fastapi-benchmark

[^4]: https://stackoverflow.com/questions/61351844/difference-between-multiprocessing-asyncio-threading-and-concurrency-futures-i

[^5]: https://leimao.github.io/blog/Python-Concurrency-High-Level/

[^6]: https://dev.to/kfir-g/unleash-the-power-of-fastapi-async-vs-blocking-io-4h0b

[^7]: https://langfuse.com/blog/2024-04-python-decorator

[^8]: https://www.reddit.com/r/learnpython/comments/1fhry6u/asyncio_vs_threading_vs_multiprocessing/

[^9]: https://www.linkedin.com/pulse/multithreading-vs-multiprocessing-asyncio-code-examples-kaushik-yxgjc

[^10]: https://dev.to/doziestar/concurrency-in-python-made-simple-51g3

[^11]: https://www.linkedin.com/pulse/python-concurrency-models-navigating-maze-concurrent-programming-srmoc

## 1.2.1 2A. Asyncio (Event Loop - Single Thread)

## 1.2.2  2B. Threading (OS Threads - Bị GIL giới hạn)

### 1.2.1 LANGFUSE SDK V3 SỬ DỤNG: **2B. THREADING (OS Threads)**

#### Phân Tích Chi Tiết

##### 🎯 Langfuse SDK v3 Architecture

Dựa trên báo cáo bạn đã đọc, Langfuse SDK v3 sử dụng: [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md)

```
Langfuse SDK v3 (Built on OpenTelemetry)
│
└─── BatchSpanProcessor
     │
     ├─ Main Thread (Application)
     │  └─ Gọi span.end() → queue.put(span)
     │
     └─ Worker Thread (Background)
        └─ Lấy từ queue → Batch → Export qua network
```

***

#### I. KẾT LUẬN: **2B. Threading (OS Threads - Bị GIL Giới Hạn)**

| Đặc Điểm | Langfuse SDK v3 | Evidence từ Báo Cáo |
|----------|-----------------|---------------------|
| **Model** | **Threading** | Sử dụng `threading.Thread` cho worker [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md) |
| **Số threads** | 2 threads (main + 1 worker) | BatchSpanProcessor tạo 1 daemon thread [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md) |
| **Queue** | `queue.Queue` (thread-safe) | Dùng để giao tiếp giữa threads [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md) |
| **Task type** | **I/O-bound** (gửi trace data qua network) | Export spans đến Langfuse server [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md) |
| **GIL impact** | ❌ **Bị ảnh hưởng** | Queue lock + GIL contention [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md) |
| **Memory** | Shared memory | Threads chia sẻ memory space [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md) |

***

#### II. CODE EVIDENCE TỪ OPENTELEMETRY

Từ báo cáo, phần Appendix A phân tích source code: [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md)

```python
### OpenTelemetry BatchSpanProcessor
class BatchSpanProcessor(SpanProcessor):
    def __init__(self, span_exporter, ...):
        ### 1. Khởi tạo queue (thread-safe)
        self.queue = queue.Queue(self.max_queue_size)
        
        ### 2. Khởi tạo threading.Lock
        self.condition = threading.Condition(threading.Lock())
        
        ### 3. Tạo worker thread
        self.worker_thread = threading.Thread(
            target=self.worker, 
            daemon=True  ### Background thread
        )
        self.worker_thread.start()
    
    def on_end(self, span):
        ### Main thread gọi hàm này
        self.queue.put(span)  ### ← Queue lock contention tại đây!
    
    def worker(self):
        ### Worker thread chạy loop này
        while True:
            spans = self.queue.get()  ### Lấy từ queue
            self.span_exporter.export(spans)  ### I/O: gửi qua network
```

***

#### III. TẠI SAO DÙNG THREADING (KHÔNG PHẢI ASYNCIO)?

##### Lý Do Lịch Sử & Thiết Kế

| Lý Do | Giải Thích |
|-------|-----------|
| **1. Compatibility** | OpenTelemetry phải hỗ trợ cả sync và async apps [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md) |
| **2. Simplicity** | Threading dễ implement hơn asyncio cho background tasks |
| **3. Standard** | Threading là tiêu chuẩn của OTEL SDK cho tất cả ngôn ngữ |
| **4. Isolation** | Worker thread độc lập, không ảnh hưởng app logic |

##### ✅ Ưu Điểm Của Lựa Chọn Này

1. **Không block main thread** - I/O export chạy background [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md)
2. **Đơn giản** - API dễ dùng, tích hợp sẵn
3. **Universal** - Hoạt động với cả sync và async apps

##### ❌ Nhược Điểm (Đã Phân Tích Trong Báo Cáo)

1. **Queue lock contention** - Nhiều spans cùng ghi vào queue [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md)
2. **GIL contention** - Main thread và worker thread tranh chấp GIL [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md)
3. **Overhead ~150-250μs** - Do queue + GIL contention [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md)

***

#### IV. SO SÁNH VỚI CÁC MODELS KHÁC

##### Nếu Langfuse Dùng Models Khác?

| Model | Feasible? | Ưu Điểm | Nhược Điểm |
|-------|-----------|---------|------------|
| **1. Sequential** | ❌ Không | - | Block main thread khi export |
| **2A. Asyncio** | ⚠️ Khó | Overhead thấp hơn | Chỉ hoạt động với async apps, breaking change |
| **2B. Threading** | ✅ **HIỆN TẠI** | Universal, simple | Queue + GIL contention [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md) |
| **3A. Multiprocessing** | ❌ Không hợp lý | Bypass GIL | Overhead cực cao cho I/O task |
| **4A. ThreadPoolExecutor** | ⚠️ Tương tự | Tương tự Threading | Không cải thiện đáng kể |
| **4B. ProcessPoolExecutor** | ❌ Không hợp lý | - | Lãng phí resources cho I/O |

##### Kết Luận: Threading Là Lựa Chọn Hợp Lý (Nhưng Không Tối Ưu)

**Threading** là compromise tốt nhất cho:
- ✅ Compatibility (sync + async apps)
- ✅ Simplicity (OpenTelemetry standard)
- ⚠️ Performance acceptable cho most cases

**Nhưng** dưới high load (>10K ops/s), overhead trở nên đáng kể. [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md)

***

#### V. GIẢI PHÁP "ZERO OVERHEAD" ĐỀ XUẤT

Báo cáo đề xuất chuyển từ **Threading** sang **3B. Hybrid**:

```
Giải Pháp Đề Xuất (Redis + Worker Process)
│
├─ Main Application (Asyncio hoặc Threading)
│  └─ Redis LPUSH (I/O async, không có queue lock)
│
└─ Separate Worker Process (Multiprocessing)
   └─ Redis BRPOP → Batch → Export
   
→ Loại bỏ hoàn toàn GIL contention giữa app và SDK!
```

##### So Sánh Models

| Aspect | Hiện Tại (Threading) | Đề Xuất (Hybrid) |
|--------|---------------------|------------------|
| **Model** | 2B. Threading | 3B. Hybrid (Asyncio + Multiprocessing) |
| **Overhead** | ~190-210μs | ~50-100μs (Redis write only) |
| **GIL Contention** | 🔴 High | 🟢 None (separate process) |
| **Throughput Limit** | ~10K ops/s | ~100K+ ops/s |
| **Complexity** | ⭐⭐ Simple | ⭐⭐⭐⭐ High (Redis + worker) |

***

#### VI. TÓM TẮT

##### Langfuse SDK v3 Hiện Tại

```
┌─────────────────────────────────────┐
│  2B. Threading (OS Threads)         │
├─────────────────────────────────────┤
│  • 1 main thread                    │
│  • 1 worker thread (background)    │
│  • queue.Queue (thread-safe)       │
│  • I/O-bound task (network export) │
│  • Bị GIL + Queue lock contention  │
└─────────────────────────────────────┘
```

##### Vị Trí Trong Phân Loại

```
Python Concurrency Models
│
├─ 2. CONCURRENCY
│   └─ 2B. Threading ← LANGFUSE SDK V3 Ở ĐÂY
│       ├─ ✅ Phù hợp: I/O-bound (export trace)
│       ├─ ✅ Đơn giản: Threading.Thread
│       └─ ❌ Vấn đề: GIL contention under high load
```

##### Key Takeaways

1. **Hiện tại:** Threading (2B) - Hợp lý cho compatibility
2. **Vấn đề:** Queue lock + GIL contention dưới high load [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md)
3. **Giải pháp:** Hybrid (3B) - Redis + Worker Process để eliminate contention [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md)
4. **Trade-off:** Simplicity vs Performance

**Nguồn:**  GIL_Langfuse_Deep_Dive_Report.md, Section 2.2 "Mổ Xẻ BatchSpanProcessor" [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/760047/b0731d15-cc68-4790-991c-ea478eb9fc3e/GIL_Langfuse_Deep_Dive_Report.md)

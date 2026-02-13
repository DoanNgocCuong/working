<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# PHÂN LOẠI MECE: CÁC PHƯƠNG PHÁP LẬP TRÌNH CONCURRENT/PARALLEL TRONG PYTHON

## I. KHUNG PHÂN LOẠI MECE (MUTUALLY EXCLUSIVE, COLLECTIVELY EXHAUSTIVE)

### Chiều 1: Execution Model (Mô Hình Thực Thi)

```
Python Concurrency/Parallelism Models
│
├─ 1. SEQUENTIAL (Baseline - Không có concurrency)
│
├─ 2. CONCURRENCY (Đồng thời - không phải song song)
│   ├─ 2A. Asyncio (Event Loop - Single Thread)
│   └─ 2B. Threading (OS Threads - Bị GIL giới hạn)
│
├─ 3. PARALLELISM (Song song thực sự)
│   ├─ 3A. Multiprocessing (Separate Processes)
│   └─ 3B. Hybrid (Threads + Processes kết hợp)
│
└─ 4. SPECIALIZED (Chuyên biệt)
    ├─ 4A. concurrent.futures.ThreadPoolExecutor
    ├─ 4B. concurrent.futures.ProcessPoolExecutor
    └─ 4C. asyncio + run_in_executor (Hybrid)
```


***

## II. BẢNG SO SÁNH TOÀN DIỆN

| Phương Pháp | Execution Unit | Memory Model | GIL Impact | Best For | Overhead | Max Throughput | Code Complexity |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| **1. Sequential** | Single thread, single process | Single memory space | N/A | Simple tasks, debugging | ⚡ Không có | Thấp nhất (baseline) | ⭐ Rất đơn giản |
| **2A. Asyncio (FastAPI `async def`)** | Single thread + Event Loop | Single memory space | ❌ Bị GIL (nhưng không matter)[^1] | **I/O-bound** (API, DB, file)[^2] | ⚡⚡ Rất thấp (~1 thread)[^3] | **RẤT CAO** cho I/O[^3] | ⭐⭐⭐ Trung bình (async/await syntax) |
| **2B. Threading** | Multiple OS threads | **Shared memory** | ❌❌ **BỊ GIL block**[^1] | I/O-bound (legacy code) | ⚡⚡⚡ Trung bình (context switch) | Trung bình | ⭐⭐⭐⭐ Khó (race conditions, locks) |
| **3A. Multiprocessing** | Multiple processes | **Separate memory** (IPC needed) | ✅ **KHÔNG bị GIL**[^1] | **CPU-bound** (tính toán nặng)[^1] | 🐢🐢🐢 Cao (spawn process ~ms) | Cao (= số cores) | ⭐⭐⭐⭐ Khó (IPC, serialization) |
| **4A. ThreadPoolExecutor** | Thread pool (reusable threads) | Shared memory | ❌❌ Bị GIL | I/O-bound batch jobs | ⚡⚡ Thấp (reuse threads) | Trung bình-Cao | ⭐⭐ Dễ (high-level API) |
| **4B. ProcessPoolExecutor** | Process pool (reusable processes) | Separate memory | ✅ Không bị GIL | CPU-bound batch jobs | 🐢🐢 Trung bình (reuse processes) | Cao (= số cores) | ⭐⭐ Dễ (high-level API) |
| **4C. asyncio + run_in_executor** | Event Loop + Thread/Process pool | Mixed | Partial (depends on executor) | **Mix I/O + CPU**[^4] | ⚡⚡⚡ Trung bình | Rất Cao | ⭐⭐⭐⭐ Khó (mix paradigms) |


***

## III. SO SÁNH CHI TIẾT: ASYNC vs THREADING vs MULTIPROCESSING

### A. Kiến Trúc \& Execution Model

| Aspect | Asyncio | Threading | Multiprocessing |
| :-- | :-- | :-- | :-- |
| **Số threads** | 1 thread duy nhất[^5] | Nhiều threads (VD: 10-100) | 1 thread/process (nhưng nhiều processes) |
| **Số processes** | 1 process | 1 process | Nhiều processes (VD: 4-16) |
| **Context switching** | **Cooperative** (tự nguyện)[^1] | **Preemptive** (OS quyết định) | **Preemptive** (OS quyết định) |
| **Shared state** | ✅ Dễ (cùng memory) | ⚠️ Khó (cần locks, race conditions)[^2] | ❌ Rất khó (cần IPC, serialization) |
| **True parallelism** | ❌ Không (sequential execution)[^1] | ❌ Không (GIL block)[^1] | ✅ **Có** (bypass GIL)[^1] |

### B. Performance Characteristics

#### I/O-Bound Task (VD: 1000 HTTP requests)

| Method | Execution Time | Resource Usage | Scalability |
| :-- | :-- | :-- | :-- |
| **Sequential** | ~1000 seconds (1s/request) | 1 CPU, minimal RAM | ❌ Không scale |
| **Asyncio** | ~10-20 seconds[^3] | 1 CPU, 50MB RAM | ✅✅✅ **Xuất sắc** (10K+ concurrent) |
| **Threading** | ~15-30 seconds | 1-2 CPU, 200MB+ RAM | ✅✅ Tốt (100-1000 concurrent) |
| **Multiprocessing** | ~25-40 seconds | 4+ CPU, 500MB+ RAM | ⚠️ Kém (overhead cao)[^1] |

**Winner:** **Asyncio** (10-50x faster, ít resources nhất)[^5][^2]

#### CPU-Bound Task (VD: Tính toán 1 triệu số)

| Method | Execution Time (4 cores) | CPU Usage | Winner |
| :-- | :-- | :-- | :-- |
| **Sequential** | ~40 seconds (baseline) | 25% (1 core) | - |
| **Asyncio** | ~39 seconds (**GẦN NHƯ KHÔNG cải thiện**)[^1] | 25% (1 core) | ❌ |
| **Threading** | ~38 seconds (GIL block, gần như không cải thiện)[^1] | 25-30% | ❌ |
| **Multiprocessing** | **~10 seconds** (4x faster)[^1] | 100% (4 cores) | ✅✅✅ |

**Winner:** **Multiprocessing** (chỉ lựa chọn duy nhất cho CPU-bound)[^1]

***

## IV. FASTAPI: ASYNC VS SYNC ENDPOINTS

### Benchmark Thực Tế (500 Concurrent Users)[^3]

| Endpoint Type | Code | Requests/sec | Median Response | Threads Created | Failures |
| :-- | :-- | :-- | :-- | :-- | :-- |
| **Sync endpoint + Sync I/O** | `def` + `requests.get()` | **35.6** | 1,300 ms | 41 threads | 0% |
| **Async endpoint + Async I/O** | `async def` + `httpx.get()` | **53.2** | 8,300 ms | **1 thread** | 0% |
| **Async endpoint + Sync I/O** | `async def` + `requests.get()` | **13.1** ❌ | 27,000 ms | 1 thread | **93.2%** ❌❌ |

### Kết Luận FastAPI:[^6][^3]

1. **✅ ĐÚNG: `async def` + async I/O (httpx, asyncpg)** → Hiệu năng cao nhất
2. **✅ OK: `def` + sync I/O (requests, psycopg2)** → FastAPI tự động dùng ThreadPool
3. **❌❌ SAI LẦM CHẾT NGƯỜI: `async def` + sync I/O** → Block event loop, failures 93%[^3]

**Nguyên tắc vàng:**

```python
# ✅ ĐÚNG
@app.get("/users")
async def get_users():
    async with httpx.AsyncClient() as client:  # Async I/O
        return await client.get("https://api.example.com/users")

# ✅ CŨNG OK (FastAPI handle bằng ThreadPool)
@app.get("/users")
def get_users():  # Sync endpoint
    return requests.get("https://api.example.com/users")  # Sync I/O

# ❌❌ TUYỆT ĐỐI TRÁNH
@app.get("/users")
async def get_users():  # Async endpoint
    return requests.get("...")  # Sync I/O → BLOCK EVENT LOOP!
```


***

## V. DECISION MATRIX (KHI NÀO DÙNG CÁI NÀO)

### A. Theo Task Type

| Task Characteristics | Best Choice | Why | Throughput Potential |
| :-- | :-- | :-- | :-- |
| **I/O-bound: API calls, DB queries, file I/O** | **Asyncio** (FastAPI `async def`)[^2] | Event loop hiệu quả nhất, ít overhead | ⭐⭐⭐⭐⭐ (10K-100K ops/s) |
| **I/O-bound: Legacy code không thể async** | ThreadPoolExecutor | Wrapper sync code dễ dàng | ⭐⭐⭐ (100-1K ops/s) |
| **CPU-bound: Tính toán, data processing** | **ProcessPoolExecutor**[^1] | Bypass GIL, true parallelism | ⭐⭐⭐⭐ (= số cores × performance) |
| **CPU-bound: Nhẹ (< 100ms)** | Sequential hoặc Threading | Overhead không đáng | ⭐⭐ |
| **Mix I/O + CPU** | asyncio + `run_in_executor()`[^4] | I/O trên event loop, CPU trên processes | ⭐⭐⭐⭐ |

### B. Theo Infrastructure \& Constraints

| Constraint | Recommendation | Reason |
| :-- | :-- | :-- |
| **Single-core server** | Asyncio | Multiprocessing vô ích khi chỉ 1 core |
| **Multi-core server (4+)** | Multiprocessing cho CPU-bound | Tận dụng parallelism |
| **RAM giới hạn** | Asyncio > Threading > Multiprocessing | Asyncio dùng ít RAM nhất[^2] |
| **Cần shared state** | Asyncio hoặc Threading | Multiprocessing cần IPC phức tạp |
| **Microservices (stateless)** | Asyncio (FastAPI) | Scalability tốt nhất |
| **Legacy codebase** | ThreadPoolExecutor | Wrap sync code mà không refactor |


***

## VI. CODE EXAMPLES ĐẦY ĐỦ

### 1. Asyncio (FastAPI Style)

```python
import asyncio
import httpx

# ✅ Best for: 1000s of concurrent I/O operations
async def fetch_many_urls(urls):
    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        # All run concurrently on 1 thread!
        responses = await asyncio.gather(*tasks)
    return responses

# FastAPI endpoint
@app.get("/data")
async def get_data():
    urls = ["https://api1.com", "https://api2.com", ...]
    return await fetch_many_urls(urls)
```


### 2. ThreadPoolExecutor

```python
from concurrent.futures import ThreadPoolExecutor
import requests

# ✅ Best for: I/O-bound batch jobs (legacy sync code)
def fetch_url(url):
    return requests.get(url).json()

urls = ["https://api1.com", "https://api2.com"] * 100

with ThreadPoolExecutor(max_workers=20) as executor:
    results = list(executor.map(fetch_url, urls))
```


### 3. ProcessPoolExecutor

```python
from concurrent.futures import ProcessPoolExecutor
import numpy as np

# ✅ Best for: CPU-intensive calculations
def heavy_compute(n):
    # Tính toán phức tạp
    return np.sum(np.random.rand(n, n) ** 2)

numbers = [5000, 10000, 15000, 20000]

with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(heavy_compute, numbers))
    # Chạy thật sự song song trên 4 cores!
```


### 4. Hybrid: Asyncio + Executor (Best of Both Worlds)

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

async def hybrid_workflow():
    loop = asyncio.get_event_loop()
    executor = ProcessPoolExecutor(max_workers=4)
    
    # I/O-bound: chạy async
    data = await fetch_data_from_api()
    
    # CPU-bound: offload sang process pool
    result = await loop.run_in_executor(
        executor, 
        heavy_computation, 
        data
    )
    
    return result
```


***

## VII. ANTI-PATTERNS \& COMMON MISTAKES

| ❌ Anti-Pattern | Vấn Đề | ✅ Fix |
| :-- | :-- | :-- |
| `async def` + `requests.get()` | Block event loop[^3] | Dùng `httpx.AsyncClient()` |
| Threading cho CPU-bound | GIL block, không cải thiện[^1] | Dùng Multiprocessing |
| Multiprocessing cho I/O-bound | Overhead cao không cần thiết[^1] | Dùng Asyncio |
| Shared state trong Multiprocessing | Cần Manager, Queue, Pipe phức tạp | Dùng Threading hoặc Asyncio |
| Threading không có locks | Race conditions, data corruption[^2] | Dùng `threading.Lock()` |
| Asyncio với CPU-intensive tasks | Block event loop lâu | Offload sang `run_in_executor()` |


***

## VIII. PERFORMANCE SUMMARY TABLE

### I/O-Bound (1000 API calls)

| Method | Time | Speedup | Resource |
| :-- | :-- | :-- | :-- |
| Sequential | 1000s | 1x | 1 CPU, 10MB |
| **Asyncio** | **15s** | **67x** | 1 CPU, 50MB |
| Threading | 25s | 40x | 2 CPU, 200MB |
| Multiprocessing | 50s | 20x | 4 CPU, 500MB |

### CPU-Bound (Heavy computation)

| Method | Time | Speedup | CPU Usage |
| :-- | :-- | :-- | :-- |
| Sequential | 40s | 1x | 25% (1 core) |
| Asyncio | 39s | 1.03x ❌ | 25% |
| Threading | 38s | 1.05x ❌ | 30% |
| **Multiprocessing** | **10s** | **4x** | 100% (4 cores) |


***

## IX. KẾT LUẬN \& BEST PRACTICES

### 🎯 Top Picks

| Use Case | \#1 Choice | Alternative | Avoid |
| :-- | :-- | :-- | :-- |
| **Web APIs (FastAPI)** | Asyncio (`async def`) | Threading (`def`) | Multiprocessing |
| **Data processing** | ProcessPoolExecutor | - | Threading |
| **Web scraping** | Asyncio + httpx | ThreadPoolExecutor | Multiprocessing |
| **Batch jobs (I/O)** | ThreadPoolExecutor | Asyncio | - |
| **ML training** | Multiprocessing | GPU (if available) | Threading |
| **Legacy sync code** | ThreadPoolExecutor | Refactor to async | Force async |

### 📏 Nguyên Tắc Chọn Lựa

1. **I/O-bound → Asyncio first** (FastAPI style)[^2][^6]
2. **CPU-bound → Multiprocessing only**[^1]
3. **Can't refactor → ThreadPoolExecutor** (wrapper cho sync code)
4. **Mix I/O + CPU → Asyncio + run_in_executor**[^4]
5. **Always measure** - Premature optimization là gốc rễ của mọi tội lỗi

### ⚠️ Caveat về Langfuse SDK

Như đã đề cập trong bảng trước:

- **@observe + ThreadPoolExecutor/ProcessPoolExecutor → BROKEN**[^7]
- **Phải dùng Low-Level API với manual trace_id propagation**

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


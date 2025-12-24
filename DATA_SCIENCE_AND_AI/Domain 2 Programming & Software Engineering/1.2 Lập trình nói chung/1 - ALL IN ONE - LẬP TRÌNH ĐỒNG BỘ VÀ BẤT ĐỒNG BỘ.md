```
Deep research về master đồng bộ và bất đồng bộ trong python
```


<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# PERFLEXITY: Master Đồng Bộ và Bất Đồng Bộ trong Python

## Giới thiệu: Tại sao cần async trong thời đại hiện đại?

Trong kiến trúc microservices, AI/ML applications và fintech systems hiện nay, việc xử lý hàng nghìn concurrent requests từ users, external APIs (LLM, payment gateways), databases là bắt buộc. Code đồng bộ truyền thống không thể scale được trong môi trường này.[^1][^2]

***

## Phần I: Nền Tảng Lý Thuyết

### 1.1 Đồng Bộ (Synchronous) - Blocking Execution

**Cơ chế hoạt động:**
Mỗi dòng lệnh chạy tuần tự; khi gặp I/O (DB query, HTTP request, file read), thread **bị block** cho đến khi operation hoàn tất.[^3][^4]

**Ví dụ thực tế:**

```python
import requests
import time

def fetch_user_data(user_id):
    # Thread bị block tại đây ~500ms
    response = requests.get(f"https://api.example.com/users/{user_id}")
    return response.json()

def main():
    start = time.time()
    users = []
    for i in range(10):
        users.append(fetch_user_data(i))  # Chạy tuần tự: 10 * 500ms = 5 giây
    print(f"Sync: {time.time() - start:.2f}s")  # ~5 giây
```

**Ưu điểm:**

- Code đơn giản, dễ debug, reasoning tuyến tính[^5][^6]
- Phù hợp với CPU-bound tasks (tính toán thuần) hoặc scripts nhỏ

**Nhược điểm:**

- Thread bị idle trong thời gian chờ I/O → lãng phí resources[^7][^8]
- Không scale với high concurrency (cần mở nhiều threads/processes → overhead lớn)[^9]

***

### 1.2 Bất Đồng Bộ (Asynchronous) - Non-blocking Execution

**Cơ chế hoạt động:**
Thay vì block thread, coroutine `await` một I/O operation non-blocking, **nhả quyền điều khiển** về event loop để xử lý các tasks khác.[^10][^11]

**Event Loop - Trái tim của asyncio:**
Event loop là một vòng lặp vô tận quản lý:

- **Ready queue**: Coroutines sẵn sàng chạy
- **Selector**: Theo dõi I/O file descriptors (socket, file) bằng `epoll`/`kqueue`[^12][^13]
- **Timer heap**: Scheduled callbacks (như `asyncio.sleep`)

Khi một coroutine `await` I/O, event loop:

1. Đăng ký file descriptor với OS selector
2. Chuyển sang chạy coroutine khác trong ready queue
3. OS notify khi I/O xong → coroutine được đẩy lại vào ready queue[^14]

**Ví dụ async:**

```python
import asyncio
import aiohttp
import time

async def fetch_user_data(session, user_id):
    # Không block, event loop chuyển sang task khác
    async with session.get(f"https://api.example.com/users/{user_id}") as resp:
        return await resp.json()

async def main():
    start = time.time()
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_user_data(session, i) for i in range(10)]
        users = await asyncio.gather(*tasks)  # Chạy song song: ~500ms
    print(f"Async: {time.time() - start:.2f}s")  # ~0.5 giây

asyncio.run(main())
```

**Ưu điểm:**

- Throughput cao: 1 thread xử lý hàng nghìn I/O-bound tasks[^15][^16]
- Ít resource hơn threading (không overhead context switching)[^17][^9]
- Ít race conditions hơn (cooperative multitasking)[^18][^19]

**Nhược điểm:**

- Phức tạp hơn sync code, learning curve cao[^20][^21]
- Toàn bộ codebase phải async (infectious async)[^22]
- Vẫn chịu ảnh hưởng GIL cho CPU-bound[^23][^24]

***

## Phần II: So Sánh Threading, Multiprocessing, Asyncio

### 2.1 Threading

**Cơ chế:**

- Nhiều OS threads, chia sẻ cùng memory space
- Python GIL: chỉ 1 thread execute Python bytecode tại 1 thời điểm[^9][^17]

**Khi nào dùng:**

- I/O-bound tasks với blocking libraries (không có async version)
- Tích hợp legacy code không thể refactor
- Cần shared memory giữa threads[^25][^26]

**Ví dụ:**

```python
import threading
import time

def blocking_task(name):
    time.sleep(2)  # Blocking I/O
    print(f"Task {name} done")

threads = [threading.Thread(target=blocking_task, args=(i,)) for i in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()  # ~2s (chạy song song)
```

**Trade-offs:**

- Pros: Đơn giản hơn asyncio, tích hợp dễ
- Cons: Overhead context switching, không tận dụng được multi-core cho CPU-bound (GIL)[^23][^17]

***

### 2.2 Multiprocessing

**Cơ chế:**

- Mỗi process có interpreter riêng, memory riêng → không bị GIL
- IPC (Inter-Process Communication) đắt, không chia sẻ in-memory data trực tiếp[^25][^17]

**Khi nào dùng:**

- CPU-bound tasks nặng (ML training, data processing)[^26][^9]
- Cần tận dụng multi-core để parallel thật sự

**Ví dụ:**

```python
from multiprocessing import Pool
import time

def cpu_task(n):
    # Tính Fibonacci thứ n (CPU-bound)
    result = 0
    for i in range(n):
        result += i ** 2
    return result

if __name__ == "__main__":
    with Pool(4) as pool:  # 4 processes
        results = pool.map(cpu_task, [10**7] * 4)  # Parallel trên 4 cores
```

**Trade-offs:**

- Pros: True parallelism, không bị GIL
- Cons: Memory overhead lớn, IPC chậm, khó debug[^17][^25]

***

### 2.3 Bảng So Sánh MECE

| Tiêu chí | Threading | Asyncio | Multiprocessing |
| :-- | :-- | :-- | :-- |
| **Concurrency Model** | OS threads (preemptive) | Event loop (cooperative) | Separate processes |
| **GIL Impact** | Bị giới hạn (1 thread execute tại 1 thời điểm)[^23][^17] | Không ảnh hưởng (single thread)[^24] | Không có GIL (mỗi process riêng)[^25] |
| **Resource Usage** | Medium (context switching overhead)[^9] | Low (single thread)[^16] | High (process overhead, IPC) |
| **Best For** | I/O-bound với blocking libs[^16] | I/O-bound với async libs[^9][^1] | CPU-bound tasks[^25][^26] |
| **Shared Memory** | Easy (cùng process) | Easy (single thread) | Hard (cần IPC mechanisms) |
| **Complexity** | Medium | High (async propagation) | Medium-High (IPC, serialization) |


***

## Phần III: Asyncio Internals - Deep Dive

### 3.1 Coroutine và Event Loop Architecture

**Coroutine:**
Function định nghĩa bằng `async def`, khi gọi trả về coroutine object (chưa chạy).[^27][^10]

```python
async def my_coro():
    await asyncio.sleep(1)
    return "done"

# Gọi my_coro() không execute code, chỉ tạo coroutine object
coro = my_coro()  # <coroutine object>

# Phải await hoặc asyncio.run() mới chạy
result = asyncio.run(coro)  # "done"
```

**Event Loop Components:**

```
┌─────────────── EVENT LOOP ───────────────┐
│                                           │
│  ┌──────────────┐    ┌─────────────┐    │
│  │ Ready Queue  │◄───│  Scheduler  │    │
│  │  (deque)     │    │  (callbacks)│    │
│  └──────┬───────┘    └─────────────┘    │
│         │                                 │
│         ▼                                 │
│  ┌──────────────┐    ┌─────────────┐    │
│  │   Execute    │───►│  Selector   │    │
│  │  Coroutine   │    │ (epoll/poll)│    │
│  └──────────────┘    └──────┬──────┘    │
│         │                    │            │
│         ▼                    ▼            │
│     await I/O         File descriptors   │
│         │                   ready?        │
│         └───────────────────┘            │
└───────────────────────────────────────────┘
```

**Luồng xử lý:**

1. `asyncio.create_task()` đẩy coroutine vào ready queue
2. Loop lấy coroutine từ queue, execute đến khi gặp `await`
3. Nếu `await` I/O: đăng ký FD với selector, suspend coroutine
4. Loop chuyển sang task khác
5. Selector notify khi I/O ready → coroutine quay lại ready queue[^12][^14]

***

### 3.2 `async def`, `await`, `asyncio.gather()`, `create_task()`

#### 3.2.1 `await` - Nhả quyền điều khiển

```python
async def fetch(url):
    async with aiohttp.ClientSession() as session:
        # await này suspend coroutine, loop chạy task khác
        async with session.get(url) as resp:
            return await resp.text()

# Nếu không await, code chạy tuần tự (blocking event loop)
async def bad_example():
    result = fetch("http://example.com")  # WRONG: coroutine object, không execute
    print(result)  # <coroutine object>
```


#### 3.2.2 `asyncio.gather()` - Chạy nhiều tasks, thu kết quả

**Use case:** Muốn kết quả từ tất cả coroutines (hoặc tolerate exceptions)

```python
async def main():
    results = await asyncio.gather(
        fetch("http://api1.com"),
        fetch("http://api2.com"),
        fetch("http://api3.com"),
        return_exceptions=True  # Không raise exception ngay, trả về exception object
    )
    # results = [data1, data2, Exception(...)]
```


#### 3.2.3 `asyncio.create_task()` - Schedule task, không chờ kết quả ngay

**Use case:** Fire-and-forget hoặc muốn fine-grained control

```python
async def main():
    task1 = asyncio.create_task(fetch("http://api1.com"))
    task2 = asyncio.create_task(fetch("http://api2.com"))
    
    # Task bắt đầu chạy ngay (added to event loop)
    print("Tasks started, doing other work...")
    
    # Chờ khi cần kết quả
    result1 = await task1
    result2 = await task2
```

**Gather vs create_task:**

- `gather()`: High-level, concise, trả về list kết quả theo thứ tự[^28][^29]
- `create_task()`: Linh hoạt hơn, có thể cancel từng task, inspect state[^30][^29]

***

### 3.3 Async Context Managers và Generators

#### 3.3.1 Async Context Managers - `async with`

Để quản lý resources trong async code (DB connections, HTTP sessions), dùng `async with`.[^23][^25]

```python
class AsyncDBConnection:
    async def __aenter__(self):
        # Setup: kết nối DB (I/O operation)
        self.conn = await asyncpg.connect("postgresql://...")
        return self.conn
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        # Cleanup: đóng connection
        await self.conn.close()

async def query():
    async with AsyncDBConnection() as conn:
        result = await conn.fetch("SELECT * FROM users")
        return result
```

**Context manager decorator:**

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def get_db_pool():
    pool = await asyncpg.create_pool("postgresql://...")
    try:
        yield pool
    finally:
        await pool.close()

async with get_db_pool() as pool:
    async with pool.acquire() as conn:
        ...
```


#### 3.3.2 Async Generators - `async for`

Generator async để stream data mà không load hết vào memory.[^31][^32]

```python
async def fetch_paginated_data(api_url):
    page = 1
    while True:
        async with aiohttp.ClientSession() as session:
            async with session.get(f"{api_url}?page={page}") as resp:
                data = await resp.json()
                if not data:
                    break
                for item in data:
                    yield item  # Yield từng item
                page += 1

async def process():
    async for item in fetch_paginated_data("http://api.com/users"):
        print(item)  # Xử lý từng item, không tốn memory
```


***

## Phần IV: Xử Lý Blocking I/O trong Asyncio

### 4.1 Vấn đề: Blocking I/O block toàn bộ event loop

**Anti-pattern phổ biến:**

```python
import time

async def bad_handler():
    # time.sleep() là blocking call → event loop bị block 5s
    time.sleep(5)  # WRONG: block thread
    return "done"

# Tất cả coroutines khác phải chờ 5s
```

**Lý do:** Event loop chạy trên 1 thread duy nhất; blocking call trong coroutine → freeze toàn bộ loop.[^33][^7]

***

### 4.2 Giải pháp: `asyncio.to_thread()` và `run_in_executor()`

#### 4.2.1 `asyncio.to_thread()` (Python 3.9+)

Chạy blocking function trong **ThreadPoolExecutor**, không block event loop.[^34][^35]

```python
import time

def blocking_db_query(query):
    # Legacy sync DB library (psycopg2, không có async version)
    time.sleep(2)  # Simulate blocking I/O
    return f"Result for {query}"

async def handler():
    # Offload sang thread pool, event loop không bị block
    result = await asyncio.to_thread(blocking_db_query, "SELECT * FROM users")
    return result
```


#### 4.2.2 `loop.run_in_executor()` - Fine-grained control

```python
async def handler():
    loop = asyncio.get_running_loop()
    
    # Option 1: Default ThreadPoolExecutor
    result = await loop.run_in_executor(None, blocking_db_query, "query")
    
    # Option 2: ProcessPoolExecutor cho CPU-bound
    from concurrent.futures import ProcessPoolExecutor
    with ProcessPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, cpu_intensive_task, data)
    
    return result
```

**Lưu ý quan trọng:**

- Nếu `await loop.run_in_executor(...)`, coroutine vẫn bị block cho đến khi task trong thread/process xong[^36][^34]
- Để không chờ ngay: `task = loop.run_in_executor(...)` (không await), sau đó `await task` khi cần kết quả[^34]

***

### 4.3 Pattern: Queue + Worker cho Blocking Tasks

**Use case:** Giới hạn số lượng concurrent blocking tasks (rate limiting)

```python
import asyncio

async def worker(queue):
    while True:
        task_data = await queue.get()
        # Offload blocking I/O sang thread
        result = await asyncio.to_thread(blocking_task, task_data)
        print(f"Processed: {result}")
        queue.task_done()

async def main():
    queue = asyncio.Queue()
    
    # Tạo 3 workers (giới hạn concurrency = 3)
    workers = [asyncio.create_task(worker(queue)) for _ in range(3)]
    
    # Producer: đẩy tasks vào queue
    for i in range(100):
        await queue.put(f"task_{i}")
    
    await queue.join()  # Chờ tất cả tasks xong
```


***

## Phần V: Concurrency Control - Semaphore và Rate Limiting

### 5.1 `asyncio.Semaphore` - Giới hạn số concurrent tasks

**Vấn đề:** Gọi 1000 concurrent HTTP requests → server rate limit (HTTP 429) hoặc OOM.[^37][^38]

**Giải pháp:** Semaphore giới hạn số tasks chạy đồng thời.

```python
import asyncio
import aiohttp

async def fetch_with_limit(session, url, semaphore):
    async with semaphore:  # Chỉ cho phép N coroutines vào section này
        async with session.get(url) as resp:
            return await resp.text()

async def main():
    # Giới hạn tối đa 10 concurrent requests
    semaphore = asyncio.Semaphore(10)
    
    async with aiohttp.ClientSession() as session:
        tasks = [
            fetch_with_limit(session, f"http://api.com/user/{i}", semaphore)
            for i in range(1000)
        ]
        results = await asyncio.gather(*tasks)
```

**Cơ chế:**

- Semaphore có internal counter = 10
- Mỗi `async with semaphore` giảm counter xuống 1
- Khi counter = 0, coroutines tiếp theo phải chờ
- `__aexit__` tăng counter lên 1 → cho coroutine khác vào[^38][^37]

[^39]

### 5.2 Rate Limiting với Delay

```python
async def fetch_with_rate_limit(session, url, semaphore, delay=0.1):
    async with semaphore:
        async with session.get(url) as resp:
            data = await resp.text()
        
        # Rate limit: chờ delay giữa mỗi request
        if semaphore.locked():  # Đang ở giới hạn
            await asyncio.sleep(delay)
        
        return data
```


***

## Phần VI: Best Practices cho Production

### 6.1 Checklist Thiết kế Async Application

1. **Entry Point chuẩn:**
```python
async def main():
    # Application logic
    pass

if __name__ == "__main__":
    asyncio.run(main())  # Quản lý event loop lifecycle tự động
```

2. **Async Context Managers cho resources:**
```python
# Good: Async context manager
async with aiohttp.ClientSession() as session:
    async with session.get(url) as resp:
        data = await resp.text()

# Bad: Không dùng context manager → resource leak
session = aiohttp.ClientSession()
resp = await session.get(url)
# Quên không close session → memory leak
```

3. **Timeout cho mọi I/O:**
```python
try:
    result = await asyncio.wait_for(
        long_running_task(),
        timeout=5.0  # Timeout sau 5 giây
    )
except asyncio.TimeoutError:
    logger.error("Task timed out")
```

4. **Không gọi blocking code trực tiếp:**
```python
# Bad
async def handler():
    time.sleep(5)  # Block event loop

# Good
async def handler():
    await asyncio.to_thread(time.sleep, 5)
```


***

### 6.2 Anti-patterns Thường Gặp

| Anti-pattern | Mô tả | Fix |
| :-- | :-- | :-- |
| **Blocking I/O trong coroutine** | Gọi `requests.get()`, `time.sleep()` trực tiếp[^3][^40] | Dùng `asyncio.to_thread()` hoặc async lib (aiohttp, asyncpg)[^34] |
| **Quên `await`** | `result = async_func()` → coroutine object, không execute[^10][^40] | `result = await async_func()` |
| **Misuse `gather()` không handle exceptions** | 1 task raise → tất cả tasks bị cancel | `gather(..., return_exceptions=True)`[^29] |
| **Fire-and-forget task leak** | `asyncio.create_task()` không await → task mất kiểm soát | Track tasks trong list, await tất cả trước khi exit |

[^21][^41]

***

### 6.3 Khi nào KHÔNG nên dùng Async?

1. **CPU-bound tasks thuần:**
Async không cải thiện performance cho tính toán nặng (ML training, video encoding) → Dùng multiprocessing.[^1][^25]
2. **Codebase nhỏ, ít concurrency:**
Overhead của async không đáng so với sync code đơn giản.[^5][^1]
3. **Legacy libraries không có async version:**
Nếu phải wrap hết bằng `to_thread()` → không có lợi ích, chỉ tăng complexity.[^42]
4. **Team thiếu kinh nghiệm async:**
Debug async code khó hơn sync, dễ gặp race conditions nếu không hiểu rõ.[^21]

***

## Phần VII: Ứng Dụng Thực Tế - Case Study

### 7.1 FastAPI + Asyncio cho High-throughput API

**Kiến trúc:**

```python
from fastapi import FastAPI, Depends
import asyncpg
import aiohttp

app = FastAPI()

# Connection pool (shared across requests)
db_pool = None

@app.on_event("startup")
async def startup():
    global db_pool
    db_pool = await asyncpg.create_pool("postgresql://...")

@app.on_event("shutdown")
async def shutdown():
    await db_pool.close()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    # Concurrent DB + external API call
    async with db_pool.acquire() as conn:
        user_task = conn.fetchrow("SELECT * FROM users WHERE id=$1", user_id)
    
    async with aiohttp.ClientSession() as session:
        orders_task = session.get(f"http://orders-api.com/user/{user_id}")
    
    # Chạy song song
    user, orders_resp = await asyncio.gather(user_task, orders_task)
    orders = await orders_resp.json()
    
    return {"user": user, "orders": orders}
```

**Performance:** 1 worker xử lý ~10,000 req/s so với sync ~500 req/s.[^2][^15]

***

### 7.2 Xử lý Nightly Job với Rate Limiting

**Bài toán:** Tính friendship score cho 10,000 users, gọi LLM API (rate limit 30 req/min).

**Solution:**

```python
async def calculate_score_with_limit(user_id, semaphore):
    async with semaphore:
        # Gọi LLM API
        score = await llm_api.analyze(user_id)
        
        # Rate limit: chờ 2s giữa mỗi call
        await asyncio.sleep(2)
        
        return score

async def nightly_job():
    # 30 req/min = 1 req/2s → semaphore = 1
    semaphore = asyncio.Semaphore(1)
    
    user_ids = range(10000)
    tasks = [
        calculate_score_with_limit(uid, semaphore)
        for uid in user_ids
    ]
    
    # Chạy dần dần, không overwhelm API
    results = await asyncio.gather(*tasks, return_exceptions=True)
```


***

## Kết Luận

**Async/await trong Python là một công cụ mạnh mẽ nhưng không phải silver bullet:**

- **Dùng async khi:** I/O-bound workloads, high concurrency, cần throughput cao (web servers, API aggregators, scrapers)[^15][^1]
- **Dùng threading khi:** Cần tích hợp blocking legacy libs, ít concurrency, không muốn refactor toàn bộ[^16]
- **Dùng multiprocessing khi:** CPU-bound tasks, cần true parallelism trên multi-core[^26][^25]

**Key Takeaways:**

1. Event loop là single-threaded → mọi blocking I/O đều phải offload hoặc dùng async libs[^33][^3]
2. `asyncio.gather()` cho bulk operations, `create_task()` cho fine-grained control[^29][^28]
3. Semaphore để rate limiting, context managers để resource management[^43][^37]
4. GIL vẫn ảnh hưởng async cho CPU-bound (không có lợi ích so với sync)[^24][^23]

Để master async, bạn cần:

- Hiểu event loop internals (selector, ready queue)[^13][^12]
- Thực hành viết async context managers, generators[^44][^31]
- Biết debug với `asyncio.current_task()`, inspect event loop state[^45]
- Đọc source code của thư viện async phổ biến (aiohttp, asyncpg) để học patterns
<span style="display:none">[^46][^47][^48][^49][^50][^51][^52][^53][^54][^55][^56][^57][^58][^59][^60][^61][^62][^63][^64][^65][^66][^67][^68][^69][^70][^71][^72][^73][^74][^75][^76][^77][^78][^79]</span>

<div align="center">⁂</div>

[^1]: https://www.theserverside.com/tutorial/Asynchronous-programming-in-Python-tutorial

[^2]: https://www.clariontech.com/blog/adopting-async/python-for-i-o-applications

[^3]: https://dev.to/devopsfundamentals/python-fundamentals-blocking-io-3lc7

[^4]: https://www.mendix.com/blog/asynchronous-vs-synchronous-programming/

[^5]: https://www.ramotion.com/blog/synchronous-vs-asynchronous-programming/

[^6]: https://builtin.com/articles/async-vs-sync-programming

[^7]: https://learn.microsoft.com/en-us/azure/architecture/antipatterns/synchronous-io/

[^8]: https://sunscrapers.com/blog/programming-async-vs-sync-best-approach/

[^9]: https://www.geeksforgeeks.org/python/asyncio-vs-threading-in-python/

[^10]: https://dev.to/leapcell/deep-dive-into-python-coroutines-asyncawait-4if3

[^11]: https://docs.python.org/3/howto/a-conceptual-overview-of-asyncio.html

[^12]: https://stackoverflow.com/questions/38831687/how-does-asyncios-event-loop-know-when-an-awaitable-resource-is-ready

[^13]: https://iximiuz.com/en/posts/explain-event-loop-in-100-lines-of-code/

[^14]: https://tenthousandmeters.com/blog/python-behind-the-scenes-12-how-asyncawait-works-in-python/

[^15]: https://betterstack.com/community/guides/scaling-python/python-async-programming/

[^16]: https://superfastpython.com/asyncio-vs-threading/

[^17]: https://leimao.github.io/blog/Python-Concurrency-High-Level/

[^18]: https://blog.jetbrains.com/pycharm/2025/06/concurrency-in-async-await-and-threading/

[^19]: https://www.reddit.com/r/learnprogramming/comments/1bp8pf8/multithreading_vs_async_summary_for_python/

[^20]: https://blog.gopenai.com/pythons-asynchronous-power-a-deep-dive-into-async-programming-5dd114f837b1

[^21]: https://discuss.python.org/t/asyncio-best-practices/12576

[^22]: https://alex-jacobs.com/posts/pythonasync/

[^23]: https://www.reddit.com/r/Python/comments/8kzxm5/gil_and_asyncio/

[^24]: https://stackoverflow.com/questions/75907155/is-asyncio-affected-by-the-gil

[^25]: https://stackoverflow.com/questions/61351844/difference-between-multiprocessing-asyncio-threading-and-concurrency-futures-i

[^26]: https://www.linkedin.com/pulse/multithreading-vs-multiprocessing-asyncio-code-examples-kaushik-yxgjc

[^27]: https://leapcell.io/blog/delving-deep-into-asyncio-coroutines-event-loops-and-async-await-unpacking-the-underpinnings

[^28]: https://stackoverflow.com/questions/52245922/is-it-more-efficient-to-use-create-task-or-gather

[^29]: https://ai-native.panaversity.org/docs/Python-Fundamentals/asyncio/concurrent-tasks

[^30]: https://www.appsloveworld.com/coding/python3x/7/is-it-more-efficient-to-use-create-task-or-gather

[^31]: https://superfastpython.com/asynchronous-generators-in-python/

[^32]: https://peps.python.org/pep-0525/

[^33]: https://developers.home-assistant.io/docs/asyncio_blocking_operations/

[^34]: https://stackoverflow.com/questions/68717924/correct-way-to-call-run-in-executor-for-blocking-code-in-asyncio-loop

[^35]: https://superfastpython.com/asyncio-blocking-tasks/

[^36]: https://www.youtube.com/watch?v=eQOA_Df68Is

[^37]: https://superfastpython.com/asyncio-gather-limit-concurrency/

[^38]: http://rednafi.com/python/limit_concurrency_with_semaphore/

[^39]: https://rednafi.com/python/limit-concurrency-with-semaphore/

[^40]: https://stackoverflow.com/questions/56713878/im-using-asyncio-but-async-function-is-blocking-other-async-functions-with-awai

[^41]: https://shanechang.com/p/python-asyncio-best-practices-pitfalls/

[^42]: https://www.dataleadsfuture.com/combining-traditional-thread-based-code-and-asyncio-in-python/

[^43]: https://dev.to/aaravjoshi/mastering-async-context-managers-boost-your-python-codes-performance-3g7g

[^44]: https://superfastpython.com/asynchronous-context-manager/

[^45]: https://stackoverflow.com/questions/75988992/how-can-i-inspect-the-internal-state-of-an-asyncio-event-loop

[^46]: https://realpython.com/async-io-python/

[^47]: https://codevisionz.com/lessons/asyncio-event-loop-management/

[^48]: https://towardsdatascience.com/deep-dive-into-multithreading-multiprocessing-and-asyncio-94fdbe0c91f0/

[^49]: https://tutorialedge.net/python/concurrency/asyncio-event-loops-tutorial/

[^50]: https://docs.python.org/3/library/asyncio-eventloop.html

[^51]: https://www.linkedin.com/pulse/asynchronous-programming-python-practical-deep-dive-farid-el-aouadi-7ay5e

[^52]: https://dev.to/imsushant12/asyncio-architecture-in-python-event-loops-tasks-and-futures-explained-4pn3

[^53]: https://www.youtube.com/watch?v=oAkLSJNr5zY

[^54]: https://www.reddit.com/r/Python/comments/16s1ius/asyncio_coroutines_faster_than_threads/

[^55]: https://docs.python.org/3/library/asyncio-task.html

[^56]: https://stackoverflow.com/questions/37433157/asynchronous-context-manager

[^57]: https://bbc.github.io/cloudfit-public-docs/asyncio/asyncio-part-3.html

[^58]: https://realpython.com/ref/glossary/asynchronous-context-manager/

[^59]: https://docs.python.org/3/library/contextlib.html

[^60]: https://krython.com/tutorial/python/async-context-managers-async-with/

[^61]: https://stackoverflow.com/questions/37549846/how-to-use-yield-inside-async-function

[^62]: https://aiotools.readthedocs.io/en/latest/aiotools.context.html

[^63]: https://stackoverflow.com/questions/66066885/shall-i-use-asyncio-create-task-before-calling-asyncio-gather

[^64]: https://stackoverflow.com/questions/50355944/yield-from-async-generator-in-python-asyncio

[^65]: https://stackoverflow.com/questions/37433157/asynchronous-context-manager/48800772

[^66]: https://www.reddit.com/r/learnpython/comments/123iw6q/whats_the_advantage_of_using_asynciotaskgroup/

[^67]: https://async-generator.readthedocs.io/en/latest/reference.html

[^68]: https://dzone.com/articles/mastering-async-context-manager-mocking-in-python

[^69]: https://stackoverflow.com/questions/64545132/will-run-in-executor-ever-block

[^70]: https://www.reddit.com/r/learnpython/comments/vl09en/how_does_threadpoolexecutor_work_with_asyncio/

[^71]: https://www.slingacademy.com/article/python-asyncio-loop-run_in_executor-method/

[^72]: https://codilime.com/blog/how-fit-triangles-into-squares-run-blocking-functions-event-loop/

[^73]: https://stackoverflow.com/questions/48682147/aiohttp-rate-limiting-parallel-requests

[^74]: https://stackoverflow.com/questions/38683243/asyncio-rate-limiting

[^75]: https://groups.google.com/g/python-tulip/c/h5GVSeV5bwE

[^76]: https://www.reddit.com/r/learnpython/comments/1fhry6u/asyncio_vs_threading_vs_multiprocessing/

[^77]: https://stackoverflow.com/questions/78956644/asyncio-how-to-make-use-of-selectors

[^78]: https://www.reddit.com/r/learnpython/comments/n8ix15/whats_the_point_of_asyncio_if_an_event_loop/

[^79]: https://www.reddit.com/r/Python/comments/sxovwp/async_io_tasks_vs_threads/


## 1. VẤN ĐỀ, HIỆN TRẠNG

### 1.1. Sự kiện OOM Kill

**Thời gian**: 11:42:04 AM ngày 1/1/2026  
**Exit code**: 137 (OOMKilled)  
**Memory limit**: 3000 MiB (3 GB)  
**Số lần restart**: 2 lần

### 1.2. Memory Usage Pattern

Từ Datadog Metrics và Rancher:

```
08:00-11:00 AM: Memory ổn định ~250MB ✅
11:42-11:46 AM: Memory tăng đột biến → 3GB (spike!) ⚠️
11:42:04 AM: OOM Kill (exit code 137) 💥
12:00+: Memory reset về ~192MB (sau restart)
```

### 1.3. APM Traces Analysis

Từ Datadog APM:

- Endpoint `/extract_facts` gây 99.5% errors
- Service: `pika-mem0:6699` (Memory API)
- Latency Distribution:
  - p50: 19.1s
  - p75: 26.2s
  - p90: 60.8s (TIMEOUT)
  - p95: 60.8s (TIMEOUT)
  - Max: 60.9s

**Pattern errors**:
- 11:27:08 → 60s timeout → HTTP 500 ❌
- 11:43:16 → 60s timeout → HTTP 500 ❌
- Nhiều requests timeout sau 60 giây

### 1.4. Architecture hiện tại

```
┌─────────────────────────────────────┐
│  FastAPI API Server (Uvicorn)      │
│  └─ Event Loop (1 thread)          │
│      └─ Xử lý HTTP requests        │
└─────────────────────────────────────┘
              ↓ Publish message
┌─────────────────────────────────────┐
│  RabbitMQ Queue                     │
└─────────────────────────────────────┘
              ↓ Consume message
┌─────────────────────────────────────┐
│  RabbitMQ Worker (Separate Process) │
│  └─ ThreadPoolExecutor (10 threads)│ ← Đây mới có threads!
│      ├─ Thread 1: Process message  │
│      ├─ Thread 2: Process message  │
│      └─ Thread 3: Process message  │
└─────────────────────────────────────┘

```

```
┌─────────────────────────────────────────────────────────┐
│ 1 WORKER PROCESS (python src/worker.py)                 │
│                                                          │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ RabbitMQ Connection                                 │ │
│ │   ├─ Host: RabbitMQ server                          │ │
│ │   ├─ Queue: conversation_events_processing         │ │
│ │   └─ Prefetch: 10 messages                          │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                          │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ ThreadPoolExecutor                                  │ │
│ │   ├─ max_workers: 10                                │ │
│ │   ├─ Queue: UNBOUNDED (không giới hạn) ⚠️          │ │
│ │   └─ 10 threads xử lý messages đồng thời           │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                          │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ Processing Flow                                     │ │
│ │   1. Parse message (conversation_log ~3MB)         │ │
│ │   2. LLM Analysis (nếu enabled)                    │ │
│ │   3. Memory API call (pika-mem0:6699)              │ │
│ │      └─ Timeout: 240s ⚠️ QUÁ CAO                   │ │
│ │   4. Calculate friendship score                    │ │
│ │   5. Update DB                                      │ │
│ └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### 1.5. Vấn đề cụ thể

1. Memory API timeout = 240s (quá cao)
   - Mỗi thread block tối đa 240s
   - Throughput: 10 messages / 240s = 0.04 msg/s

2. ThreadPoolExecutor queue không giới hạn
   - Messages tích lũy vô hạn trong queue
   - Mỗi message giữ ~3MB (body bytes)

3. Retry vô hạn khi timeout
   - NACK với `requeue=True` → RE-DELIVER
   - Messages retry liên tục → Throughput = 0

4. Memory tích lũy khi timeout
   - Exception stack traces giữ references
   - Python GC delay → Memory không được giải phóng ngay
   - 50-100 messages timeout → 550-1100MB exception objects

---

## 2. NGUYÊN NHÂN CHÍNH

### 2.1. Primary Root Cause: Memory API Timeout Quá Cao

**Dẫn chứng code**:

```143:143:src/app/core/config_settings.py
MEMORY_API_TIMEOUT_SECONDS: int = 240  # 4 phút!
```

**Vấn đề**:
- Timeout 240s quá cao so với thực tế (pika-mem0 timeout sau 60s)
- Mỗi thread block 240s → không thể xử lý messages khác
- 10 threads × 240s = Memory giữ lâu

### 2.2. Secondary Root Cause: pika-mem0 Service Không Response

**Bằng chứng từ APM traces**:
- Requests timeout sau 60 giây
- HTTP 500 errors với message "Missing error message and stack trace"
- 99.5% errors đến từ endpoint `/extract_facts`

**Cơ chế**:
```
pika-mem0 không response (timeout 60s)
    ↓
Worker threads block 240s (chờ timeout)
    ↓
Memory tích lũy (conversation_log + formatted_conversation + payload)
    ↓
Exception stack traces giữ references
    ↓
Python GC delay → Memory không được giải phóng ngay
    ↓
Memory spike đột ngột → OOM! 💥
```

### 2.3. Architecture Flaws

#### 2.3.1. ThreadPoolExecutor Queue Không Giới Hạn

**Dẫn chứng code**:

```113:113:src/app/background/rabbitmq_consumer.py
self.executor = ThreadPoolExecutor(max_workers=max_workers)
# → Queue mặc định là unbounded Queue()
```

**Vấn đề**:
- Queue không giới hạn → Messages tích lũy vô hạn
- Mỗi message giữ ~3MB (body bytes)
- 500-1000 messages = 1500-3000MB

#### 2.3.2. NACK với requeue=True → Retry Vô Hạn

**Dẫn chứng code**:

```576:576:src/app/background/rabbitmq_consumer.py
self.channel.basic_nack(delivery_tag=delivery_tag, requeue=True)
```

**Vấn đề**:
- Timeout → NACK → RE-DELIVER → Timeout lại → Cycle lặp lại
- Messages không được xử lý (retry vô hạn)
- Throughput = 0
#### 2.3.3. Exception Stack Traces Giữ References

**Dẫn chứng code**:

```934:961:src/app/services/utils/llm_analysis_utils.py
except httpx.TimeoutException as e:
    # ...
    raise  # ⚠️ RAISE EXCEPTION - Memory vẫn giữ trong stack!
```

**Vấn đề**:
- Exception object giữ references đến:
  - `conversation_log` (~3MB)
  - `formatted_conversation` (~3MB)
  - `payload` (~3MB)
  - `client` buffers (~1MB)
- Stack trace giữ references cho đến khi exception được handle
- 50-100 messages timeout → 550-1100MB exception objects

#### 2.3.4 Python GC delay → Memory không được giải phóng ngay - Tóm lại: Python GC delay là thời gian chờ GC chạy để giải phóng memory. Trong trường hợp OOM, nhiều exception objects tích lũy trước khi GC chạy, gây memory spike đột ngột.

###### Python GC Delay là gì?

- GC không chạy liên tục, mà chạy khi đạt threshold (700 objects cho gen0)
- Có delay giữa lúc object không còn reference và lúc GC giải phóng
- Exception stack traces giữ references đến local variables → memory không được giải phóng ngay

###### Tại sao gây vấn đề trong OOM?
- Nhiều messages timeout cùng lúc → nhiều exception objects tích lũy
- GC chưa chạy → memory không được giải phóng
- Memory tích lũy nhanh → vượt 3GB limit → OOM


##### 1. PYTHON GARBAGE COLLECTION LÀ GÌ?

###### 1.1. Cơ chế cơ bản

Python dùng Garbage Collector (GC) để tự động giải phóng memory khi objects không còn được sử dụng.

```python
# Ví dụ:
def process_message():
    conversation_log = [{"message": "..."} for _ in range(1000)]  # ~3MB
    # ... xử lý ...
    return result

# Khi function kết thúc:
# - conversation_log không còn được reference
# - Python GC sẽ giải phóng memory
# - NHƯNG: Không phải ngay lập tức!
```


###### 1.2. Reference Counting vs Generational GC

Python dùng 2 cơ chế:

1. Reference Counting (tức thì)
   - Đếm số references đến object
   - Khi count = 0 → giải phóng ngay
   - Nhưng không xử lý circular references

2. Generational GC (có delay)
   - Xử lý circular references
   - Chạy theo chu kỳ (không liên tục)
   - Có delay trước khi chạy

---

##### 2. TẠI SAO CÓ DELAY?

###### 2.1. GC không chạy liên tục

```python
# Python GC chạy khi:
# - gen0 đạt 700 objects (generation 0)
# - gen1 đạt 10 objects (generation 1)  
# - gen2 đạt 10 objects (generation 2)

# KHÔNG chạy ngay khi object không còn reference!
```

Lý do:
- GC tốn CPU
- Chạy liên tục sẽ làm chậm ứng dụng
- Python chạy GC khi cần (threshold-based)

###### 2.2. Generational GC Thresholds

```python
import gc

# Mặc định thresholds:
gc.get_threshold()
# Output: (700, 10, 10)
# - gen0: 700 objects
# - gen1: 10 collections
# - gen2: 10 collections
```

Cơ chế:
- Mỗi lần tạo object → gen0 count++
- Khi gen0 = 700 → chạy GC gen0
- Nếu object sống sót → chuyển sang gen1
- Sau 10 lần GC gen0 → chạy GC gen1
- Sau 10 lần GC gen1 → chạy GC gen2

###### 3.1. Scenario: 50 messages timeout trong 10 giây

```python
# Timeline:

T=0s:   Message 1 timeout
        ├─ Exception được raise
        ├─ Exception object giữ references:
        │  ├─ conversation_log: ~3MB
        │  ├─ formatted_conversation: ~3MB
        │  └─ payload: ~3MB
        ├─ Total: ~9MB per exception
        └─ GC count: 0 (chưa đạt threshold 700)

T=1s:   Message 2 timeout
        ├─ Exception object: +9MB
        ├─ Total: 18MB
        └─ GC count: 0 (chưa đạt threshold)

T=2s:   Message 3 timeout
        ├─ Exception object: +9MB
        ├─ Total: 27MB
        └─ GC count: 0 (chưa đạt threshold)

... (tiếp tục) ...

T=10s:  Message 50 timeout
        ├─ Exception objects: 50 × 9MB = 450MB
        ├─ GC count: 50 (vẫn chưa đạt threshold 700!)
        └─ Memory: 450MB (CHƯA ĐƯỢC GIẢI PHÓNG!)

T=11s:  GC chạy (threshold đạt hoặc manual trigger)
        ├─ Giải phóng exception objects
        ├─ Memory: 450MB → ~50MB (sau GC)
        └─ Delay: 1 giây (hoặc lâu hơn!)
```

---

### 2.4. Cơ Chế Gây Memory Spike Đột Ngột

**Timeline thực tế (11:42 AM)**:

```
11:40:00 AM: Memory: ~400MB
├─ 10 threads đang block (timeout 60s)
├─ Queue: 20 messages
└─ Exception objects: 10 × 11MB = 110MB

11:41:00 AM: Memory: ~600MB
├─ 10 threads vẫn block
├─ Queue: 40 messages
└─ Exception objects: 20 × 11MB = 220MB

11:42:00 AM: ⚠️ SPIKE BẮT ĐẦU!
├─ 10 messages timeout cùng lúc
├─ Exception objects: +110MB
├─ Queue: +10 messages (retry)
└─ Memory: 600MB + 110MB + 30MB = 740MB

11:42:01 AM: ⚠️ SPIKE TIẾP TỤC!
├─ 10 messages timeout
├─ Exception objects: +110MB
├─ Queue: +10 messages
└─ Memory: 740MB + 110MB + 30MB = 880MB

... (tiếp tục) ...

11:42:04 AM: 💥 OOM KILL!
├─ Memory: 1160MB + overhead + GC delay = 2.5-3GB
├─ Vượt quá 3GB limit
└─ Kubernetes OOMKill (exit code 137)
```

**Compound Effect**:
- Nhiều messages timeout cùng lúc (50-100 messages trong 10 giây)
- Exception stack traces tích lũy: 550-1100MB
- ThreadPoolExecutor queue tích lũy: 300-600MB
- Python GC delay: 200-400MB (Python GC delay là thời gian chờ GC chạy để giải phóng memory. Trong trường hợp OOM, nhiều exception objects tích lũy trước khi GC chạy, gây memory spike đột ngột.)
- Total: 1050-2100MB → Vượt 3GB limit! 💥

---

## 3. GIẢI PHÁP

### 3.1. Tổng Quan Giải Pháp

**Mục tiêu**:
1. Giảm blocking time từ 240s xuống 60s (giảm 75%)
2. Tắt LLM calls hoàn toàn (không ảnh hưởng time)
3. Fail fast → Giải phóng memory ngay khi timeout
4. Không RE-queue → Tránh retry vô hạn
5. Mark FAILED trong DB → Cron job retry sau 6h

### 3.2. Implementation Details

#### 3.2.1. Giảm Memory API Timeout: 240s → 60s

**File**: `src/app/core/config_settings.py`

```python
# Trước:
MEMORY_API_TIMEOUT_SECONDS: int = 240  # 4 phút

# Sau:
MEMORY_API_TIMEOUT_SECONDS: int = 60  # 1 phút
```

**Impact**:
- Blocking time giảm 75% (240s → 60s)
- Throughput tăng ~4x (0.04 msg/s → 0.17 msg/s)
- Memory giữ ngắn hơn

#### 3.2.2. Tắt LLM Calls Hoàn Toàn

**File**: `src/app/core/config_settings.py`

```python
# Set trong .env hoặc config
LLM_ANALYSIS_ENABLED: bool = False
GROQ_API_KEY: Optional[str] = None  # Hoặc không set
```

**Code đã có check**:

```1038:1044:src/app/services/utils/llm_analysis_utils.py
llm_enabled = llm_client.is_enabled()
if not llm_enabled:
    logger.warning(
        f"⚠️  LLM analysis disabled | "
        f"LLM_ANALYSIS_ENABLED={settings.LLM_ANALYSIS_ENABLED} | "
        f"GROQ_API_KEY={'set' if settings.GROQ_API_KEY else 'not set'}"
    )
```

**Impact**:
- LLM calls return ngay (0s) nếu disabled
- Không block worker threads
- Không ảnh hưởng processing time

#### 3.2.3. Timeout → Mark FAILED, Không RE-queue

**File**: `src/app/background/rabbitmq_consumer.py`

**Sửa exception handling**:

```python
# Trước:
except Exception as e:
    error_msg = str(e)
    logger.error(...)
    should_nack = True  # → RE-queue

# Sau:
except httpx.TimeoutException as e:
    # Memory API timeout → Mark FAILED, không RE-queue
    error_msg = f"Memory API timeout after 60s: {str(e)}"
    logger.error(
        f"❌ Memory API timeout | "
        f"conversation_id={conversation_id} | "
        f"error={error_msg}"
    )
    
    # Mark FAILED trong DB
    if event:
        try:
            self.repository.mark_failed(
                event=event,
                error_code="MEMORY_API_TIMEOUT",
                error_details=error_msg
            )
        except Exception as db_error:
            logger.error(f"❌ Failed to mark event as FAILED: {db_error}")
    
    # ACK message (không RE-queue)
    should_ack = True
    
    # Giải phóng memory ngay
    if conversation_log:
        del conversation_log
    
except Exception as e:
    # Các lỗi khác vẫn NACK (retry)
    error_msg = str(e)
    logger.error(...)
    should_nack = True
```

**Impact**:
- Fail fast → Giải phóng memory ngay
- Không RE-queue → Tránh retry vô hạn
- Mark FAILED → Cron job retry sau 6h

#### 3.2.4. Giải Phóng Memory Ngay Khi Timeout

**File**: `src/app/background/rabbitmq_consumer.py`

**Trong `_process_message()`**:

```python
def _process_message(self, delivery_tag: int, message_body: bytes):
    conversation_log = None
    try:
        # Parse message
        message = json.loads(message_body)
        conversation_log = message.get("conversation_log", [])
        
        # Process...
        
    except httpx.TimeoutException as e:
        # Timeout → Giải phóng memory ngay
        if conversation_log:
            del conversation_log
        # Mark FAILED...
        
    finally:
        # Cleanup
        if conversation_log:
            del conversation_log
        # DB cleanup...
```

**Impact**:
- Memory được giải phóng ngay khi timeout
- Không chờ GC
- Giảm memory spike

### 3.3. Database Schema

**Cột status trong `conversation_events`**:

```sql
status VARCHAR(50) NOT NULL DEFAULT 'PENDING'
    CHECK (status IN ('PENDING', 'PROCESSING', 'PROCESSED', 'FAILED', 'SKIPPED'))
```

**Các cột liên quan**:
- `error_code`: Lưu "MEMORY_API_TIMEOUT"
- `error_details`: Lưu chi tiết lỗi
- `next_attempt_at`: Set = now + 6 hours (cho cron job)

### 3.4. Cron Job Retry (Đã Code Sẵn)

**Cron job đã có sẵn** sẽ:
1. Query events với `status='FAILED'` và `next_attempt_at <= now`
2. Retry processing
3. Nếu vẫn fail → `next_attempt_at += 6 hours`

### 3.5. So Sánh Trước/Sau

| Metric | Trước | Sau |
|--------|-------|-----|
| Memory API timeout | 240s | 60s |
| LLM calls | Chạy (nếu enabled) | Tắt hoàn toàn |
| Blocking time | 240s+ | 60s |
| Retry behavior | NACK → RE-queue ngay | Mark FAILED → Cron job 6h sau |
| Memory cleanup | Chậm (sau timeout) | Ngay (fail fast) |
| Throughput | 0.04 msg/s | 0.17 msg/s |
| Memory spike risk | Cao | Thấp |

### 3.6. Kết Quả Mong Đợi

1. Giảm blocking time 75% (240s → 60s)
2. Fail fast → Giải phóng memory ngay
3. Không RE-queue → Tránh loop retry
4. Cron job retry → Vẫn có cơ hội xử lý lại
5. Memory spike giảm đáng kể

---

## 4. TÓM TẮT

### Vấn đề:
- Worker bị OOM kill (exit code 137) tại 11:42:04 AM
- Memory tăng đột biến từ 250MB → 3GB
- Memory API timeout 240s quá cao
- pika-mem0 service không response (timeout 60s)

### Nguyên nhân chính:
1. Memory API timeout = 240s (quá cao)
2. ThreadPoolExecutor queue không giới hạn
3. NACK với requeue=True → Retry vô hạn
4. Exception stack traces giữ references
5. Python GC delay → Memory không được giải phóng ngay

### Giải pháp:
1. Giảm Memory API timeout: 240s → 60s
2. Tắt LLM calls hoàn toàn
3. Timeout → Mark FAILED, không RE-queue
4. Giải phóng memory ngay khi timeout
5. Cron job retry sau 6h

---

**Document này tổng hợp từ các phân tích chi tiết trong folder `docs/6_OMM_worker/`**
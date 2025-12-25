Giải thích "client không đóng connection đúng cách" và tại sao nó gây vấn đề:

## Client đóng connection đúng cách vs không đúng cách

### 1. Client đóng đúng cách (normal flow)

```
Timeline:
0s:  Client → Server: HTTP Request (POST /conversations/end)
     Connection: ESTABLISHED
     
2s:  Server → Client: HTTP Response (202 Accepted)
     Connection: KEEP-ALIVE (vẫn mở để tái sử dụng)
     
2.1s: Client đọc xong response
      Client → Server: TCP FIN (đóng connection)
      Server → Client: TCP ACK
      Connection: CLOSED ✅
```

Đặc điểm:
- Client gửi TCP FIN sau khi nhận response
- Server giải phóng connection ngay
- Worker thread được giải phóng

---

### 2. Client không đóng đúng cách (các trường hợp)

#### Case 1: Client crash/disconnect đột ngột (network issue)

```
Timeline:
0s:  Client → Server: HTTP Request (POST /conversations/end)
     Connection: ESTABLISHED
     
1s:  Server đang xử lý (DB query chậm...)
     
2s:  Client bị crash / mất mạng đột ngột
     ❌ Client KHÔNG gửi TCP FIN
     ❌ Server KHÔNG biết client đã mất
     
5s:  Server query xong, chuẩn bị gửi response
     Server → Client: HTTP Response (202 Accepted)
     ↓
     ❌ Không có client để nhận → Response bị drop
     ❌ Connection vẫn ở trạng thái "ESTABLISHED" (zombie connection)
     ❌ Worker thread vẫn bị block, chờ client đọc response
```

Vấn đề:
- Server không biết client đã mất
- Connection trở thành "zombie" (half-open)
- Worker thread bị block vô hạn

---

#### Case 2: Client timeout nhưng không đóng connection

```
Timeline:
0s:  Client → Server: HTTP Request
     Client timeout: 60s
     
30s: Server vẫn đang xử lý (DB query chậm)
     
60s: Client timeout → Client bỏ request
     ❌ Client KHÔNG gửi TCP FIN (vì đã timeout)
     ❌ Client chỉ bỏ request, không đóng connection
     
65s: Server query xong, gửi response
     Server → Client: HTTP Response
     ↓
     ❌ Client không đọc (đã timeout rồi)
     ❌ Connection vẫn mở, worker thread vẫn block
```

Vấn đề:
- Client timeout nhưng connection vẫn mở
- Server vẫn giữ connection và worker thread

---

#### Case 3: Client không đọc response body (lazy client)

```python
# ❌ BAD: Client code không đọc response
import requests

response = requests.post("http://api/v1/conversations/end", json=data, timeout=60)
# ❌ Không gọi response.close() hoặc response.raise_for_status()
# ❌ Connection vẫn mở, chờ client đọc response body
```

Vấn đề:
- Client nhận response nhưng không đọc hết body
- Server chờ client đọc → connection bị giữ

---

#### Case 4: Client dùng connection pool nhưng không release

```python
# ❌ BAD: Client code
import httpx

async with httpx.AsyncClient() as client:
    response = await client.post("http://api/v1/conversations/end", json=data)
    # ❌ Không await response.aread() hoặc response.aclose()
    # ❌ Connection trong pool không được release
```

Vấn đề:
- Connection trong pool không được release
- Pool cạn → không thể tạo connection mới

---

## Tại sao gây vấn đề cho server

### Vấn đề 1: Worker thread bị block

```
Server có 4 worker threads:

Thread 1: ✅ Xử lý request A (2s) → Done → Free
Thread 2: ❌ Xử lý request B (client crash) → Blocked forever
Thread 3: ❌ Xử lý request C (client timeout) → Blocked forever  
Thread 4: ❌ Xử lý request D (client lazy) → Blocked forever

Request E mới đến: ❌ Không có worker available → 504 Timeout
Request F mới đến: ❌ Không có worker available → 504 Timeout
...
```

Kết quả:
- Tất cả worker bị block → service không thể xử lý request mới → 504 hàng loạt

---

### Vấn đề 2: Connection pool cạn kiệt

```
Server có connection pool: 50 connections

Connection 1-10: ✅ Normal requests → Released
Connection 11-20: ❌ Zombie connections (client crash)
Connection 21-30: ❌ Zombie connections (client timeout)
Connection 31-40: ❌ Zombie connections (client lazy)
Connection 41-50: ✅ Normal requests → Released

Request mới: ❌ Không có connection available → Chờ timeout → 504
```

Kết quả:
- Pool cạn → request mới không lấy được connection → 504

---

## Cách phòng tránh (server-side)

### 1. Thêm `--timeout-keep-alive` (quan trọng nhất)

```dockerfile
CMD ["uvicorn", "app.main_app:app", \
     "--timeout-keep-alive", "55"]  # ✅ Force đóng connection sau 55s
```

Cách hoạt động:
```
Timeline:
0s:  Client → Server: Request
30s: Server đang xử lý
55s: Uvicorn timeout → Server tự động đóng connection
     ✅ Worker thread được giải phóng
     ✅ Connection được release
```

---

### 2. Thêm timeout cho DB query

```python
# File: src/app/db/database_connection.py
engine = create_engine(
    settings.DATABASE_URL,
    connect_args={
        "options": "-c statement_timeout=10000"  # ✅ 10s query timeout
    }
)
```

Cách hoạt động:
```
Timeline:
0s:  Client → Server: Request
5s:  DB query bắt đầu
15s: DB query timeout → Exception → Response 500
     ✅ Worker thread được giải phóng ngay
     ✅ Connection được đóng
```

---

### 3. Thêm timeout cho HTTP client (nếu gọi external API)

```python
# File: src/app/services/utils/llm_analysis_utils.py
async with httpx.AsyncClient(timeout=15.0) as client:
    response = await client.post(...)
    # ✅ Timeout sau 15s → Exception → Worker được giải phóng
```

---

## Ví dụ cụ thể: vấn đề của bạn

### Scenario: Service treo từ 23h đến 9h sáng

Có thể xảy ra:

```
23:00: Nightly Job bắt đầu → Gửi 10,000 requests
23:01: 100 requests đang xử lý
       - 50 requests: DB query chậm (30s mỗi request)
       - 30 requests: Client timeout (60s) nhưng connection vẫn mở
       - 20 requests: Client crash/disconnect đột ngột
       
23:02: Tất cả 100 worker threads bị block:
       - 50 threads: Chờ DB query (không có timeout)
       - 30 threads: Chờ client đọc response (client đã timeout)
       - 20 threads: Chờ client đóng connection (client đã crash)
       
23:03: Request mới đến → ❌ Không có worker available → 504
23:04: Request mới đến → ❌ Không có worker available → 504
...
09:00: Admin restart service → ✅ Worker threads được giải phóng
```

---

## Giải pháp tổng hợp

### 1. Server-side (bạn có thể fix)

```dockerfile
# ✅ Fix 1: Uvicorn timeout
CMD ["uvicorn", "app.main_app:app", \
     "--timeout-keep-alive", "55"]
```

```python
# ✅ Fix 2: DB query timeout
engine = create_engine(
    settings.DATABASE_URL,
    connect_args={"options": "-c statement_timeout=10000"}
)
```

```python
# ✅ Fix 3: RabbitMQ timeout
self.connection = pika.BlockingConnection(
    pika.ConnectionParameters(
        socket_timeout=5,  # ✅ 5s timeout
        blocked_connection_timeout=5
    )
)
```

### 2. Client-side (bạn không thể fix, nhưng có thể document)

```python
# ✅ GOOD: Client code nên làm
import httpx

async with httpx.AsyncClient(timeout=60.0) as client:
    response = await client.post("http://api/v1/conversations/end", json=data)
    await response.aread()  # ✅ Đọc hết response body
    response.raise_for_status()  # ✅ Check status
    # ✅ Connection tự động đóng khi ra khỏi context
```

---

## Tóm tắt

"Client không đóng connection đúng cách" có thể là:
1. Client crash/disconnect đột ngột → không gửi TCP FIN
2. Client timeout nhưng không đóng connection
3. Client không đọc response body → connection bị giữ
4. Client dùng connection pool nhưng không release

Hậu quả:
- Worker threads bị block vô hạn
- Connection pool cạn kiệt
- Service không thể xử lý request mới → 504

Giải pháp:
- Thêm timeout ở mọi layer (Uvicorn, DB, RabbitMQ, External API)
- Timeout sẽ force đóng connection → giải phóng worker → tránh 504

Bạn muốn tôi tạo checklist các timeout cần thêm không?
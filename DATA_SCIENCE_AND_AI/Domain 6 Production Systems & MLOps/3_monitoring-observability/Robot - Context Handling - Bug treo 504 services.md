
# 1. MECE? 

```
Chưa hiểu lắm

1. MECE toàn bộ các yếu tố ảnh hưởng đến 2 đầu API

- API suggest @.sh (59-66)

- API end @.sh (148-182)
```


```
Mình có đang xử lý cái gì đó gây ra block crash treo 504 ko ?
```


Mình sẽ rà các endpoint chính, service nặng (LLM, alerts, external calls, DB/Redis, RabbitMQ) để xem có chỗ nào block lâu, timeout cao, hoặc chạy đồng bộ trong request có thể gây 504/treo, rồi tóm lại cho bạn phần nào là “risk cao”.

[14 tools called]

### Ngắn gọn: Có chỗ **có thể gây treo / 504**, nhưng **LLM & alert Google Chat gần như không phải thủ phạm chính**. Rủi ro lớn nhất là **DB / RabbitMQ**.

---

### 1. Những thứ nặng **không nằm trực tiếp trong HTTP request** (ít khả năng gây 504)

- **LLM + Memory API (`groq`, `Mem0`)**  
  - Được gọi qua `analyze_conversation_with_llm` trong `FriendshipScoreCalculationService`, nhưng:
    - Chỉ chạy trong **worker / background** (job xử lý event), không nằm trong `/conversations/end`.
    - Có **circuit breaker** + **timeout tổng** `PARALLEL_ANALYSIS_TIMEOUT_SECONDS` (default 180s) và từng task có giới hạn.
  - Nên LLM có chậm / timeout cũng **không chặn** HTTP request chính.

- **Alert Google Chat (`AlertManager`, `GoogleChatClient`)**  
  - Gửi HTTP ra ngoài bằng `aiohttp` với:
    - `timeout` mặc định `5s`, `max_retries=3`, backoff tối đa vài giây.
  - Được gọi qua `send_alert_fire_and_forget()` → dùng `asyncio.create_task(...)` → **fire‑and‑forget**, **không chặn response**.

=> Hai mảng này **đã được thiết kế để không block request**.

---

### 2. Những chỗ **có khả năng gây treo / 504**

#### a) **Database (Postgres) – connection pool & timeout**

- `database_connection.py`:

```python
engine = create_engine(
    settings.DATABASE_URL,
    pool_pre_ping=True,
    pool_size=settings.DB_POOL_SIZE,        # default 50
    max_overflow=settings.DB_MAX_OVERFLOW,  # default 100
    pool_timeout=settings.DB_POOL_TIMEOUT,  # default 30s
    pool_recycle=settings.DB_POOL_RECYCLE,
)
```

- Nếu:
  - DB chậm / down / network issue, hoặc
  - Pool bị **exhausted** (quá nhiều request cùng lúc, connection không được trả về),
- Thì **mỗi request mới** lấy DB session (`SessionLocal()`) có thể:
  - Chờ tới **`DB_POOL_TIMEOUT` (mặc định 30s)** rồi mới raise lỗi.
  - Với Nginx / API Gateway timeout nhỏ hơn (ví dụ 10–15s) → **504 Gateway Timeout** bên ngoài.

Đặc biệt các endpoint **phụ thuộc DB**:
- `/v1/conversations/end` → `ConversationEventService.create_event(...)`  
- `/v1/activities/suggest` → `ActivitySuggestionService.get_suggestions(...)`  
- `/v1/friendship_status/...` / `calculate-score-and-update` …  
- `/v1/health` (check DB bằng `SELECT 1`)

Nếu dưới tải cao hoặc DB có sự cố, các route này **có thể treo đến ~30s / request**.

#### b) **RabbitMQ publisher trong `/conversations/end`**

- Endpoint:

```python
data = service.create_event(request)          # DB write
publish_success = await publish_conversation_event(...)  # RabbitMQ
```

- `publish_conversation_event` gọi `RabbitMQPublisher.publish(...)` dùng **`pika.BlockingConnection`**:

```python
self.connection = pika.BlockingConnection(
    pika.ConnectionParameters(
        host=...,
        port=...,
        credentials=...,
        connection_attempts=3,
        retry_delay=2,
        # không set socket_timeout
    )
)
```

- Nếu **RabbitMQ down / không reachable**:
  - `BlockingConnection` + `connection_attempts=3` + `retry_delay=2` + TCP timeout mặc định → có thể mất **vài giây đến hàng chục giây** trước khi ném exception.
  - Dù lỗi cuối cùng **không raise ra endpoint** (bị catch trong `publish_conversation_event`), nhưng quá trình connect/publish **vẫn chặn request** trong thời gian đó.
  - Điều này có thể góp phần gây **504** nếu gateway timeout thấp.

#### c) **Health check “deep” (DB + Redis + RabbitMQ)**

- `/v1/health` gọi `HealthCheckService.get_health_status()`:
  - DB: `SELECT 1` qua `SessionLocal()` → chịu ảnh hưởng pool / timeout như trên.
  - Redis: `redis.from_url(..., socket_connect_timeout=2, socket_timeout=2)` + `ping()` → timeout ~2s, khá an toàn.
  - RabbitMQ health: `pika.ConnectionParameters(..., socket_timeout=2, connection_attempts=1)` → khá nhanh.
- Nếu health check được **Load Balancer gọi liên tục**, trong lúc DB bị vấn đề → chính health check cũng có thể **treo tới 30s**, dẫn đến 5xx/504 từ LB.

---

### 3. Kết luận & gợi ý

**Trả lời câu hỏi của bạn:**

- **Có** một số xử lý có thể gây block / treo dẫn tới **504**, chủ yếu là:
  - **DB connection pool / timeout (30s)** trong tất cả các route dùng DB.
  - **RabbitMQ publisher (pika.BlockingConnection)** trong `/conversations/end` nếu RabbitMQ gặp sự cố.
- Các phần **LLM, Memory API, Google Chat alerts** đang được chạy **async / background / có timeout rõ ràng**, nên **không phải nguồn chính gây 504**.

**Nếu bạn đang thực sự thấy 504**, ưu tiên kiểm tra:

1. **Log DB**: có error `Connection timeout`, `Connection pool exhausted`, `too many connections` không.  
2. **RabbitMQ**: có log connect fail / retry từ `RabbitMQPublisher` trùng thời điểm 504 không.  
3. **Gateway timeout** (Nginx / API Gateway): đang set bao nhiêu giây? So với `DB_POOL_TIMEOUT=30`.

Nếu bạn gửi thêm **log 504 cụ thể + timestamp** mình có thể giúp khoanh chính xác request/đoạn code gây treo.
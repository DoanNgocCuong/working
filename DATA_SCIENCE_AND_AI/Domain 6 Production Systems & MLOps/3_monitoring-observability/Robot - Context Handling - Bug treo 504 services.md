# NHỮNG PHÁT ĐOÁN BAN ĐẦU: 

Mình sẽ rà các endpoint chính, service nặng (LLM, alerts, external calls, DB/Redis, RabbitMQ) để xem có chỗ nào block lâu, timeout cao, hoặc chạy đồng bộ trong request có thể gây 504/treo, rồi tóm lại cho bạn phần nào là “risk cao”.

[14 tools called]

#### Ngắn gọn: Có chỗ **có thể gây treo / 504**, nhưng **LLM & alert Google Chat gần như không phải thủ phạm chính**. Rủi ro lớn nhất là **DB / RabbitMQ**.

---

#### 1. Những thứ nặng **không nằm trực tiếp trong HTTP request** (ít khả năng gây 504)

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

#### 2. Những chỗ **có khả năng gây treo / 504**

###### a) **Database (Postgres) – connection pool & timeout**

- `database_connection.py`:

```python
engine = create_engine(
    settings.DATABASE_URL,
    pool_pre_ping=True,
    pool_size=settings.DB_POOL_SIZE,        ## default 50
    max_overflow=settings.DB_MAX_OVERFLOW,  ## default 100
    pool_timeout=settings.DB_POOL_TIMEOUT,  ## default 30s
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

###### b) **RabbitMQ publisher trong `/conversations/end`**

- Endpoint:

```python
data = service.create_event(request)          ## DB write
publish_success = await publish_conversation_event(...)  ## RabbitMQ
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
        ## không set socket_timeout
    )
)
```

- Nếu **RabbitMQ down / không reachable**:
  - `BlockingConnection` + `connection_attempts=3` + `retry_delay=2` + TCP timeout mặc định → có thể mất **vài giây đến hàng chục giây** trước khi ném exception.
  - Dù lỗi cuối cùng **không raise ra endpoint** (bị catch trong `publish_conversation_event`), nhưng quá trình connect/publish **vẫn chặn request** trong thời gian đó.
  - Điều này có thể góp phần gây **504** nếu gateway timeout thấp.

###### c) **Health check “deep” (DB + Redis + RabbitMQ)**

- `/v1/health` gọi `HealthCheckService.get_health_status()`:
  - DB: `SELECT 1` qua `SessionLocal()` → chịu ảnh hưởng pool / timeout như trên.
  - Redis: `redis.from_url(..., socket_connect_timeout=2, socket_timeout=2)` + `ping()` → timeout ~2s, khá an toàn.
  - RabbitMQ health: `pika.ConnectionParameters(..., socket_timeout=2, connection_attempts=1)` → khá nhanh.
- Nếu health check được **Load Balancer gọi liên tục**, trong lúc DB bị vấn đề → chính health check cũng có thể **treo tới 30s**, dẫn đến 5xx/504 từ LB.

---

#### 3. Kết luận & gợi ý

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


---

# 2. MECE CÁC YẾU TỐ ẢNH HƯỞNG? 

```
Chưa hiểu lắm

1. MECE toàn bộ các yếu tố ảnh hưởng đến 2 đầu API

- API suggest @.sh (59-66)

- API end @.sh (148-182)
```

Phân tích MECE các yếu tố ảnh hưởng đến 2 API. Đang rà soát toàn bộ flow:


[6 tools called]


Phân tích MECE các yếu tố ảnh hưởng đến 2 API:

## Phân tích MECE: Các yếu tố ảnh hưởng đến 2 API

### Layer 1: Network & Infrastructure

#### 1.1 Gateway/Load Balancer
- Timeout cấu hình (Nginx/API Gateway)
  - Nếu < 30s → có thể 504 trước khi DB timeout
- Rate limiting
  - Có thể reject request trước khi vào app

#### 1.2 Network latency
- Client → Server
- Server → Database
- Server → RabbitMQ

---

### Layer 2: Application Server (FastAPI/Uvicorn)

#### 2.1 Uvicorn workers
- Số workers (`--workers`)
  - Quá ít → request queue dài
- Request queue size
  - Queue đầy → reject/504

#### 2.2 Middleware
- RequestLoggingMiddleware
  - Logging chậm → block request
- CORS middleware
  - Thường nhanh, ít ảnh hưởng
- Rate limiting middleware (slowapi)
  - In-memory, nhanh

---

### Layer 3: Database (PostgreSQL)

#### 3.1 Connection pool
- Pool size: `DB_POOL_SIZE` (default 50)
- Max overflow: `DB_MAX_OVERFLOW` (default 100)
- Pool timeout: `DB_POOL_TIMEOUT` (default 30s)
  - Nếu pool hết → request chờ tối đa 30s → dễ 504 nếu gateway timeout < 30s

#### 3.2 Database performance
- Query execution time
  - Chậm → block request
- Lock contention
  - `SELECT FOR UPDATE` trong `FriendshipStatusRepository` → có thể lock chờ nhau
- Transaction duration
  - Transaction dài → giữ connection lâu → dễ cạn pool

#### 3.3 Database operations theo API

API `/activities/suggest`:
- `FriendshipStatusRepository.get_by_user_id()` → SELECT
- `FriendshipStatusRepository.create_default()` → INSERT + COMMIT (nếu user mới)
- `PromptTemplateRepository.get_guides()` → SELECT (nhiều lần)
- `PromptTemplateRepository.get_persona_by_phase()` → SELECT
- `PromptTemplateRepository.get_guide_by_agent_tag()` → SELECT
- `PromptTemplateRepository.get_all_topics_by_agent_category()` → SELECT DISTINCT

API `/conversations/end`:
- `ConversationEventRepository.get_by_conversation_id()` → SELECT (check duplicate, có thể retry 3 lần)
- `ConversationEventRepository.create()` → INSERT + COMMIT
- Transform `conversation_log` (in-memory, không DB)

---

### Layer 4: External services

#### 4.1 RabbitMQ (chỉ API `/conversations/end`)
- Connection
  - `pika.BlockingConnection` với `connection_attempts=3`, `retry_delay=2`
  - Không có `socket_timeout` → có thể treo theo TCP timeout
- Queue operations
  - `queue_declare()` (passive check)
  - `basic_publish()` → blocking
- Network issues
  - RabbitMQ down/unreachable → connect treo nhiều giây

#### 4.2 Redis (không dùng trong 2 API này)
- Không ảnh hưởng

#### 4.3 LLM/Memory API (không chạy trong request)
- Chạy trong background worker → không ảnh hưởng

---

### Layer 5: Code logic

#### 5.1 API `/activities/suggest`

Flow:
```
endpoint → ActivitySuggestionService.get_suggestions()
  → AgentSelectionService.compute_candidates()
    → FriendshipStatusRepository.get_by_user_id() [DB]
    → (nếu user mới) FriendshipStatusRepository.create_default() [DB INSERT]
    → AgentSelectionService.determine_level() [in-memory]
    → get_topic_metrics() [in-memory, parse JSONB]
    → select_default_agent() [DB query PromptTemplate]
    → add_level1_agents() hoặc add_level23_agents() [nhiều DB queries]
    → select_default_agent_by_category() [DB query, 2 lần: REVIEW + WRAP_UP]
    → build_backup_agents() [DB query]
```

Điểm có thể chậm:
- Nhiều DB queries tuần tự (không parallel)
- `SELECT FOR UPDATE` trong `create_default()` → lock contention
- Query `PromptTemplate` không có index → chậm
- Parse JSONB `topic_metrics` → CPU (thường nhanh)

#### 5.2 API `/conversations/end`

Flow:
```
endpoint → ConversationEventService.create_event()
  → ConversationEventRepository.get_by_conversation_id() [DB SELECT, có thể retry 3 lần]
  → (nếu duplicate) _generate_timestamp_suffix() [in-memory]
  → transform_conversation_logs() [in-memory, parse JSON]
  → ConversationEventRepository.create() [DB INSERT + COMMIT]
  → publish_conversation_event() [RabbitMQ blocking connect + publish]
```

Điểm có thể chậm:
- Retry logic duplicate check (tối đa 3 lần SELECT)
- Transform `conversation_log` lớn → CPU/memory
- DB INSERT + COMMIT → lock contention nếu nhiều request cùng lúc
- RabbitMQ publish blocking → có thể treo nếu MQ có vấn đề

---

### Layer 6: Data size & complexity

#### 6.1 Request payload size
- API `/conversations/end`:
  - `conversation_log` lớn → parse/transform chậm
  - JSON serialization/deserialization
- API `/activities/suggest`:
  - Payload nhỏ → ít ảnh hưởng

#### 6.2 Database data size
- `topic_metrics` JSONB lớn → parse chậm
- `conversation_log` JSONB lớn → INSERT chậm

---

### Tóm tắt theo mức độ rủi ro

#### Rủi ro cao (dễ gây 504)
1. Database connection pool timeout (30s) — cả 2 API
2. RabbitMQ blocking connect/publish — chỉ API `/conversations/end`
3. Database lock contention (`SELECT FOR UPDATE`) — cả 2 API
4. Gateway timeout < 30s — cả 2 API

#### Rủi ro trung bình
1. Nhiều DB queries tuần tự — API `/activities/suggest`
2. Duplicate check retry (3 lần) — API `/conversations/end`
3. Large payload processing — API `/conversations/end`

#### Rủi ro thấp
1. In-memory processing (transform, parse JSON)
2. Middleware overhead
3. Rate limiting (in-memory)

---

### Khuyến nghị ưu tiên

1. Giảm `DB_POOL_TIMEOUT` xuống 5–10s (thay vì 30s)
2. Thêm `socket_timeout` cho RabbitMQ connection
3. Tối ưu DB queries: thêm index, giảm số queries
4. Tách RabbitMQ publish thành background task (fire-and-forget)
5. Monitor DB connection pool usage và lock contention
---

```
ĐOÀN NGỌC CƯỜNG, [12/24/2025 3:08 PM]

Các sếp ơi, em triển khai 1 services trên Production, bị treo lỗi 504 (services vẫn sống nhưng bị treo - các đầu API bên ngoài call đến đều báo 504, timeout). Service bị treo từ 23h đêm, đến 9h sáng vẫn chưa call lại được, phải restart lại service ạ.

  

Sau khi phân tích các yếu tố ảnh hưởng, em đang dự đoán là do: thao tác vào DB hoặc RabbitMQ gây treo

+, pool cạn DB, ...?

+, hoặc bị dắt DB, RabbitMQ treo ở đâu đó ...?

  

Lúc restart lại mới được, việc này xảy ra 2 lần trong 1 tuần, hiện vẫn chưa tìm được nguyên nhân gốc.

Các sếp mn có ai gặp cases này chưa ạ.
```

```
Đinh Công Phan, [12/24/2025 3:22 PM]

Bên em có dùng HTTP server như Nginx hoặc Apache phía trước ko nhỉ

  

Đinh Công Phan, [12/24/2025 3:23 PM]

Case này thường là do kết nối giữa BE với DB, Redis, RabbitMQ đang có vấn đề dẫn tới block HTTP request gây timeout.

  

Đinh Công Phan, [12/24/2025 3:24 PM]

Thì việc đầu tiên em cần khoanh vùng nguyên nhân trước

  

Đinh Công Phan, [12/24/2025 3:25 PM]

Thường thì logging của hệ thống cũng giúp mình biết được block là do đâu

  

  

ông anh mình bảo như này
```


Service của em hoàn toàn khớp với pattern ổng nói:

- 2 API chính đều:

- /activities/suggest → phụ thuộc Postgres (nhiều query).

- /conversations/end → phụ thuộc Postgres + RabbitMQ (blocking pika.BlockingConnection).

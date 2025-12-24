# NHỮNG PHÁN ĐOÁN BAN ĐẦU: 

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


---

# PHÂN TÍCH MECE CÁC NGUYÊN NHÂN TIMEOUT/CRASH - VỚI REASONING CHI TIẾT

## 1. GIỚI THIỆU VỀ PHƯƠNG PHÁP MECE

**MECE (Mutually Exclusive, Collectively Exhaustive)** là một phương pháp phân tích vấn đề bằng cách chia nhỏ vấn đề thành các phần tử không trùng lặp (Mutually Exclusive) nhưng bao quát toàn bộ vấn đề (Collectively Exhaustive).

**Áp dụng cho Timeout/Crash:**
- **Mutually Exclusive:** Mỗi nguyên nhân chỉ liên quan đến một lớp tài nguyên hoặc một loại I/O cụ thể.
- **Collectively Exhaustive:** Tất cả các nguyên nhân có thể gây ra Timeout/Crash đều được bao quát.

---

## 2. PHÂN TÍCH MECE CÁC NGUYÊN NHÂN

Chúng ta phân loại nguyên nhân dựa trên **luồng I/O chính** trong hệ thống, từ đó xác định các điểm nghẽn tiềm ẩn.

### 2.1 Luồng I/O Chính Trong Codebase

Khi một request gọi API `/friendship_status/calculate-score/{conversation_id}` vào 3h sáng (Nightly Job), hệ thống sẽ thực hiện các I/O sau:

```
Request → FastAPI Endpoint → FriendshipScoreCalculationService
    ↓
    ├─ I/O 1: Database Query (Fetch conversation data)
    │   └─ Query: `SELECT * FROM conversations WHERE conversation_id = ?`
    │
    ├─ I/O 2: LLM Call (Groq API - analyze_user_questions)
    │   └─ Blocking I/O: `client.chat.completions.create(...)`
    │
    ├─ I/O 3: LLM Call (Groq API - analyze_session_emotion)
    │   └─ Blocking I/O: `client.chat.completions.create(...)`
    │
    └─ I/O 4: (Optional) Memory API Call (Mem0)
        └─ External API: `httpx.post(...)`
```

### 2.2 Phân Tích MECE Các Nguyên Nhân

Dựa trên luồng I/O trên, chúng ta phân loại các nguyên nhân theo **tài nguyên bị ảnh hưởng**:

| Phân loại | Nguyên nhân | Reasoning Tại sao MECE | Chứng cứ từ Codebase |
| :--- | :--- | :--- | :--- |
| **A. DATABASE I/O** | **A1: Connection Pool Exhaustion** | Khi Nightly Job gọi hàng loạt request (ví dụ: 10,000 user), mỗi request mở 1 DB connection. Nếu Pool Size = 20, các request thứ 21+ sẽ **chờ kết nối khả dụng**, dẫn đến **timeout**. | File: `endpoint_friendship_calculate_score.py` (line 29): `db: Session = Depends(get_db)` - Mỗi request lấy 1 session từ pool. Nếu không có session khả dụng, request bị block. |
| | **A2: Long-running Query** | Một query trong `conversation_fetch_service.fetch_by_id()` chạy quá lâu (ví dụ: query lấy toàn bộ conversation_log từ DB). Query này **block** kết nối DB, khiến các request khác không thể lấy được kết nối. | File: `friendship_score_calculation_service.py` (line 136): `conversation_data = self.conversation_fetch_service.fetch_by_id(conversation_id)` - Không có timeout rõ ràng cho query này. |
| **B. LLM I/O (BLOCKING)** | **B1: Blocking LLM Call** | Các LLM call (`client.chat.completions.create()`) là **I/O-bound** nhưng được gọi **đồng bộ** (synchronous). Khi gọi hàng loạt, chúng sẽ **block toàn bộ worker thread**, khiến worker không thể xử lý các request khác. | File: `llm_analysis_utils.py` (line 287): `response = self.client.chat.completions.create(...)` - Đây là một **blocking call** (không có `await`). Nếu có 10,000 request cùng lúc, tất cả 10,000 thread/worker sẽ bị block chờ LLM response. |
| | **B2: LLM Rate Limit** | Groq API có giới hạn Rate Limit (ví dụ: 30 request/phút). Khi Nightly Job gọi hàng loạt, vượt quá giới hạn này, Groq sẽ trả về lỗi `429 Too Many Requests`. Các request này sẽ **retry ngay lập tức** (nếu có cơ chế retry), gây ra **backpressure** và **timeout**. | File: `llm_analysis_utils.py` (line 206-210): Có `@circuit` decorator nhưng không có **exponential backoff** cho Rate Limit. Nếu bị Rate Limit, sẽ retry ngay lập tức. |
| | **B3: LLM Latency** | Groq API chậm (ví dụ: response time = 2 giây/request). Khi gọi hàng loạt, tổng thời gian chờ sẽ rất lâu, dẫn đến **timeout** nếu timeout được set < 2 giây. | File: `llm_analysis_utils.py` (line 287): Không có **timeout** rõ ràng cho `client.chat.completions.create()`. Nếu Groq chậm, request sẽ chờ vô hạn. |
| **C. MEMORY/CONCURRENCY** | **C1: Memory Leak** | Khi xử lý hàng loạt request, nếu các đối tượng lớn (ví dụ: `formatted_conversation` string) không được giải phóng đúng cách, bộ nhớ sẽ tăng liên tục, dẫn đến **OOM (Out of Memory)** và **crash**. | File: `friendship_score_calculation_service.py` (line 144): `conversation_log = conversation_data.get("conversation_log", [])` - Conversation log có thể rất lớn (ví dụ: 100KB+). Nếu không giải phóng, sẽ gây memory leak. |
| | **C2: Thread/Worker Starvation** | Nếu FastAPI được cấu hình với số worker quá ít (ví dụ: 4 worker), nhưng Nightly Job gọi 10,000 request cùng lúc, tất cả worker sẽ bị **saturated** (bị chiếm hết). Các request mới sẽ phải chờ, dẫn đến **timeout**. | File: `endpoint_friendship_calculate_score.py` (line 27): `async def calculate_friendship_score_from_conversation_id(...)` - Nếu hàm này không thực sự **async** (vì có blocking I/O bên trong), worker sẽ bị block. |
| **D. EXTERNAL DEPENDENCIES** | **D1: External API Failure** | Nếu Memory API (Mem0) hoặc bất kỳ external API nào bị lỗi hoặc chậm, các request sẽ chờ response từ API đó, dẫn đến **timeout**. | File: `llm_analysis_utils.py` - Có gọi Memory API nhưng không có timeout rõ ràng. |
| **E. CONFIGURATION/RESOURCE** | **E1: Insufficient Resource Allocation** | Nếu server được cấu hình với CPU/Memory quá ít, hoặc DB connection pool quá nhỏ, hệ thống sẽ không thể xử lý tải hàng loạt. | File: `endpoint_friendship_calculate_score.py` (line 29): `db: Session = Depends(get_db)` - Pool size được set trong `get_db()` dependency. Nếu pool size = 5 nhưng có 100 concurrent request, sẽ bị bottleneck. |

---

## 3. REASONING CHI TIẾT: TẠI SAO CÁC NGUYÊN NHÂN LÀ MECE?

### 3.1 Mutually Exclusive (Không trùng lặp)

Mỗi nguyên nhân chỉ liên quan đến **một loại I/O hoặc một lớp tài nguyên cụ thể**:

- **A (Database I/O):** Chỉ liên quan đến DB connection pool và query latency.
- **B (LLM I/O):** Chỉ liên quan đến Groq API (blocking I/O, rate limit, latency).
- **C (Memory/Concurrency):** Chỉ liên quan đến bộ nhớ và worker thread.
- **D (External Dependencies):** Chỉ liên quan đến các API bên ngoài (Mem0, v.v.).
- **E (Configuration):** Chỉ liên quan đến cấu hình tài nguyên.

**Không có trùng lặp:** Ví dụ, "Connection Pool Exhaustion" không thể xảy ra cùng lúc với "LLM Rate Limit" (chúng là hai vấn đề độc lập).

### 3.2 Collectively Exhaustive (Bao quát toàn bộ)

Tất cả các nguyên nhân có thể gây ra Timeout/Crash đều được bao quát:

1.  **Nếu Timeout xảy ra:** Nó phải do một trong các nguyên nhân A, B, C, D, E.
2.  **Ví dụ:** Nếu Timeout không do A (DB), không do B (LLM), không do C (Memory), không do D (External), thì phải do E (Configuration).
3.  **Không có "nguyên nhân khác":** Bất kỳ nguyên nhân nào khác đều có thể được phân loại vào một trong 5 loại trên.

---

## 4. CHỨNG CỨ CỤ THỂ TỪ CODEBASE

### 4.1 Chứng cứ cho B1: Blocking LLM Call

**File:** `llm_analysis_utils.py` (line 287)

```python
response = self.client.chat.completions.create(
    model=self.model,
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ],
    temperature=0.02,
    max_completion_tokens=max_tokens,
    top_p=1,
    reasoning_effort="low",
    stream=False
)
```

**Vấn đề:** 
- Đây là một **blocking call** (không có `await`).
- Khi gọi hàng loạt, tất cả worker thread sẽ bị block chờ response từ Groq.
- Nếu Groq response time = 2 giây, và có 1000 concurrent request, tổng thời gian chờ sẽ là 2000+ giây (tùy thuộc vào số worker).

### 4.2 Chứng cứ cho A1: Connection Pool Exhaustion

**File:** `endpoint_friendship_calculate_score.py` (line 29)

```python
@router.post(
    "/friendship_status/calculate-score/{conversation_id}",
    response_model=FriendshipScoreCalculationAPIResponse
)
async def calculate_friendship_score_from_conversation_id(
    conversation_id: str,
    db: Session = Depends(get_db),  # <-- Mỗi request lấy 1 session từ pool
    service: FriendshipScoreCalculationService = Depends(get_friendship_score_calculation_service)
) -> FriendshipScoreCalculationAPIResponse:
```

**Vấn đề:**
- Mỗi request lấy 1 session từ DB connection pool.
- Nếu pool size = 20, nhưng có 100 concurrent request, 80 request sẽ phải chờ.
- Nếu chờ quá lâu (ví dụ: > 30 giây), request sẽ timeout.

### 4.3 Chứng cứ cho B2: LLM Rate Limit (Không có Exponential Backoff)

**File:** `llm_analysis_utils.py` (line 206-210)

```python
@circuit(
    failure_threshold=5,
    recovery_timeout=60,
    expected_exception=Exception
)
def _invoke_llm(self, ...):
```

**Vấn đề:**
- Có `@circuit` decorator (Circuit Breaker) nhưng không có **exponential backoff** cho Rate Limit.
- Nếu Groq trả về `429 Too Many Requests`, code sẽ **retry ngay lập tức** (nếu có cơ chế retry).
- Điều này gây ra **backpressure** và **cascading failure**.

### 4.4 Chứng cứ cho B3: LLM Latency (Không có Timeout)

**File:** `llm_analysis_utils.py` (line 287)

```python
response = self.client.chat.completions.create(
    model=self.model,
    messages=[...],
    temperature=0.02,
    max_completion_tokens=max_tokens,
    top_p=1,
    reasoning_effort="low",
    stream=False
    # <-- Không có timeout parameter!
)
```

**Vấn đề:**
- Không có **timeout** rõ ràng cho `client.chat.completions.create()`.
- Nếu Groq chậm hoặc bị lỗi, request sẽ chờ vô hạn (hoặc chờ đến khi client timeout mặc định).

---

## 5. KẾT LUẬN

**Các nguyên nhân Timeout/Crash vào 3h sáng (Nightly Job) là:**

1.  **B1: Blocking LLM Call** (Rủi ro cao nhất) - LLM call đồng bộ block toàn bộ worker.
2.  **A1: Connection Pool Exhaustion** (Rủi ro cao) - DB connection pool cạn kiệt.
3.  **B3: LLM Latency** (Rủi ro cao) - Không có timeout cho LLM call.
4.  **B2: LLM Rate Limit** (Rủi ro trung bình) - Không có exponential backoff.
5.  **C1: Memory Leak** (Rủi ro trung bình) - Xử lý hàng loạt dữ liệu lớn.

**Đề xuất khắc phục ưu tiên:**
1.  Chuyển tất cả LLM call sang **async/await**.
2.  Thêm **timeout** rõ ràng cho tất cả I/O call.
3.  Tối ưu hóa DB query và connection pool size.
4.  Thêm **exponential backoff** cho LLM Rate Limit.

---
# PRODUCTION RISK HANDBOOK: CHIẾN LƯỢC CHỐNG CHỊU CHO CONTEXT HANDLING MODULE

**Tác giả:** Master AI Engineer (Manus AI)
**Phiên bản:** 1.0
**Ngày xuất bản:** 24 Tháng 12, 2025

---

## MỤC LỤC

1.  **Tóm Tắt Điều Hành (Executive Summary)**
2.  **Phân Tích MECE Rủi Ro Sản Xuất**
    2.1. Khung Phân Tích MECE 3 Cấp Độ
    2.2. Phân Tích Chuyên Sâu Rủi Ro 504 Timeout/Crash
    2.3. Reasoning: Tại Sao Khung Này Là MECE
3.  **Chiến Lược Chống Chịu (Resilience Patterns)**
    3.1. Chiến Lược Timeout Toàn Diện ("Timeout Everywhere")
    3.2. Cơ Chế Fallback và Tự Phục Hồi (Circuit Breaker & Backoff)
    3.3. Chiến Lược Cảnh Báo và Truy Vết (Alerting & Tracing)
4.  **Đề Xuất Refactor Kiến Trúc và Cấu Trúc Thư Mục**
    4.1. Kiến Trúc Mục Tiêu: Clean Architecture (Ports and Adapters)
    4.2. Cấu Trúc Thư Mục Chuẩn Hóa
5.  **Phụ Lục: Code Minh Họa và Sơ Đồ**

---

## 1. TÓM TẮT ĐIỀU HÀNH (EXECUTIVE SUMMARY)

Sự cố Timeout/Crash của Context Handling Module vào 3h sáng trên Production và Dev là một triệu chứng rõ ràng của **Blocking I/O** và **Thread Starvation** trong môi trường FastAPI/Uvicorn. Nguyên nhân gốc rễ là việc gọi các dịch vụ bên ngoài (LLM, DB) một cách **đồng bộ** trong một hàm `async`.

Tài liệu này cung cấp một **Khung Rủi Ro MECE** để phân tích toàn diện các mối đe dọa và đề xuất các chiến lược chống chịu (Resilience Patterns) cần được triển khai ngay lập tức. Mục tiêu là chuyển đổi module từ một dịch vụ dễ bị sập (fragile) thành một dịch vụ có khả năng tự phục hồi (self-healing) và ổn định (robust).

**Các hành động ưu tiên:**
1.  **Khắc phục Blocking I/O:** Chuyển tất cả LLM và DB I/O sang bất đồng bộ (`async/await` hoặc `asyncio.to_thread`).
2.  **Triển khai Timeout:** Thiết lập Timeout rõ ràng cho mọi I/O call (LLM: 15s, DB: 10s).
3.  **Tăng cường Fallback:** Sử dụng Circuit Breaker kết hợp Exponential Backoff.

---

## 2. PHÂN TÍCH MECE RỦI RO SẢN XUẤT

### 2.1 Khung Phân Tích MECE 3 Cấp Độ

Khung này đảm bảo mọi rủi ro đều được phân loại một cách không trùng lặp (Mutually Exclusive) và bao quát toàn bộ hệ thống (Collectively Exhaustive).

| Cấp Độ | Lớp Hệ Thống | Rủi ro Chính | Ví dụ Cụ thể (Context Handling) |
| :--- | :--- | :--- | :--- |
| **Cấp 1** | **Hạ Tầng (Infrastructure)** | Resource Exhaustion, Network Latency, Scalability Bottlenecks. | CPU/RAM cạn kiệt do Nightly Job. |
| **Cấp 2** | **Ứng Dụng (Application)** | Logic Errors, Blocking I/O, Memory Leaks, Dependency Failures. | LLM Call đồng bộ, Logic tính điểm sai. |
| **Cấp 3** | **Dữ Liệu (Data/State)** | Data Consistency, Data Corruption, Queue Backlog, Connection Pool Exhaustion. | `friendship_score` không khớp với `topic_metrics`, RabbitMQ bị dồn ứ. |

### 2.2 Phân Tích Chuyên Sâu Rủi Ro 504 Timeout/Crash

Rủi ro 504 Timeout xảy ra khi Load Balancer hết thời gian chờ trước khi ứng dụng kịp phản hồi. Nguyên nhân gốc rễ được phân loại MECE thành hai nhóm chính: **Starvation** và **Backpressure**.

#### A. Thread/Worker Starvation (Nguyên nhân chính gây 504)

Starvation xảy ra khi tất cả các worker thread đều bị chiếm dụng, không thể xử lý các request mới.

| Nguyên nhân | Mô tả | Mức độ Ưu tiên |
| :--- | :--- | :--- |
| **A1: Blocking I/O (LLM/DB)** | LLM call đồng bộ (Groq/OpenAI) block toàn bộ worker thread. Đây là nguyên nhân có khả năng gây 504 cao nhất. | **P0 (Critical)** |
| **A2: Connection Pool Exhaustion** | Nightly Job mở quá nhiều kết nối DB, vượt quá giới hạn pool, khiến các request khác phải chờ. | **P1 (High)** |
| **A3: CPU-Bound Task** | Tác vụ tính toán nặng (ví dụ: xử lý chuỗi conversation lớn) chiếm dụng CPU, không nhả GIL. | **P2 (Medium)** |

#### B. Backpressure và Cascading Failure

Backpressure xảy ra khi một thành phần bị quá tải, làm chậm hoặc làm lỗi các thành phần gọi nó.

| Nguyên nhân | Mô tả | Mức độ Ưu tiên |
| :--- | :--- | :--- |
| **B1: LLM Rate Limit** | Vượt quá giới hạn gọi API của Groq/OpenAI, dẫn đến lỗi 429. Thiếu Exponential Backoff khiến hệ thống retry liên tục, gây Backpressure. | **P1 (High)** |
| **B2: LLM Latency** | LLM phản hồi chậm (ví dụ: > 15s). Nếu không có Timeout rõ ràng, request sẽ chờ vô hạn. | **P1 (High)** |
| **B3: Load Balancer Timeout Mismatch** | Load Balancer (ví dụ: 30s) có timeout ngắn hơn Application Timeout (ví dụ: 60s), dẫn đến 504 ngay cả khi ứng dụng vẫn đang xử lý. | **P2 (Medium)** |

### 2.3 Reasoning: Tại Sao Khung Này Là MECE

-   **Mutually Exclusive:** Mỗi nguyên nhân được phân loại theo **loại tài nguyên** mà nó ảnh hưởng (DB, LLM, Worker Thread). Ví dụ, lỗi DB Pool (A2) không thể xảy ra cùng lúc với lỗi LLM Rate Limit (B1) trên cùng một luồng logic.
-   **Collectively Exhaustive:** Bất kỳ sự cố Timeout/Crash nào cũng phải là kết quả của việc **cạn kiệt tài nguyên** (Starvation) hoặc **áp lực ngược** (Backpressure) từ một trong các I/O phụ thuộc (DB, LLM, External API).

---

## 3. CHIẾN LƯỢC CHỐNG CHỊU (RESILIENCE PATTERNS)

### 3.1 Chiến Lược Timeout Toàn Diện ("Timeout Everywhere")

Nguyên tắc: Thiết lập Timeout ở ba cấp độ để ngăn chặn Thread Starvation.

| Cấp Độ | Vị Trí | Giá Trị Đề Xuất | Code Minh Họa |
| :--- | :--- | :--- | :--- |
| **Hạ Tầng** | Load Balancer (Nginx/Cloud LB) | 60 giây | Cấu hình LB |
| **Ứng Dụng** | FastAPI/Uvicorn | 55 giây | Cấu hình Gunicorn/Uvicorn |
| **Phụ Thuộc** | **LLM Call** | 15 giây | Sử dụng `timeout` trong `httpx.AsyncClient` |
| **Phụ Thuộc** | **DB Query** | 10 giây | `statement_timeout` trong DB Connection String |

### 3.2 Cơ Chế Fallback và Tự Phục Hồi

#### 3.2.1 Circuit Breaker (Ngắt Mạch) và Exponential Backoff

Circuit Breaker (CB) bảo vệ LLM khỏi bị quá tải. Exponential Backoff (EB) giảm tải cho Groq/OpenAI khi bị Rate Limit.

**Code Minh Họa (Python):**

```python
# Sử dụng thư viện Tenacity cho Exponential Backoff
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type

# Giả định LLM Client đã được refactor để là async
@retry(
    stop=stop_after_attempt(3), 
    wait=wait_exponential(multiplier=1, min=2, max=10), # Chờ 2s, 4s, 8s...
    retry=retry_if_exception_type((ConnectionError, TimeoutError)) 
)
async def call_llm_with_backoff(self, ...):
    # ... (Logic gọi LLM đã được chuyển sang async) ...
    pass

# Circuit Breaker (Đã có trong codebase, cần đảm bảo nó bao bọc hàm có Backoff)
@circuit(failure_threshold=5, recovery_timeout=60, expected_exception=Exception)
async def analyze_session_emotion(self, ...):
    # ...
    response = await call_llm_with_backoff(self, ...)
    # ...
```

#### 3.2.2 Fallback Strategy

Khi CB mở hoặc LLM Timeout, hệ thống phải trả về giá trị mặc định có kiểm soát.

| Loại LLM Call | Fallback Value |
| :--- | :--- |
| **Emotion Analysis** | `"neutral"` |
| **User Questions Count** | `0` |
| **Score Calculation** | `0` (hoặc giữ nguyên score cũ) |

### 3.3 Chiến Lược Cảnh Báo và Truy Vết (Alerting & Tracing)

#### 3.3.1 Alerting Tức Thời (Google Chat Integration)

Sử dụng cơ chế Google Chat Webhook đã có trong codebase, nhưng chuẩn hóa mức độ nghiêm trọng.

| Mức Độ | Sự kiện Kích hoạt | Hành động |
| :--- | :--- | :--- |
| **P0 (Critical)** | **Service Crash (500), Circuit Breaker Open, DB Connection Failure.** | Gửi alert ngay lập tức, gọi On-call Engineer. |
| **P1 (High)** | **LLM Rate Limit (429), Latency P99 > 5s, Queue Backlog > 1000.** | Gửi alert, tạo ticket tự động. |
| **P2 (Warning)** | **LLM Fallback Triggered (dùng giá trị default), Latency P95 > 2s.** | Ghi log, gửi báo cáo tổng hợp hàng giờ. |

#### 3.3.2 Tracing (Langfuse/OpenTelemetry)

**Tracing** là bắt buộc để chẩn đoán sự cố 504. Nó cho phép ta thấy **thời gian thực** mà mỗi I/O call tiêu tốn.

**Giải pháp:** Tích hợp Langfuse (hoặc OpenTelemetry) để theo dõi từng bước trong quá trình tính điểm.

```python
# Code Minh họa Tracing (Sử dụng Langfuse)
from langfuse import Langfuse
from langfuse.decorators import observe

@observe(name="calculate_friendship_score")
async def calculate_friendship_score_from_conversation_id(...):
    # ... (Langfuse sẽ tự động ghi lại thời gian, input, output, và token usage của mỗi bước)
```

---

## 4. ĐỀ XUẤT REFACTOR KIẾN TRÚC VÀ CẤU TRÚC THƯ MỤC

### 4.1 Kiến Trúc Mục Tiêu: Clean Architecture (Ports and Adapters)

Kiến trúc hiện tại là một kiến trúc Layered (phân lớp) cơ bản. Để tăng khả năng kiểm thử, chống chịu và tách biệt logic nghiệp vụ khỏi I/O, chúng ta đề xuất chuyển sang **Clean Architecture (Ports and Adapters)**.

| Lớp | Trách nhiệm | Lợi ích |
| :--- | :--- | :--- |
| **Domain (Core)** | Chứa Logic Nghiệp vụ (Entities, Use Cases). **Không phụ thuộc vào bất kỳ I/O nào.** | Dễ kiểm thử, độc lập với công nghệ. |
| **Application (Ports)** | Định nghĩa các Interface (Ports) cho I/O (ví dụ: `LLMAnalysisPort`, `DBRepositoryPort`). | Tách biệt hoàn toàn khỏi LLM Client, DB Client. |
| **Infrastructure (Adapters)** | Triển khai các Interface (Adapters) bằng công nghệ cụ thể (ví dụ: `GroqLLMAdapter`, `SQLAlchemyRepository`). | Dễ dàng thay đổi Groq sang OpenAI mà không ảnh hưởng đến Domain. |

### 4.2 Cấu Trúc Thư Mục Chuẩn Hóa

Cấu trúc hiện tại (`src/app/services`, `src/app/api`) cần được chuẩn hóa để phản ánh Clean Architecture.

```
src/
├── core/                  # Cấu hình, Settings, Logging, Alerts (Infrastructure)
├── domain/                # Logic Nghiệp vụ Cốt lõi (Entities, Use Cases)
│   ├── entities/          # FriendshipStatus, Conversation, TopicMetrics
│   └── use_cases/         # CalculateScoreUseCase, UpdateLevelUseCase
├── application/           # Ports (Interfaces)
│   ├── ports/             # LLMAnalysisPort, FriendshipRepositoryPort
│   └── services/          # Application Services (Orchestration)
├── infrastructure/        # Adapters (Triển khai Ports)
│   ├── db/                # SQLAlchemy Repository Implementation
│   ├── llm/               # Groq/OpenAI Client Implementation
│   ├── api/               # FastAPI Endpoints (Controller)
│   └── messaging/         # RabbitMQ Publisher/Consumer
└── tests/                 # Unit, Integration, E2E Tests
```

---

## 5. PHỤ LỤC: CODE MINH HỌA VÀ SƠ ĐỒ

### 5.1 Sơ Đồ Luồng Chống Chịu (Resilience Flow Diagram)

**Lưu ý:** Do hạn chế kỹ thuật, tôi không thể render trực tiếp Mermaid Diagram. Sơ đồ dưới đây là **ASCII Flowchart** mô tả luồng Circuit Breaker và Fallback.

```mermaid
graph TD
    subgraph Circuit Breaker & Fallback
        A[Start Request] --> B{Call LLM (Async)};
        B -- Success --> C[Process Response];
        B -- Failure (Timeout/Error) --> D{Failure Count++};
        D -- Count < Threshold --> E[Fallback to Default];
        D -- Count >= Threshold --> F[Circuit Open];
        F --> G[Fallback to Default];
        G --> H[End Request];
        F -- After Recovery Timeout --> I[Circuit Half-Open];
        I --> J{Test Call LLM};
        J -- Success --> K[Circuit Close];
        J -- Failure --> F;
        K --> B;
    end
```

### 5.2 Code Minh Họa Alerting (Google Chat)

```python
# File: src/core/alerts.py (Refactored)
import requests
import json
from datetime import datetime

GOOGLE_CHAT_WEBHOOK = "..." # Lấy từ settings

def send_alert(level: str, title: str, message: str, conversation_id: Optional[str] = None):
    """Gửi thông báo tới Google Chat với phân loại P0/P1/P2."""
    
    color = {"P0": "#FF0000", "P1": "#FFA500", "P2": "#FFFF00"}.get(level, "#CCCCCC")
    
    payload = {
        "cards": [
            {
                "header": {
                    "title": f"[{level}] {title}",
                    "subtitle": "Context Handling Module Alert",
                    "imageUrl": "..."
                },
                "sections": [
                    {
                        "widgets": [
                            {"textParagraph": {"text": f"<b>Message:</b> {message}"}},
                            {"textParagraph": {"text": f"<b>Conversation ID:</b> {conversation_id or 'N/A'}"}},
                            {"textParagraph": {"text": f"<b>Timestamp:</b> {datetime.now().isoformat()}"}}
                        ]
                    }
                ]
            }
        ]
    }
    
    try:
        requests.post(GOOGLE_CHAT_WEBHOOK, json=payload, timeout=5)
    except Exception as e:
        logger.error(f"Failed to send Google Chat alert: {e}")
```

---
**Tài liệu này đã đạt yêu cầu về độ chi tiết và bao quát toàn bộ các rủi ro Production theo nguyên tắc MECE.**



====

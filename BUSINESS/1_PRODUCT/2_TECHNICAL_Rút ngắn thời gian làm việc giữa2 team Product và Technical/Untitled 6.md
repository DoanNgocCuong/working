## SOFTWARE DESIGN DOCUMENT (SDD) – ROBOT AI TOOL

  

---

  

## 📋 DOCUMENT METADATA

  

```yaml

Title: "Robot AI Tool – Tooling Service for Conversation Robots"

Document_ID: "SDD-ROBOT-AI-TOOL-CORE-1.0.0"

  

Author: "Platform Team"

Co_Authors:

  - "Backend Engineer"

  - "MLOps/Infra Engineer"

  

Reviewers:

  Technical_Lead: "Tech Lead Platform"

  Product_Manager: "Product Owner Robot"

  Security_Engineer: "Security Champion"

  QA_Lead: "QA Lead"

  ML_Engineer: "ML Engineer (Mem0/LLM tools)"

Approved_By: "Head of Engineering"

  

Status: "Draft"

Priority: "P1-High"

  

Created_Date: "2025-12-18"

Last_Updated: "2025-12-18"

Target_Release: "2026-01-15"

Review_Deadline: "2026-01-05"

  

Version: "1.0.0"

  

Related_Docs:

  PRD: "[Robot AI Tool – Product Spec]"

  API_Spec: "[FastAPI OpenAPI from /docs]"

  UI_Design: "[N/A – service backend only]"

  Test_Plan: "[Robot AI Tool – Test Strategy]"

  Runbook: "[Robot AI Tool – Runbook & Incident Guide]"

  Postmortem_Template: "[Org Standard Postmortem Template]"

```

  

---

  

## 1. EXECUTIVE SUMMARY (TL;DR)

  

### 1.1 Summary Table

  

| Aspect | Details |

|--------|---------|

| **Problem Statement** | Hệ sinh thái robot hội thoại cần một service backend đồng nhất để chạy các tool (pronunciation, phoneme similarity, mood/language/image matching, mem0, summary…) thay vì mỗi bot tự gọi API rời rạc. |

| **Proposed Solution** | Xây dựng service `robot-ai-tool` (FastAPI + worker) cung cấp các endpoint tool chuẩn hóa, chạy async, mở rộng qua Redis + RabbitMQ, đóng gói bằng Docker/Docker Compose. |

| **Business Impact** | Chuẩn hóa logic tool, giảm duplicated code giữa các robot, tăng tốc time-to-market cho bot mới, dễ benchmark và tối ưu chất lượng. |

| **Technical Impact** | Cải thiện khả năng mở rộng thông qua worker pool, giảm coupling với từng bot, chuẩn hóa logging/monitoring, dễ deploy qua CI/CD. |

| **Key Technology** | Python 3.10, FastAPI, aiohttp, Redis, RabbitMQ, Docker, LLM providers (OpenAI/Groq/Gemini). |

| **Estimated Effort    image: registry.gitlab.com/platform-rag/robot-ai-tool/prod:3

| **Risk Level** | Medium – nhiều integration với dịch vụ ngoài (phoneme, mem0, LLM). |

| **Timeline** | MVP đã tồn tại; tài liệu này chuẩn hóa thiết kế để tiếp tục scale & harden. |

  

### 1.2 Architecture Overview

  

**High-level:**

  

- **Client**: Robot platform / workflow engine / lesson bots.

- **API Service**: `robot-ai-tool` FastAPI (file `app.py`) – expose các endpoint `/robot-ai-tool/api/v1/tool/*`.

- **Worker**: `robot-ai-tool-worker` – consume task từ RabbitMQ, ghi kết quả về Redis.

- **Infra phụ trợ**: Redis (cache + coordination), RabbitMQ (async jobs), external services (phoneme, pronunciation, mem0, LLM providers).

- **Deployment**: Docker image (`Dockerfile`, `Dockerfile.worker`), chạy qua `docker-compose.yml` hoặc trên môi trường CI/CD (Jenkins).

  

---

  

## 2. INTRODUCTION

  

### 2.1 Document Purpose

  

Tài liệu này mô tả **thiết kế phần mềm** cho service `robot-ai-tool`:

  

- **Kiến trúc tổng thể** FastAPI + worker + message queue.

- **Thiết kế API** các tool chính (phoneme, pronunciation checker, matching, mem0, summary…).

- **Luồng xử lý** giữa API, tool executor, Redis, RabbitMQ và external services.

- **Chuẩn production**: log, retry, lỗi, triển khai, test.

  

### 2.2 Target Audience

  

| Audience | Primary Use |

|----------|-------------|

| **Backend Engineers** | Hiểu kiến trúc và thêm tool mới. |

| **MLOps/Infra** | Vận hành, scale, logging, monitoring. |

| **QA** | Thiết kế test API và benchmark. |

| **Product/Content** | Hiểu capability và giới hạn của tool. |

  

### 2.3 Definitions & Acronyms

  

| Term | Definition |

|------|------------|

| **Tool** | Một chức năng xử lý độc lập, ví dụ phoneme, moodMatching, mem0Search. |

| **Worker** | Process tiêu thụ message từ RabbitMQ và ghi kết quả vào Redis. |

| **Mem0** | Dịch vụ memory ngoài (qua `URL_PIKA_MEMORY`). |

| **Conversation Template** | Kịch bản hội thoại được sinh/gửi qua queue và đọc lại từ Redis. |

  

---

  

## 3. GOALS, SCOPE & CONSTRAINTS

  

### 3.1 Goals

  

**Business goals**

  

- **Chuẩn hóa** cách gọi tool cho toàn bộ robot.

- **Giảm chi phí phát triển** khi thêm bot/tool mới.

- **Cải thiện chất lượng** (phát âm, matching) nhờ có benchmark chung (`benchmark.py`).

  

**Technical goals**

  

- Thiết kế service **async-first** (FastAPI + aiohttp).

- Dễ mở rộng worker (`robot-ai-tool-worker`) cho các job tốn tài nguyên.

- Dễ monitor performance và lỗi của từng tool.

  

### 3.2 In-Scope

  

- Các endpoint trong `app.py`:

  - `execute` (router tool tổng).

  - `profileMatching`, `profileExtraction`, `properityMatching`, `moodMatching`, `summaryConversation`.

  - `phoneme`, `phonemeSimilarity`.

  - `mem0Generation`, `mem0Search`, `extractFacts`.

  - `checkCallTool`, `completionsCreate`, `makeConversationTemplate`, `getConversationTemplate`.

- Worker xử lý qua `worker_tools.py` + channel RabbitMQ/Redis.

- Docker hóa và chạy cùng Redis trong `docker-compose.yml`.

  

### 3.3 Out-of-Scope / Non-Goals

  

- Không thiết kế UI/frontend.

- Không mô tả chi tiết nội bộ các external services (phoneme API, mem0 server, OpenAI/Groq/Gemini).

- Không chuẩn hóa toàn bộ observability stack (Prometheus/Grafana) – chỉ định hướng.

  

### 3.4 Constraints & Assumptions

  

- Python 3.10 (theo Dockerfile).

- Dựa trên Redis và RabbitMQ đã được cấu hình bởi môi trường triển khai.

- Giao tiếp với external API qua HTTP(s), không ownership về độ sẵn sàng của chúng.

  

---

  

## 4. SYSTEM OVERVIEW

  

### 4.1 Business Context

  

`robot-ai-tool` là **service backend trung gian** cho toàn bộ hệ sinh thái robot:

  

- Robot/lesson/workflow gọi vào một endpoint chuẩn (`/robot-ai-tool/api/v1/tool/…`).

- Service này gọi tiếp các LLM provider, dịch vụ phoneme, mem0, matching logic… và trả kết quả về.

- Một số job chạy **async** qua RabbitMQ (ví dụ makeConversationTemplate, properityMatching sub-jobs).

  

### 4.2 High-Level Components

  

- **FastAPI Application (`app.py`)**

  - Định nghĩa router, models (`InputRequest`, `PhonemeRequest`).

  - Tạo `ToolExecutor` từ `src/tool_executor.py`.

  - Tạo client Redis (`RedisClient`) và RabbitMQ (`RabbitMQClient`).

  - Định nghĩa tất cả các endpoint v1.

  

- **Tool Layer (`src/*`)**

  - `tool_phoneme.py`: gọi phoneme API, tính Levenshtein similarity giữa 2 IPA.

  - `tool_pronunciation_checker.py`: gọi dịch vụ pronunciation check.

  - `mood_matching.py`, `language_matching.py`, `image_matching.py`, `profile_matching.py`: các logic matching theo domain.

  - `mem0_generation.py`, `summary_conversation.py`, `check_call_tool.py`, `tool_grammar_checker.py`: các tool LLM-based.

  - `llm_base.py`: lớp base cho LLM providers, dùng cấu hình trong `config.yml`.

  

- **Execution Orchestrator (`src/tool_executor.py`)**

  - Khởi tạo và giữ instance các tool.

  - Hàm `execute` route theo `tool_name`.

  

- **Async Infrastructure**

  - `src/channel/redis_client.py`: wrapper cho Redis.

  - `src/channel/rabbitmq_client.py` & `src/channel/rabbitmq_consumer.py`: gửi và tiêu thụ job.

  - `worker_tools.py`: chạy worker, lặp lại kết nối nếu lỗi.

  

- **Deployment / Ops**

  - `Dockerfile`, `Dockerfile.worker`: image cho API & worker.

  - `docker-compose.yml`: API + Redis + worker (10 replicas).

  - `requirement.txt`: dependency chính (fastapi, aiohttp, redis, pika, openai, groq…).

  - `Jenkinsfile`: build & push image theo branch/profile.

  

---

  

## 5. HIGH-LEVEL DESIGN (HLD)

  

### 5.1 Architecture Pattern

  

- **Pattern**: Service backend + worker (gần giống microservice + async worker).

- **Phong cách**: RESTful API + async task queue.

- **SOLID áp dụng ở mức module**:

  - Mỗi tool ở file riêng (`tool_phoneme.py`, `tool_pronunciation_checker.py`, v.v.) ⇒ **Single Responsibility**.

  - `ToolExecutor` dùng composition để lắp các tool ⇒ dễ mở rộng (Open/Closed).

  

### 5.2 System Context

  

- **Actors chính**:

  - Robot platform (lesson, agent, workflow).

  - External services: phoneme API (`PHONEME_URL`), pronunciation, mem0 (`URL_PIKA_MEMORY`), OpenAI/Groq/Gemini (qua `config.yml`).

  

- **Data flow tổng quan**:

  1. Client gọi API `/robot-ai-tool/api/v1/tool/<endpoint>`.

  2. FastAPI nhận, validate, log `[START TOOL]`.

  3. Gọi `tool_executor` tương ứng:

     - Nếu job nhanh (phoneme, phonemeSimilarity, mem0Search, summaryConversation…) → xử lý sync trong request.

     - Nếu job async (makeConversationTemplate, properityMatching sub-jobs) → gửi task qua RabbitMQ, dùng Redis để check status.

  4. Trả về JSON `{status, result, msg}` và log `[END TOOL]`.

  

### 5.3 Container & Deployment View

  

- **Containers**

  - `robot-ai-tool`:

    - Chạy `python -u app.py --host $HOST --port $PORT --workers $WORKERS`.

    - Expose cổng (mặc định 9405 trong Dockerfile; bind thực tế qua env).

  - `robot-ai-tool-worker`:

    - Chạy `python -u worker_tools.py`.

    - `docker-compose.yml` cấu hình `deploy.replicas: 10` để scale worker.

  - `redis`:

    - Chạy `redis:7.2.4`, `network_mode: host`, cấu hình từ `.env`.

  

- **CI/CD**

  - `Jenkinsfile` build image với tag dựa trên branch (`prod/staging/dev/...`) và `BUILD_NUMBER`.

  - Push lên GitLab Container Registry.

  

---

  

## 6. LOW-LEVEL DESIGN (LLD) – KEY MODULES

  

### 6.1 `ToolExecutor` (src/tool_executor.py)

  

**Mục đích**

  

- Là **orchestrator** cho tất cả tool.

- Cung cấp:

  - `execute(...)` – thực thi tool theo `tool_name` khi gọi endpoint `/v1/tool/execute`.

  - Property/method cho từng tool chuyên biệt (ví dụ `profile_matching`, `summary_conversation`, `tool_phoneme`…).

  

**Thiết kế**

  

- Nhận `PROVIDER_MODELS` & `TOOL_CONFIG` từ `config.yml`.

- Khởi tạo instance của:

  - `LLMBase` (cho generative tasks).

  - `ToolPhoneme`, `ToolPronunciationChecker`, `ToolGrammarChecker`, `ProfileMatching`, `LanguageMatching`, `MoodMatching`, `ImageMatching`, `SummaryConversation`, `Mem0Generation`, v.v.

- `execute`:

  - Nhận `tool_name`, `audio_url`, `message`, `text_refs`, `question`, `metadata`.

  - Dựa vào `tool_name` để chọn tool tương ứng.

  - Trả về dict hoặc message lỗi.

  

### 6.2 `ToolPhoneme` (src/tool_phoneme.py)

  

**Chức năng**

  

- Gọi service phoneme/API IPA để chuyển `text` thành biểu diễn IPA.

- Tính **Levenshtein similarity** giữa 2 chuỗi IPA để đánh giá độ giống nhau của phát âm (`phoneme_similarity`).

  

**Chi tiết chính**

  

- Thuộc tính:

  - `self.url` (từ env `PHONEME_URL`).

  - `self.timeout` (mặc định 5s).

  - `self.headers` (tạm thời rỗng, có thể mở rộng thêm auth/trace id).

  

- Phương thức:

  - `process(text, lang, mode)`:

    - Dùng `aiohttp.ClientSession` để POST JSON `{text, lang, mode}` tới `self.url`.

    - Log `[TOOLS][INFO] Request Tool: {self.url}`.

    - Trả về `response_json` chứa ít nhất field `ipa`.

  - `tokenize_ipa(ipa_str)`:

    - Tách chuỗi IPA thành token (xử lý combining characters).

  - `levenshtein_similarity(tokens1, tokens2)`:

    - Tính edit distance giữa 2 dãy token, trả về similarity (0–1) + distance.

  - `phoneme_similarity(text_1, text_2)`:

    - Gọi **song song** `process(text_1)` và `process(text_2)` bằng `asyncio.gather`.

    - Nếu cả hai trả về field `ipa` → tính similarity và wrap trong:

      ```json

      {

        "phoneme_similarity": <float>,

        "phoneme": {

          "text_1": "...",

          "text_2": "...",

          "tokens_1": "...ipa...",

          "tokens_2": "...ipa..."

        }

      }

      ```

    - Nếu lỗi hoặc thiếu field → trả `{ "msg": "...", "phoneme": { ...raw... } }`.

  

### 6.3 `ToolPronunciationChecker` & Related Tools

  

**Pronunciation Checker (`tool_pronunciation_checker.py`)**

  

- Nhận input (audio_url, message, text_refs, threshold, metadata).

- Gọi pronunciation API bên ngoài (`TOOL_CONFIG.PRONUNCIATION_CHECKER_TOOL` nếu được cấu hình).

- Chuẩn hóa kết quả về dạng:

  - feedback text (tiếng Việt).

  - chi tiết phoneme word-level.

  

**Matching tools (`mood_matching.py`, `language_matching.py`, `image_matching.py`, `profile_matching.py`)**

  

- Nhận `messages`/`bot_id`/`profile_description`.

- Gọi LLM (qua `LLMBase`) để dự đoán:

  - ngôn ngữ người dùng.

  - mood người dùng.

  - matching giữa conversation và profile/image.

- Trả JSON đã chuẩn hóa để platform dễ dùng.

  

### 6.4 Mem0 & Conversation Tools

  

- `mem0_generation.py`, `mem0_search.py`, `extract_facts` (logic trong `app.py`):

  - Gọi dịch vụ `URL_PIKA_MEMORY` với payload gồm `user_id`, `conversation_id`, `conversation`.

  - Các endpoint:

    - `/generate_response` – sinh response + store memory.

    - `/search_facts` – tìm facts liên quan.

    - `/extract_facts` – trích xuất facts từ metadata.

  

- `summary_conversation.py`:

  - Nhận lịch sử conversation, gọi LLM để sinh summary ngắn.

  

- `check_call_tool.py`:

  - Phân tích conversation history để gợi ý có nên gọi tool khác không (dựa context).

  

---

  

## 7. API DESIGN & CONTRACTS (TÓM TẮT)

  

> Ghi chú: Chi tiết OpenAPI được sinh tự động từ FastAPI; phần này tóm tắt các endpoint chính.

  

### 7.1 Common Pattern

  

- Base route: `/robot-ai-tool/api`.

- Phiên bản: `/v1/tool/...`.

- Response chuẩn:

  

```json

{

  "status": 0,

  "result": { "...": "..." },

  "msg": "Success"

}

```

  

- Lỗi logic/tool:

  - Trả `status: -1`, `msg: <mô tả>`.

  - Với `phoneme` và `phonemeSimilarity`: dùng `JSONResponse` với `status_code` 400 khi input/tool lỗi.

  

### 7.2 Representative Endpoints

  

- **Health check**

  - `GET /robot-ai-tool/api`

  - Output: `{ "status": "OK" }`

  

- **Generic tool execution**

  - `POST /robot-ai-tool/api/v1/tool/execute`

  - Body: `InputRequest`:

    - `conversation_id`, `tool_name`, `audio_url`, `message`, `text_refs`, `question`, `metadata`, …

  - Chuyển tiếp vào `ToolExecutor.execute`.

  

- **Profile/matching & mood/language/image**

  - `POST /v1/tool/profileMatching`

  - `POST /v1/tool/profileExtraction`

  - `POST /v1/tool/properityMatching`

  - `POST /v1/tool/moodMatching`

  - `POST /v1/tool/summaryConversation`

  

- **Phoneme & pronunciation**

  - `POST /v1/tool/phoneme`

    - Body: `PhonemeRequest { text, lang, mode, conversation_id, bot_id }`.

    - Gọi `ToolPhoneme.process`.

  - `POST /v1/tool/phonemeSimilarity`

    - Body: raw JSON `{ conversation_id, text_1, text_2 }`.

    - Gọi `ToolPhoneme.phoneme_similarity`.

  

- **Mem0**

  - `POST /v1/tool/mem0Generation`

  - `POST /v1/tool/mem0Search`

  - `POST /v1/tool/extractFacts`

  

- **LLM completions**

  - `POST /v1/tool/completionsCreate`

  - Lấy messages từ Redis để giữ context, thêm system prompt từ metadata nếu có, gọi LLM (OpenAI/Groq/Gemini) qua `LLMBase`.

  

---

  

## 8. DATA DESIGN (TÓM TẮT)

  

### 8.1 Persistent Stores

  

- **Redis**

  - Lưu:

    - Trạng thái conversation template (`INIT`/`END`).

    - Kết quả `properityMatching` sub-job theo `task_id`.

    - Lịch sử conversation cho `completionsCreate`.

  - Kiểu dữ liệu: string JSON.

  

- **RabbitMQ**

  - Exchange + queue được cấu hình qua env:

    - Message format: JSON `{conversation_id, bot_id, messages, task_id, task_name, job_name}`.

  - Được worker tiêu thụ và ghi kết quả vào Redis.

  

- **Không dùng DB SQL** trong scope này; lưu state ngắn hạn/ephemeral trong Redis.

  

### 8.2 Config

  

- `config.yml`:

  - `PROVIDER_MODELS` cho `groq`, `openai`, `gemini`:

    - `openai_setting.api_key`, `base_url`.

    - `generation_params` mặc định (max_tokens, temperature, top_p, model…).

  - Có phần `TOOL_CONFIG` (commented) cho pronunciation checker – có thể bật khi cần.

  

---

  

## 9. SECURITY DESIGN (TÓM TẮT)

  

### 9.1 Threat Model (Cao cấp)

  

- Service này **không tự xử lý auth** trong code hiện tại:

  - Giả định được bảo vệ bởi API gateway/nginx layer phía trước (auth, rate limit).

- Rủi ro chính:

  - Abuse endpoint để gọi LLM/phoneme/mem0 quá nhiều lần (DoS / cost spike).

  - Input không được sanitize (message, metadata) có thể làm log injection nếu không cẩn thận.

  

### 9.2 Planned Controls

  

- Bổ sung ở layer trước:

  - Auth (API key/JWT) cho tất cả endpoint `/robot-ai-tool/api/v1`.

  - Rate limit theo bot/user.

- Trong service:

  - Chuẩn hóa logging (escape newline trong message nếu cần).

  - Validate kiểu dữ liệu đầu vào (dựa trên Pydantic models nhiều hơn, giảm `Request.json()` thủ công).

  

---

  

## 10. RESILIENCE & RELIABILITY (TÓM TẮT)

  

### 10.1 Timeouts & Retries

  

- `ToolPhoneme`:

  - Timeout HTTP 5s.

  - Hiện chưa có retry, có thể bổ sung pattern retry + circuit breaker cho external services quan trọng.

  

- `mem0*` endpoints:

  - Dùng `aiohttp.ClientTimeout(total=10)` hoặc `5` tùy endpoint.

  

### 10.2 Async Job Handling

  

- `properityMatching`:

  - Gửi 3 task song song (image/language/mood) qua RabbitMQ.

  - API loop chờ tối đa 5s, polling Redis mỗi 0.1s.

  - Nếu không đầy đủ, trả `None` cho field tương ứng.

  

- `makeConversationTemplate`:

  - Gửi message vào RabbitMQ, set Redis status `INIT`.

  - Client dùng `getConversationTemplate` polling đến khi Redis `END` hoặc timeout.

  

---

  

## 11. OBSERVABILITY & MONITORING (ĐỊNH HƯỚNG)

  

### 11.1 Logging

  

- Dùng `logging.basicConfig` với format:

  - `%(asctime)s - %(name)s - %(levelname)s - %(message)s`.

- Pattern log:

  - `[START TOOL] {conversation_id} - {tool_name} - {payload}`

  - `[END TOOL] {conversation_id} - {output}`

  - `[TOOLS][INFO] Request Tool: <url>`

  - Sẵn sàng để ship sang ELK/Datadog.

  

### 11.2 Metrics & Alerts (Future)

  

- Đề xuất:

  - Wrap middleware FastAPI để đo latency/error per endpoint.

  - Metric custom cho:

    - Tỷ lệ lỗi external services (phoneme, mem0, LLM).

    - Thời gian xử lý trung bình mỗi tool.

  

---

  

## 12. DEPLOYMENT & OPERATIONS

  

### 12.1 Docker & Compose

  

- `Dockerfile`:

  - Base `python:3.10`.

  - Cài dependency từ `requirement.txt`.

  - Cài thêm `google-generativeai`.

  - Run `app.py` ghi log ra `resource/log_<timestamp>.txt`.

  

- `Dockerfile.worker`:

  - Tương tự nhưng chạy `worker_tools.py`.

  

- `docker-compose.yml`:

  

  - Service `robot-ai-tool`:

    - Image build sẵn: `registry.gitlab.com/platform-rag/robot-ai-tool/prod:3`.

    - `network_mode: host`.

    - Mount `./dataset` vào `/opt/dataset`.

  

  - Service `redis`:

    - `redis:7.2.4`, cấu hình bằng `.env`, `network_mode: host`.

  

  - Service `robot-ai-tool-worker`:

    - Image worker `worker-prod:3`.

    - `deploy.replicas: 10`.

  

### 12.2 CI/CD (Jenkins)

  

- `Jenkinsfile`:

  - Xác định profile (`prod/staging/dev`) dựa trên branch.

  - Build image `REGISTRY/platform-rag/robot-ai-tool/<profile>:<BUILD_NUMBER>`.

  - Login GitLab Registry, push image.

  - Gửi thông báo Slack cho start/success/failure/aborted.

  

---

  

## 13. TESTING STRATEGY (TÓM TẮT)

  

### 13.1 Unit & Integration Tests (Plan)

  

- Unit test:

  - Các tool logic độc lập: `ToolPhoneme.levenshtein_similarity`, tokenizer IPA, matching logic (mood/language/profile).

  - Mock external API (phoneme, mem0, LLM).

  

- Integration test:

  - Chạy FastAPI app với Redis & RabbitMQ (test containers/local).

  - Test full flow cho từng endpoint: happy path + error path.

  

### 13.2 Benchmark

  

- `benchmark.py` hiện hỗ trợ benchmark cho `properityMatching`:

  - Đọc file test case `language_matching_testcases_*.json`.

  - Gọi API `/robot-ai-tool/api/v1/tool/properityMatching`.

  - Ghi log kết quả, tính accuracy, lưu JSON summary.

  

---

  

## 14. NON-FUNCTIONAL REQUIREMENTS (NFR) – HIGH LEVEL

  

- **Performance**

  - P95 latency cho các endpoint sync (phoneme, mem0Search, summaryConversation…) < 1s trong điều kiện external services ổn định.

  - properityMatching chấp nhận latency cao hơn (vì chờ 3 sub-job và poll Redis).

  

- **Scalability**

  - Scale ngang bằng:

    - Tăng số replicas API.

    - Tăng số worker `robot-ai-tool-worker`.

  - Redis/RabbitMQ scale theo hạ tầng chung.

  

- **Reliability**

  - Sử dụng Redis và RabbitMQ đã được quản lý/monitor bởi team infra.

  - Service cần thêm healthcheck chi tiết (kết nối Redis/RabbitMQ/external) để phục vụ load balancer.

  

---

  

## 15. TRADE-OFFS & OPEN ISSUES

  

### 15.1 Trade-Offs Hiện Tại

  

- **Không có auth trong service**:

  - Ưu: đơn giản; dễ dùng trong internal network.

  - Nhược: phụ thuộc hoàn toàn vào gateway phía trước để bảo vệ.

  

- **Dùng Redis làm state store tạm**:

  - Ưu: nhanh, đơn giản, phù hợp cho cache & short-lived state.

  - Nhược: không phù hợp cho use case yêu cầu lịch sử dài hạn.

  

- **Giao tiếp external API trực tiếp trong tool**:

  - Ưu: code rõ ràng, ít layer.

  - Nhược: mỗi tool tự lo retry/timeout; nên chuẩn hóa pattern (retry/circuit breaker) về sau.

  

### 15.2 Open Issues / Future Work

  

- Bổ sung:

  - Middleware metrics & tracing (OpenTelemetry).

  - Rate limiting / auth layer.

  - Bộ test unit/integration đầy đủ cho từng tool.

  - Runbook chi tiết cho lỗi Redis, RabbitMQ, external APIs.

  

---

  

## 16. IMPLEMENTATION ROADMAP (RÚT GỌN)

  

### 16.1 Phân Phase

  

- **Phase 1 – Stabilize hiện tại**

  - Viết test cơ bản cho `ToolPhoneme`, `phonemeSimilarity`.

  - Thêm healthcheck chi tiết (Redis, RabbitMQ, external API).

  - Chuẩn hóa logging (trace id theo conversation_id).

  

- **Phase 2 – Observability & Security**

  - Thêm metrics + dashboard (latency/error per tool).

  - Thêm auth/rate limit ở gateway, chuẩn hóa error codes.

  

- **Phase 3 – Scale & Optimize**

  - Tối ưu pattern async cho mọi external call (song song hóa khi hợp lý).

  - Thử nghiệm load test cho các endpoint chính (execute, properityMatching, phoneme, mem0Search).

  

---

  

## 17. APPENDIX

  

### 17.1 Mapping Code → Responsibility

  

- `app.py`:

  - Định nghĩa API layer, wiring Redis/RabbitMQ, mapping endpoint → tool_executor.

- `src/tool_executor.py`:

  - Orchestrator cho mọi tool, entry point logic.

- `src/tool_phoneme.py`:

  - Phoneme API client + similarity logic.

- `src/tool_pronunciation_checker.py`:

  - Dịch vụ pronunciation checker.

- `src/mood_matching.py`, `src/language_matching.py`, `src/image_matching.py`, `src/profile_matching.py`:

  - Biz logic cho các loại matching.

- `src/mem0_generation.py`, `src/summary_conversation.py`, `src/check_call_tool.py`, `src/tool_grammar_checker.py`:

  - Tool liên quan LLM & memory.

- `src/channel/*`, `worker_tools.py`:

  - Hạ tầng queue/worker (RabbitMQ/Redis).

- `Dockerfile*`, `docker-compose.yml`, `Jenkinsfile`:

  - Build, run, deploy.

  

### 17.2 Phù hợp Template SDD

  

- Tài liệu này bám theo `SDD_template.md` nhưng **rút gọn**:

  - Giữ các phần quan trọng cho service hiện tại: kiến trúc, API, module chính, NFR, deployment.

  - Có thể mở rộng dần (bổ sung diagram, test strategy chi tiết, incident runbooks) khi hệ thống lớn hơn.
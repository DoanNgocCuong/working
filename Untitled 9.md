Mình sẽ tổng hợp thành một spec ngắn gọn, dễ đọc cho team: mục tiêu, thành phần, luồng chạy, nguyên tắc validate/gating, và kèm đúng 2 sơ đồ ASCII (pipeline tổng + timing pre-commit validator).


# Spec: Automation Agents Pack cho Comet-like Browser (n8n-style)

## 0) Mục tiêu

Xây một “Automation Agents Pack” có thể **thực thi workflow automation** trên các nền tảng:

* Google Sheets, Google Drive, Notion (data hub)
* Facebook (publish)
  Theo triết lý **API-first**, chỉ dùng **UI automation (Playwright/CDP)** làm fallback khi API không đủ.

Pack này **không tạo plan**. Việc tạo plan thuộc về **Planner Agent** ở kiến trúc tổng.

---

## 1) Kiến trúc tổng quan

### 1.1 Vai trò chính

**Planner Agent (outside pack)**

* Nhận intent + constraints từ user
* Sinh **Plan (DAG/steps)**: thứ tự step, mapping input/output, `risk_level` per step
* Không gánh runtime details (retry, rate-limit, idempotency…)

**Automation Agents Pack (inside)**

* Nhận Plan từ Planner và **thực thi** theo step
* Gồm:

  1. **Automation Runtime (Coordinator mỏng)**
     Quản lý state, retries, rate-limit, idempotency, data handoff, chọn specialist, gọi validator khi cần.
  2. **Specialist Agents (theo platform/capability)**
     Sheets / Drive / Notion / Facebook Publisher / Auth / UI Automation
  3. **Validation & Gating**

     * Rule-based validation trong runtime và trong từng specialist (luôn có)
     * Optional **Global Validator Agent (LLM)** chỉ bật khi high-risk/flow phức tạp, chạy ở **pre-commit**

---

## 2) Tách agent theo “điểm gãy” (boundary đúng đau)

### 2.1 Tại sao không gom 1 agent?

Một agent “ôm hết” sẽ nhanh cho demo, nhưng dễ thành spaghetti vì trộn:

* OAuth/scopes
* quota/rate-limit
* schema mapping khác nhau theo nền tảng
* publish vs read (rủi ro khác nhau)
* UI automation brittle

### 2.2 Khuyến nghị thực thi

**P0 (MVP) tách theo platform** (debug dễ, build nhanh):

* Sheets Specialist
* Notion Specialist
* Facebook Publisher Specialist
* (P0 tuỳ chọn) Drive Specialist
* Auth module (dùng chung)
* UI Automation Specialist (fallback)

**P1 trở đi** có thể thêm layer theo capability (Read/Write/Publish/Schedule) nếu mở rộng nhiều platform.

---

## 3) Automation Runtime (Coordinator) — trách nhiệm “n8n runtime”

Runtime không “lập kế hoạch lại”, chỉ lo chạy đáng tin:

* **Step scheduling** (theo DAG hoặc sequence)
* **State store**: `run_id`, `step_id`, status, artifacts
* **Retries + backoff** (429/quota/timeout)
* **Rate-limit + batching**
* **Idempotency** (đảm bảo chạy lại không tạo duplicate)
* **Data handoff**: map output step A → input step B
* **Capability routing**: (connector, action) → specialist
* **Gating** cho các action tác động mạnh (write/publish/delete/permission-change)
* **Pre-commit global validation** (khi risk cao)

---

## 4) Specialists — trách nhiệm & output chuẩn

Mỗi specialist:

* Implement API-first logic cho domain
* Xử lý lỗi đặc thù (quota, schema mismatch, permission)
* Trả về:

  * `artifact` (ví dụ: `post_id`, `page_id`, `file_id`, row count…)
  * `domain_checks` (kết quả validate domain)
  * `telemetry` (timing, rate-limit headers nếu có)

Gợi ý danh sách specialist tối thiểu:

* **Sheets Specialist**: read/write ranges, append rows, batch update
* **Drive Specialist**: list/export/download, permissions/folders
* **Notion Specialist**: query DB, create/update pages/blocks
* **Facebook Publisher Specialist**: publish post/media (và schedule nếu có API)
* **Auth Specialist (module dùng chung)**: OAuth/refresh/scopes
* **UI Automation Specialist**: Playwright/CDP fallback, selector strategy, screenshot proof

---

## 5) Validation & Gating (tối quan trọng)

### 5.1 Hai lớp validate “rule-based” trong pack (luôn có)

1. **Runtime Validation (rule-based)**

* Contract/schema check theo `expected_output_schema`
* Mapping check giữa steps (missing fields, type mismatch)
* Idempotency requirement cho write/publish
* Policy gate: action thuộc nhóm high-risk phải qua gating

2. **Specialist Validation (rule-based)**

* Domain correctness: object shape, counts, checksum/diff, ID trả về hợp lệ, v.v.

=> Hai lớp này deterministic, không cần LLM.

### 5.2 Global Validator Agent (LLM) — chỉ bật khi cần

Dùng khi:

* **High-stakes**: publish/delete/permission-change/bulk write lớn
* **UI automation fallback** (dễ “làm sai mà tưởng đúng”)
* Flow phức tạp, nhiều mapping, cần “second opinion” + audit reasoning

**Điểm đặt đúng:** Global Validator chạy ở **pre-commit** (trước hành động khó hoàn tác).
Nếu chạy “sau khi đã publish rồi” thì chỉ còn audit/alert, không còn là lớp bảo vệ chính.

### 5.3 Nguyên tắc xử lý fail (tránh loop rối)

* Fail do domain/data/selector → gửi lại **đúng specialist** rerun step đó
* Fail do policy/risk → dừng, yêu cầu user confirm hoặc sửa intent
* Chỉ đẩy về Planner nếu plan sai cấu trúc nghiêm trọng (hiếm)

---

## 6) Contract tối thiểu giữa Planner và Pack

Planner xuất Plan gồm các step có trường tối thiểu:

* `run_id`, `step_id`
* `connector`: sheets|drive|notion|facebook|ui
* `action`: read|query|write|publish|delete|permission_change
* `params`: payload cụ thể
* `inputs_mapping`: lấy data từ output step trước
* `risk_level`: low|medium|high
* `constraints`: max_retries, timeout, rate_limit_hint…
* `idempotency_key` (bắt buộc cho write/publish)
* `expected_output_schema`

---

## 7) Đồ thị pipeline workflow (tổng quan hệ thống)

```text
┌──────────────────────────────────────────────────────────────────────────────┐
│                               USER / PRODUCT UI                              │
└───────────────┬──────────────────────────────────────────────────────────────┘
                │  (Goal / Intent / Constraints)
                v
┌──────────────────────────────────────────────────────────────────────────────┐
│                               PLANNER AGENT                                  │
│  - tạo PLAN (DAG/steps) + mapping input/output + risk_level per step          │
│  - KHÔNG làm runtime retries/rate-limit/idempotency                           │
└───────────────┬──────────────────────────────────────────────────────────────┘
                │  Plan JSON
                v
╔══════════════════════════════════════════════════════════════════════════════╗
║                        AUTOMATION AGENTS PACK (Executor)                     ║
║                                                                              ║
║   ┌───────────────────────────────┐                                          ║
║   │     AUTOMATION RUNTIME        │  state/retry/rate-limit/idempotency       ║
║   └───────────────┬───────────────┘                                          ║
║                   │                                                           ║
║                   v                                                           ║
║   ┌───────────────────────────────┐                                          ║
║   │        GATING / POLICY        │  (write/publish/delete -> stricter)       ║
║   └───────────────┬───────────────┘                                          ║
║                   │                                                           ║
║                   v                                                           ║
║   ┌───────────────────────────────┐                                          ║
║   │      CAPABILITY REGISTRY      │  (connector,action)->specialist           ║
║   └───────────────┬───────────────┘                                          ║
║                   │ route                                                      ║
║        ┌──────────┼───────────┬───────────┬─────────────┬───────────┐        ║
║        v          v           v           v             v           v        ║
║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐║
║  │ Sheets   │ │ Drive    │ │ Notion   │ │ Facebook │ │ Auth     │ │ UI     │║
║  │Specialist│ │Specialist│ │Specialist│ │Publisher │ │Specialist│ │Auto    │║
║  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘ └───┬────┘║
║       │ API-first        │ API-first       │ API-first       │          │     ║
║       └──────────────────────────────────────────────────────────────┐  │     ║
║                                                                      v  v     ║
║                ┌──────────┐ ┌──────────┐      ┌──────────────┐   ┌──────────┐║
║                │  Google  │ │  Notion  │      │ Meta Graph API │   │Browser/  │║
║                │ APIs     │ │  APIs    │      │ (or official)  │   │CDP/PW   │║
║                └──────────┘ └──────────┘      └──────────────┘   └──────────┘║
║                                                                              ║
║   (Fallback) If API missing/blocked -> route -> UI Automation                 ║
║   (Rule-based validation) Runtime + Specialist domain checks                  ║
╚══════════════════════════════════════════════════════════════════════════════╝
                │
                v
┌──────────────────────────────────────────────────────────────────────────────┐
│                           OUTPUT / UX FEEDBACK                               │
│  - result summary + artifacts + run logs                                      │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 8) Sơ đồ timing gọn (mental model đúng)

```text
Planner -> Automation Pack(Runtime+Specialists rule-check)
                         |
                         | if risk LOW: commit directly
                         |
                         | if risk HIGH:
                         v
                   (Pre-commit package)
                         |
                    Message Hub (optional)
                         |
                     Global Validator Agent (LLM)
                    PASS / FAIL / NEED-USER-CONFIRM
                         |
                         v
                   Runtime commits/publishes
                         |
                         v
                    Post-run Audit (optional)
               (message hub -> audit/monitoring/report)
```

**Ghi chú quan trọng:**

* “Global Validator Agent” dùng để **gating trước commit** khi high-risk.
* “Validate sau message hub” nên coi là **audit/monitor**, không phải lớp bảo vệ chính (vì lúc đó có thể đã publish rồi).

---

## 9) Roadmap đề xuất

**P0**

* Runtime + Registry + 3 specialists (Sheets, Notion, Facebook) + Auth
* Rule-based validation + gating cho publish/write
* Log run/step rõ ràng

**P1**

* Drive specialist
* UI automation fallback specialist
* Global Validator Agent pre-commit cho high-risk
* Replay/re-run step theo `run_id`

---

## 10) Kết luận

Thiết kế tối ưu: **Planner sinh plan** → **Automation Agents Pack** thực thi với **Runtime (đáng tin)** + **Specialists (đúng domain)** + **rule-based validation**; khi high-risk thì gọi **Global Validator Agent** ở **pre-commit** để chặn sai trước khi gây hậu quả.

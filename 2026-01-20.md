# Logging Standards - Chuẩn hóa Log cho các Service

> **Mục đích**: Tài liệu định nghĩa chuẩn viết log JSON cho tất cả các service trong hệ thống. Áp dụng cho mọi ngôn ngữ lập trình.

**Version**: 1.0
**Cập nhật**: 2026-01-20

---

## Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [Cấu trúc JSON Log](#2-cấu-trúc-json-log)
3. [Hướng dẫn xác định giá trị các field](#3-hướng-dẫn-xác-định-giá-trị-các-field)
4. [Quy định Log Level](#4-quy-định-log-level)
5. [Chi tiết theo loại Log](#5-chi-tiết-theo-loại-log)
6. [Quy tắc đặt tên](#6-quy-tắc-đặt-tên)
7. [Bảo mật](#7-bảo-mật)
8. [Checklist](#8-checklist)

---

## 1. Tổng quan

### 1.1 Tại sao cần chuẩn hóa log?

- **Truy vấn dễ dàng**: Log có cấu trúc giúp filter, search nhanh trên Datadog/ELK
- **Correlation**: Liên kết log giữa các service qua trace_id
- **Monitoring**: Tạo dashboard, alert dựa trên các field chuẩn
- **Debug**: Nhanh chóng xác định vấn đề khi có incident

### 1.2 Format output

- **JSON format**: Mỗi dòng log = 1 JSON object (không có newline trong JSON)
- **Encoding**: UTF-8
- **Output**: Console (stdout)

```json
{"@timestamp":"2024-01-20T10:30:45.123+07:00","level":"INFO","message":"Request completed","log_type":"api","feature":"USER","event":"API_REQUEST_COMPLETE","duration_ms":"150"}
```

---

## 2. Cấu trúc JSON Log

### 2.1 Fields bắt buộc

Tất cả log **PHẢI** có các field sau:

| Field | Type | Mô tả | Ví dụ |
|-------|------|-------|-------|
| `@timestamp` | string | Thời gian, ISO 8601 với timezone | `"2024-01-20T10:30:45.123+07:00"` |
| `level` | string | Log level | `"DEBUG"`, `"INFO"`, `"WARN"`, `"ERROR"` |
| `message` | string | Mô tả ngắn gọn, dễ đọc | `"User login successful"` |
| `log_type` | string | Phân loại log | `"api"`, `"websocket"`, `"external_api"`, `"job"` |
| `feature` | string | Tính năng/module | `"AUTH"`, `"PAYMENT"`, `"ORDER"` |
| `event` | string | Tên event cụ thể | `"LOGIN_SUCCESS"`, `"ORDER_CREATED"` |
| `service` | string | Tên service | `"user-service"`, `"payment-service"` |
| `environment` | string | Môi trường | `"dev"`, `"staging"`, `"prod"` |

### 2.2 Fields context (nếu có)

Thêm vào khi có thông tin context:

| Field | Type | Mô tả |
|-------|------|-------|
| `user_id` | string | ID người dùng đang thực hiện action |
| `request_id` | string | ID duy nhất của request (UUID) |
| `session_id` | string | Session ID (WebSocket, HTTP session) |
| `correlation_id` | string | ID để trace across services |

### 2.3 Fields tracing (APM integration)

Nếu sử dụng Datadog APM hoặc tương tự:

| Field | Type | Mô tả |
|-------|------|-------|
| `dd.trace_id` | string | Datadog trace ID |
| `dd.span_id` | string | Datadog span ID |
| `trace_id` | string | Generic trace ID (OpenTelemetry) |
| `span_id` | string | Generic span ID |

### 2.4 Fields error (khi có lỗi)

| Field | Type | Mô tả |
|-------|------|-------|
| `error_type` | string | Tên exception/error class |
| `error_message` | string | Error message |
| `stack_trace` | string | Stack trace (chỉ cho ERROR level) |

---

## 3. Hướng dẫn xác định giá trị các field

### 3.1 `log_type` - Phân loại theo loại operation

Chọn **MỘT** giá trị phù hợp nhất:

| log_type | Khi nào dùng | Ví dụ |
|----------|--------------|-------|
| `api` | HTTP request/response | REST API, GraphQL |
| `websocket` | WebSocket connection và message | Real-time chat, notifications |
| `external_api` | Gọi API/service bên ngoài | Call OpenAI, gọi payment gateway |
| `grpc` | gRPC calls | Microservice communication |
| `job` | Background job, scheduled task | Cron job, queue consumer |
| `event` | Event publish/consume | Kafka, RabbitMQ events |
| `database` | Database operations | Query, transaction |
| `cache` | Cache operations | Redis get/set |

**Quy tắc**: Chọn dựa trên **loại operation đang thực hiện**, không phải endpoint.

### 3.2 `feature` - Phân loại theo business domain

**Quy tắc xác định**:

1. **Dựa vào URL path prefix** (khuyến nghị cho API):
   ```
   /api/auth/*     → AUTH
   /api/users/*    → USER
   /api/orders/*   → ORDER
   /api/payments/* → PAYMENT
   ```

2. **Dựa vào module/package name**:
   ```
   com.example.auth.*    → AUTH
   com.example.order.*   → ORDER
   services/auth/*       → AUTH
   ```

3. **Dựa vào business function**:
   ```
   Xử lý đăng nhập      → AUTH
   Xử lý thanh toán     → PAYMENT
   Gửi notification     → NOTIFICATION
   ```

**Quy ước đặt tên**:
- UPPERCASE
- Ngắn gọn, 1-2 từ
- Không chứa ký tự đặc biệt
- Ví dụ: `AUTH`, `USER`, `ORDER`, `PAYMENT`, `NOTIFICATION`, `REPORT`

**Ví dụ mapping phổ biến**:

| Path Pattern | Feature |
|--------------|---------|
| `/auth/*`, `/login`, `/logout` | `AUTH` |
| `/users/*`, `/profile/*` | `USER` |
| `/orders/*`, `/checkout/*` | `ORDER` |
| `/payments/*`, `/transactions/*` | `PAYMENT` |
| `/products/*`, `/catalog/*` | `PRODUCT` |
| `/notifications/*` | `NOTIFICATION` |
| `/reports/*`, `/analytics/*` | `REPORT` |
| `/admin/*` | `ADMIN` |
| `/health`, `/metrics` | `SYSTEM` |

### 3.3 `event` - Tên event cụ thể

**Format**: `{ACTION}_{RESULT}` hoặc `{COMPONENT}_{ACTION}_{RESULT}`

#### Actions phổ biến

| Action | Mô tả |
|--------|-------|
| `CREATE` | Tạo mới resource |
| `UPDATE` | Cập nhật resource |
| `DELETE` | Xóa resource |
| `GET` | Lấy thông tin |
| `LIST` | Lấy danh sách |
| `LOGIN` | Đăng nhập |
| `LOGOUT` | Đăng xuất |
| `SEND` | Gửi (email, notification) |
| `PROCESS` | Xử lý |
| `VALIDATE` | Kiểm tra, xác thực |

#### Results (suffix)

| Result | Mô tả | Log Level |
|--------|-------|-----------|
| `_START` | Bắt đầu operation | DEBUG |
| `_ATTEMPT` | Bắt đầu (thường dùng cho auth) | DEBUG |
| `_SUCCESS` | Thành công | INFO |
| `_COMPLETE` | Hoàn thành | INFO |
| `_FAILED` | Thất bại | ERROR |
| `_ERROR` | Lỗi xảy ra | ERROR |

#### Ví dụ event names

```
LOGIN_ATTEMPT        -> Bắt đầu đăng nhập
LOGIN_SUCCESS        -> Đăng nhập thành công
LOGIN_FAILED         -> Đăng nhập thất bại

ORDER_CREATE_START   -> Bắt đầu tạo order
ORDER_CREATE_SUCCESS -> Tạo order thành công
ORDER_CREATE_FAILED  -> Tạo order thất bại

API_REQUEST_START    -> Bắt đầu xử lý request
API_REQUEST_COMPLETE -> Hoàn thành request
API_REQUEST_ERROR    -> Lỗi xử lý request
```

---

## 4. Quy định Log Level

### 4.1 Bảng quy định

| Level | Khi nào dùng | Event suffix | Production |
|-------|--------------|--------------|------------|
| `DEBUG` | Bắt đầu operation, thông tin debug | `*_START`, `*_ATTEMPT` | Có thể tắt |
| `INFO` | Operation thành công, business events | `*_SUCCESS`, `*_COMPLETE` | Bật |
| `WARN` | Vấn đề có thể recover, cần chú ý | Rate limit, retry, fallback | Bật |
| `ERROR` | Lỗi, exception, cần xử lý | `*_FAILED`, `*_ERROR` | Bật |

### 4.2 Flow chuẩn

```
DEBUG: *_START / *_ATTEMPT     (bắt đầu)
         ↓
    [Xử lý logic]
         ↓
INFO:  *_SUCCESS / *_COMPLETE  (nếu thành công)
ERROR: *_FAILED / *_ERROR      (nếu thất bại)
```

### 4.3 Ví dụ

```
DEBUG → "Processing payment for order #123"
INFO  → "Payment successful for order #123"
ERROR → "Payment failed for order #123: Card declined"
```

---

## 5. Chi tiết theo loại Log

### 5.1 API Logging (`log_type: "api"`)

**Dùng cho**: HTTP request/response

#### Fields bổ sung

| Field | Type | Bắt buộc | Mô tả |
|-------|------|----------|-------|
| `http_method` | string | Có | `"GET"`, `"POST"`, `"PUT"`, `"DELETE"` |
| `endpoint` | string | Có | URL path (không include query string) |
| `status_code` | string | Có* | HTTP status code (*cho response) |
| `duration_ms` | string | Có* | Thời gian xử lý (*cho response) |
| `client_ip` | string | Không | IP client |
| `user_agent` | string | Không | User agent |

#### Events chuẩn

| Event | Level | Khi nào |
|-------|-------|---------|
| `API_REQUEST_START` | DEBUG | Nhận request |
| `API_REQUEST_COMPLETE` | INFO | Trả response 2xx |
| `API_REQUEST_COMPLETE` | ERROR | Trả response 4xx/5xx |
| `API_REQUEST_ERROR` | ERROR | Exception xảy ra |

#### Ví dụ

```json
{
  "@timestamp": "2024-01-20T10:30:45.273+07:00",
  "level": "INFO",
  "message": "Request completed",
  "log_type": "api",
  "feature": "ORDER",
  "event": "API_REQUEST_COMPLETE",
  "http_method": "POST",
  "endpoint": "/api/orders",
  "status_code": "201",
  "duration_ms": "150",
  "user_id": "user_123",
  "service": "order-service",
  "environment": "prod"
}
```

---

### 5.2 WebSocket Logging (`log_type: "websocket"`)

**Dùng cho**: WebSocket connections

#### Fields bổ sung

| Field | Type | Bắt buộc | Mô tả |
|-------|------|----------|-------|
| `ws_event` | string | Có | `"connect"`, `"disconnect"`, `"message"`, `"error"` |
| `session_id` | string | Có | WebSocket session ID |
| `endpoint` | string | Có | WS endpoint |
| `connection_duration_ms` | string | Không | Thời gian kết nối (cho disconnect) |
| `close_code` | string | Không | WebSocket close code |
| `close_reason` | string | Không | Close reason |
| `message_type` | string | Không | `"text"`, `"binary"` |
| `message_size` | string | Không | Kích thước message (bytes) |

#### Events chuẩn

| Event | Level | Khi nào |
|-------|-------|---------|
| `WEBSOCKET_CONNECT` | INFO | Client kết nối thành công |
| `WEBSOCKET_DISCONNECT` | INFO | Client ngắt kết nối |
| `WEBSOCKET_MESSAGE` | DEBUG | Nhận/gửi message |
| `WEBSOCKET_ERROR` | ERROR | Lỗi xảy ra |

#### Ví dụ

```json
{
  "@timestamp": "2024-01-20T10:30:45.123+07:00",
  "level": "INFO",
  "message": "Client connected",
  "log_type": "websocket",
  "feature": "CHAT",
  "event": "WEBSOCKET_CONNECT",
  "ws_event": "connect",
  "session_id": "ws_abc123",
  "endpoint": "/ws/chat",
  "user_id": "user_123",
  "service": "chat-service",
  "environment": "prod"
}
```

---

### 5.3 External API Logging (`log_type: "external_api"`)

**Dùng cho**: Gọi API/service bên ngoài

#### Fields bổ sung

| Field | Type | Bắt buộc | Mô tả |
|-------|------|----------|-------|
| `target_service` | string | Có | Tên service đích |
| `target_endpoint` | string | Có | URL đích (có thể mask sensitive parts) |
| `http_method` | string | Có | HTTP method |
| `status_code` | string | Có* | Response status (*cho response) |
| `duration_ms` | string | Có* | Thời gian gọi (*cho response) |
| `is_timeout` | string | Không | `"true"` nếu timeout |
| `retry_count` | string | Không | Số lần retry |

#### Events chuẩn

| Event | Level | Khi nào |
|-------|-------|---------|
| `EXTERNAL_API_REQUEST` | DEBUG/INFO | Bắt đầu gọi |
| `EXTERNAL_API_RESPONSE` | INFO | Response thành công |
| `EXTERNAL_API_RESPONSE` | WARN | Response non-2xx |
| `EXTERNAL_API_ERROR` | ERROR | Exception, timeout |

#### Ví dụ

```json
{
  "@timestamp": "2024-01-20T10:30:45.523+07:00",
  "level": "INFO",
  "message": "External API call completed",
  "log_type": "external_api",
  "feature": "PAYMENT",
  "event": "EXTERNAL_API_RESPONSE",
  "target_service": "stripe",
  "target_endpoint": "https://api.stripe.com/v1/charges",
  "http_method": "POST",
  "status_code": "200",
  "duration_ms": "350",
  "service": "payment-service",
  "environment": "prod"
}
```

---

### 5.4 Job Logging (`log_type: "job"`)

**Dùng cho**: Background jobs, scheduled tasks

#### Fields bổ sung

| Field | Type | Bắt buộc | Mô tả |
|-------|------|----------|-------|
| `job_name` | string | Có | Tên job |
| `job_id` | string | Không | ID của job instance |
| `duration_ms` | string | Có* | Thời gian chạy (*cho complete) |
| `items_processed` | string | Không | Số items đã xử lý |
| `items_failed` | string | Không | Số items lỗi |

#### Events chuẩn

| Event | Level | Khi nào |
|-------|-------|---------|
| `JOB_START` | INFO | Job bắt đầu |
| `JOB_COMPLETE` | INFO | Job hoàn thành |
| `JOB_FAILED` | ERROR | Job thất bại |
| `JOB_PROGRESS` | DEBUG | Progress update |

#### Ví dụ

```json
{
  "@timestamp": "2024-01-20T10:30:45.123+07:00",
  "level": "INFO",
  "message": "Job completed successfully",
  "log_type": "job",
  "feature": "REPORT",
  "event": "JOB_COMPLETE",
  "job_name": "daily_report_generator",
  "job_id": "job_xyz789",
  "duration_ms": "45000",
  "items_processed": "1500",
  "service": "report-service",
  "environment": "prod"
}
```

---

## 6. Quy tắc đặt tên

### 6.1 Field names

- **snake_case**: `user_id`, `http_method`, `duration_ms`
- Ngoại lệ: `@timestamp` (convention của logging frameworks)

### 6.2 Event names

- **UPPER_SNAKE_CASE**: `LOGIN_SUCCESS`, `ORDER_CREATE_FAILED`
- Format: `{ACTION}_{RESULT}` hoặc `{COMPONENT}_{ACTION}_{RESULT}`

### 6.3 Feature names

- **UPPERCASE**: `AUTH`, `USER`, `ORDER`
- Ngắn gọn, 1-2 từ

### 6.4 Service names

- **lowercase với dấu gạch ngang**: `user-service`, `payment-gateway`

### 6.5 Values

| Type | Convention | Ví dụ |
|------|------------|-------|
| Boolean | String `"true"` / `"false"` | `"is_timeout": "true"` |
| Number | String format | `"duration_ms": "150"` |
| Null/Empty | Không include field | (bỏ qua field) |
| Timestamp | ISO 8601 với timezone | `"2024-01-20T10:30:45.123+07:00"` |

---

## 7. Bảo mật

### 7.1 Dữ liệu KHÔNG được log

- Password, secret key, API key
- Full credit card number
- Personal documents (CMND, passport)
- Full session token, JWT token

### 7.2 Dữ liệu cần MASK

| Data | Cách mask | Ví dụ |
|------|-----------|-------|
| Phone number | Giữ 4 số cuối | `"******5678"` |
| Email | Giữ domain | `"u***@gmail.com"` |
| Credit card | Giữ 4 số cuối | `"****1234"` |
| IP address | Có thể giữ nguyên hoặc mask 2 octet cuối | `"192.168.x.x"` |

### 7.3 Ví dụ

```json
{
  "phone": "******5678",
  "email": "n***@example.com",
  "card_last4": "1234"
}
```

---

## 8. Checklist

### 8.1 Trước khi implement

- [ ] Xác định `log_type` (api, websocket, external_api, job, ...)
- [ ] Xác định `feature` dựa trên path/module
- [ ] List các events cần log (START → SUCCESS/FAILED pattern)
- [ ] Xác định fields cần thiết cho mỗi event
- [ ] Kiểm tra log level phù hợp

### 8.2 Trong khi implement

- [ ] Tất cả log đều có đủ fields bắt buộc
- [ ] `duration_ms` có trong các response/complete events
- [ ] Error logs có `error_type` và `error_message`
- [ ] Dữ liệu nhạy cảm được mask

### 8.3 Sau khi implement

- [ ] Test log output đúng JSON format (valid JSON)
- [ ] Log có thể parse được bởi logging system
- [ ] Verify trên dev/staging trước khi lên production

---

## Quick Reference

### Log Level

```
DEBUG → Bắt đầu (_START, _ATTEMPT)
INFO  → Thành công (_SUCCESS, _COMPLETE)
WARN  → Cảnh báo (có thể recover)
ERROR → Lỗi (_FAILED, _ERROR)
```

### Template JSON tối thiểu

```json
{
  "@timestamp": "ISO8601_TIMESTAMP",
  "level": "INFO",
  "message": "Human readable message",
  "log_type": "api|websocket|external_api|job",
  "feature": "FEATURE_NAME",
  "event": "EVENT_NAME",
  "service": "service-name",
  "environment": "prod"
}
```

### Feature mapping từ path

```
/auth/*     → AUTH
/users/*    → USER
/orders/*   → ORDER
/payments/* → PAYMENT
/admin/*    → ADMIN
/health     → SYSTEM
```

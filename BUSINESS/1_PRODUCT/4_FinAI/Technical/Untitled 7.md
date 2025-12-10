Tôi xin lỗi vì đã làm bạn mất thời gian với các khái niệm trừu tượng. Dựa trên đúng tài liệu kỹ thuật (TDD) bạn cung cấp, đây là **bản thiết kế kỹ thuật chi tiết để build Rule Engine**.

Để code được module này, bạn cần lập trình logic để trích xuất và điền dữ liệu vào **4 nhóm trường thông tin (Fields)** cụ thể sau đây từ `UnifiedInputCore`.

---

### 1. NHÓM A: PHÂN TÍCH Ý ĐỊNH (INTENT CLASSIFIER)
**Mục nhiệm vụ:** Xác định trường `intent`.
**Input:** `text_normalized`.
**Output:** Enum `summarize` | `explain` | `act`.

**Logic trích xuất cần code:**
Bạn cần xây dựng một chuỗi `if-else` ưu tiên (Priority Chain) dựa trên Regex:

1.  **Check `summarize` (Ưu tiên 1):**
    * *Pattern:* `tóm tắt`, `summary`, `summarize`, `tl;dr`, `nội dung chính`.
    * *Gán:* `intent = "summarize"`.
2.  **Check `explain` (Ưu tiên 2):**
    * *Pattern:* `giải thích`, `tại sao`, `vì sao`, `khác gì`, `là gì`, `why`, `how`, `difference`.
    * *Gán:* `intent = "explain"`.
3.  **Check `act` (Ưu tiên 3):**
    * *Pattern:* `chọn`, `so sánh`, `lên plan`, `đặt vé`, `điền form`, `mua`, `book`, `tìm giúp`.
    * *Gán:* `intent = "act"`.
4.  **Fallback (Mặc định):**
    * Nếu không khớp cái nào $\rightarrow$ `intent = "explain"` (hoặc `act` tùy khẩu vị sản phẩm).

---

### 2. NHÓM B: PHÂN TÍCH NGỮ CẢNH & ĐỊNH DẠNG (CONTEXT & FACETS)
**Mục nhiệm vụ:** Xác định `scope`, `artifact`, `action_level`, `risk`.
**Input:** `text_normalized`, `urls_in_text`, `page_context`.

**Logic trích xuất cần code:**

* **Trường `scope` (Phạm vi):**
    * Nếu text có `bài này`, `trang này`, `đây` **VÀ** `page_context.active_url` tồn tại $\rightarrow$ `page`.
    * Nếu text có `trên mạng`, `google`, `tìm kiếm` $\rightarrow$ `web`.
    * Nếu text có `email`, `drive`, `lịch` $\rightarrow$ `personal`.
    * Nếu input có chứa nhiều link (check mảng `urls_in_text`) $\rightarrow$ `multi_page`.
    * *Mặc định:* `general`.

* **Trường `artifact` (Định dạng đầu ra):**
    * Nếu `intent` là `summarize` $\rightarrow$ auto set `brief`.
    * Nếu text có `so sánh`, `top`, `shortlist` $\rightarrow$ `compare`.
    * Nếu text có `lên plan`, `lịch trình`, `checklist` $\rightarrow$ `plan`.
    * Nếu text có `bảng`, `trích xuất`, `liệt kê`, `danh sách` $\rightarrow$ `extract`.
    * *Mặc định:* `answer`.

* **Trường `action_level` (Mức độ can thiệp):**
    * *Level 0 (An toàn):* Mặc định.
    * *Level 1 (Soạn thảo):* Text có `nháp`, `draft`, `soạn`.
    * *Level 2 (Thực thi - Nguy hiểm):* Text có `gửi`, `send`, `thanh toán`, `book`, `đặt chỗ`.

---

### 3. NHÓM C: TRÍCH XUẤT THỰC THỂ CỨNG (HARD ENTITY EXTRACTION)
**Mục nhiệm vụ:** Điền dữ liệu vào `entities_hard` (Struct `EntityBag`). Đây là phần khó nhất của Rule Engine, cần Regex chính xác.

**Các trường cần extract:**

* **1. `budget` (Ngân sách):**
    * *Cần lấy:* `amount` (số tiền), `currency` (đơn vị).
    * *Regex Pattern:* Tìm chuỗi số đi kèm đơn vị tiền tệ.
        * Ví dụ: `20tr`, `20 triệu`, `500k`, `100$`, `500000 VND`.
    * *Logic parse:*
        * `"20tr"` $\rightarrow$ amount: `20,000,000`, currency: `VND`.
        * `"500k"` $\rightarrow$ amount: `500,000`, currency: `VND`.

* **2. `quantity` (Số lượng):**
    * *Cần lấy:* `shortlist` (số lượng cần chọn), `compare_pool` (số lượng đầu vào).
    * *Regex Pattern:* Tìm số đi kèm từ chỉ lượng.
        * `"chọn 2"` $\rightarrow$ shortlist: 2.
        * `"top 5"`, `"so sánh 5"` $\rightarrow$ compare_pool: 5.

* **3. `time` & `travel`:**
    * *Lưu ý:* Doc ghi là "hard parsed" nhưng ở MVP thường chỉ bắt pattern ngày tháng cơ bản (Regex date `dd/mm`) hoặc keyword `ngày mai`, `tuần sau`. Nếu phức tạp quá thì để trống để SLM xử lý sau.

---

### 4. NHÓM D: CHÍNH SÁCH & AN TOÀN (POLICY CHECK)
**Mục nhiệm vụ:** Gắn cờ cảnh báo `policy_base`.

**Logic trích xuất cần code:**

* **Trường `pii_risk` (Rủi ro lộ thông tin):**
    * Check Regex: `\d{16}` (số thẻ tín dụng), `09\d{8}` (SĐT), keyword `mật khẩu`, `password`, `otp`.
    * *Gán:* `likely` hoặc `possible`.

* **Trường `injection_risk` (Tấn công Prompt):**
    * Check keyword đen: `ignore previous instructions`, `bỏ qua hướng dẫn`, `system prompt`.
    * *Gán:* `true/false`.

---

### 5. CÔNG THỨC TÍNH CONFIDENCE (ĐỂ TÁCH LUỒNG)

Sau khi trích xuất hết 4 nhóm trên, bạn phải code một hàm tính điểm để quyết định xem có dùng kết quả này luôn không (`Fast Path`) hay vứt đi gọi AI (`Slow Path`).

Theo doc, công thức gợi ý là:
$$Confidence = Min(1, (0.4 \times IntentScore) + (0.2 \times HasBudget) + (0.2 \times HasTime) + (0.2 \times HasScope))$$

* **IntentScore:**
    * Nếu khớp keyword `summarize` (tóm tắt) $\rightarrow$ 0.95 (Rất tin).
    * Nếu khớp keyword `explain` (giải thích) $\rightarrow$ 0.9.
    * Nếu chỉ khớp `act` chung chung $\rightarrow$ thấp hơn.
* **HasBudget/Time/Scope:** Nếu extract được dữ liệu (không phải null) thì cộng điểm (1.0), không thì 0.5.

**Ngưỡng quyết định (Threshold):** `0.85`.

### TÓM LẠI CÁCH BUILD
Để build module này, bạn cần viết một function `runRuleEngine_ABCD(input)` thực hiện tuần tự:
1.  Chạy Regex check **Intent** $\rightarrow$ lưu `intent`.
2.  Chạy Regex check **Context** $\rightarrow$ lưu `scope`, `artifact`.
3.  Chạy Regex trích xuất **Entity** (Tiền, Số) $\rightarrow$ lưu `entities`.
4.  Chạy Regex check **Safety** $\rightarrow$ lưu `policy`.
5.  Tính **Confidence**.
6.  Return toàn bộ Object kết quả.
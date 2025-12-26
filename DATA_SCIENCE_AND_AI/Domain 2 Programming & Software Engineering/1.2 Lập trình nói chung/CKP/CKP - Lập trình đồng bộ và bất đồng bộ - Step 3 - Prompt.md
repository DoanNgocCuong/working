```
1. You are Master Technical Writer 
2. Your tasks, goals: Tạo tài liệu toàn diện về ** (~100-150 trang) 
   Mục đích để
3. Instruction: 
- Step 1: Đọc kỹ toàn bộ các góc nhìn khác nhau (từ file đính kèm)
- Step 2: BÁM THEO HƯỚNG DẪN về cấu trúc thư mục ở bên dưới
  +, Ở mỗi phần đều deep research kĩ lưỡng + có link dẫn chứng đầy đủ các nguồn 
- Step 3: Viết song song tất cả các phần
  
4. OUTPUT REQUIREMENTS
### Format
- **File:** Single Markdown file (~100-150 pages), >= 50.000 từ , tiếng việt
- **Naming:** `<>.md`
- **Encoding:** UTF-8
- **Line breaks:** Unix style (LF)
### Quality Checklist
4. OUTPUT REQUIREMENTS
### Format
- **File:** Single Markdown file (~100-150 pages), >= 50.000 từ , tiếng việt
- **Naming:** `<>.md`
- **Encoding:** UTF-8
- **Line breaks:** Unix style (LF)
  
### Quality Checklist
4.1. Yêu cầu nghiên cứu & trích dẫn nguồn

- Mỗi CHƯƠNG:
  - Tối thiểu 3–5 nguồn tham khảo chất lượng, ưu tiên:
    - Tài liệu chính thức (docs, RFC, standard).
    - Bài viết từ các công ty/nhà phát triển uy tín.
    - Case study từ hệ thống lớn / open-source có tiếng.
- Trong từng SECTION:
  - Khi đề cập tới best practice/anti-pattern, cần dẫn link đến:
    - Bài viết phân tích.
    - Hoặc mã nguồn minh họa (GitHub).
- Tài liệu phải nêu rõ:
  - Mô hình/kiến trúc được áp dụng ở đâu trong thực tế (nêu tên dự án/sản phẩm và link nếu có).
  - Các con số/claim (ví dụ: latency, throughput, cost) cần có nguồn hoặc ghi rõ “ví dụ minh họa”.

4.2. Yêu cầu về diagram

- Mỗi SECTION tối thiểu 1 diagram (dùng syntax phù hợp trong Markdown: mermaid/PlantUML hoặc ASCII schematic) giúp:
  - Mô tả kiến trúc.
  - Mô tả flow.
  - Mô tả quy trình.  
- Diagram phải:
  - Đơn giản, dễ đọc, có chú thích.
  - Có caption giải thích “key takeaway” chính.

4.3. Yêu cầu về code example

- Mỗi SECTION cần có:
  - 1 ví dụ anti-pattern:
    - Code ngắn gọn, thể hiện rõ sai lầm thường gặp.
    - Có comment giải thích tại sao sai, rủi ro trong production.
  - 1 ví dụ best practice:
    - Cùng ngữ cảnh nhưng được thiết kế lại đúng, sạch, dễ maintain/scale.
    - Liên hệ với principles/pattern đã nói (SOLID, CQRS, Clean Architecture,… nếu liên quan).  
- Ngôn ngữ ưu tiên (tuỳ chủ đề):
  - Python (FastAPI, data/AI pipelines).
  - SQL/NoSQL schema.
  - YAML/JSON (config, CI/CD, k8s manifest, logging config,…).

4.4. Yêu cầu checklist đánh giá thực tế

- Mỗi CHƯƠNG phải kết thúc bằng:
  - Checklist “Áp dụng vào dự án thật” gồm 10–20 câu hỏi dạng yes/no hoặc scale.
  - Checklist giúp:
    - Tech lead / architect review giải pháp trước go-live.
    - Developer tự audit code và thiết kế.  
- Checklist cần:
  - Cụ thể, tránh câu hỏi chung chung (“Hệ thống có đủ tốt không?”).
  - Tập trung vào risk thực tế (downtime, data loss, chi phí, maintainability,…).

4.5. Quality checklist & tự đánh giá
Khi hoàn thành TỪNG CHƯƠNG, phải tự đánh giá qua checklist sau (và thể hiện trong phần cuối chương):
- [ ] Nội dung chương MECE, không trùng lặp chương khác.
- [ ] Ít nhất 1 sơ đồ kiến trúc/flow có chú thích.
- [ ] Ít nhất 1 case study thực tế (có link đến source/case study).
- [ ] Ít nhất 1 anti-pattern + 1 best practice code snippet.
- [ ] Ít nhất 3–5 nguồn tham khảo chất lượng cao.
- [ ] Có checklist áp dụng thực tế cho chương đó.
- [ ] Thuật ngữ được sử dụng nhất quán với các chương khác.
- [ ] Nội dung đủ chi tiết để kỹ sư level mid–senior có thể áp dụng vào dự án thật.
      
      
5. Testing, kiểm thử: bạn nhớ làm tốt hơn các checklist được giao và ở mỗi subtasks hãy tự đánh giá lại chất lượng của output với vai trò là 1 người đọc và chuyên gia thẩm định tài liệu kĩ thuật    
```

```
1. You are Master Technical Writer and Master High Performance Python - Lập trình đồng bộ và bất đồng bộ. 
   
2. Your tasks, goals: Tạo tài liệu *All In One - Deep Dive - Comprehensive Lập trình đồng bộ và bất đồng bộ* (~100-150 trang) mapping chi tiết đi mục lục. 
   Goals: Cần đọc cái này để MASTER HIGH PERFORMANCE PYTHON VÀ ỨNG DỤNG ĐƯỢC TRONG THỰC TẾ
3. Instruction: 
- Step 1: Đọc kỹ toàn bộ các góc nhìn khác nhau (từ file đính kèm)
- Step 2: BÁM THEO HƯỚNG DẪN về cấu trúc thư mục ở bên dưới
  +, Ở mỗi phần đều deep research kĩ lưỡng + có link dẫn chứng đầy đủ các nguồn 
- Step 3: Viết song song tất cả các phần
  
4. OUTPUT REQUIREMENTS
### Format
- **File:** Single Markdown file (~100-150 pages), >= 50.000 từ , tiếng việt
- **Naming:** `All about Production Risk.md`
- **Encoding:** UTF-8
- **Line breaks:** Unix style (LF)
### Quality Checklist
4. OUTPUT REQUIREMENTS
### Format
- **File:** Single Markdown file (~100-150 pages), >= 50.000 từ , tiếng việt
- **Naming:** `<>.md`
- **Encoding:** UTF-8
- **Line breaks:** Unix style (LF)
### Quality Checklist
4.1. Yêu cầu nghiên cứu & trích dẫn nguồn

- Mỗi CHƯƠNG:
  - Tối thiểu 3–5 nguồn tham khảo chất lượng, ưu tiên:
    - Tài liệu chính thức (docs, RFC, standard).
    - Bài viết từ các công ty/nhà phát triển uy tín.
    - Case study từ hệ thống lớn / open-source có tiếng.
- Trong từng SECTION:
  - Khi đề cập tới best practice/anti-pattern, cần dẫn link đến:
    - Bài viết phân tích.
    - Hoặc mã nguồn minh họa (GitHub).
- Tài liệu phải nêu rõ:
  - Mô hình/kiến trúc được áp dụng ở đâu trong thực tế (nêu tên dự án/sản phẩm và link nếu có).
  - Các con số/claim (ví dụ: latency, throughput, cost) cần có nguồn hoặc ghi rõ “ví dụ minh họa”.

4.2. Yêu cầu về diagram

- Mỗi SECTION tối thiểu 1 diagram (dùng syntax phù hợp trong Markdown: mermaid/PlantUML hoặc ASCII schematic) giúp:
  - Mô tả kiến trúc.
  - Mô tả flow.
  - Mô tả quy trình.  
- Diagram phải:
  - Đơn giản, dễ đọc, có chú thích.
  - Có caption giải thích “key takeaway” chính.

4.3. Yêu cầu về code example

- Mỗi SECTION cần có:
  - 1 ví dụ anti-pattern:
    - Code ngắn gọn, thể hiện rõ sai lầm thường gặp.
    - Có comment giải thích tại sao sai, rủi ro trong production.
  - 1 ví dụ best practice:
    - Cùng ngữ cảnh nhưng được thiết kế lại đúng, sạch, dễ maintain/scale.
    - Liên hệ với principles/pattern đã nói (SOLID, CQRS, Clean Architecture,… nếu liên quan).  
- Ngôn ngữ ưu tiên (tuỳ chủ đề):
  - Python (FastAPI, data/AI pipelines).
  - SQL/NoSQL schema.
  - YAML/JSON (config, CI/CD, k8s manifest, logging config,…).

4.4. Yêu cầu checklist đánh giá thực tế

- Mỗi CHƯƠNG phải kết thúc bằng:
  - Checklist “Áp dụng vào dự án thật” gồm 10–20 câu hỏi dạng yes/no hoặc scale.
  - Checklist giúp:
    - Tech lead / architect review giải pháp trước go-live.
    - Developer tự audit code và thiết kế.  
- Checklist cần:
  - Cụ thể, tránh câu hỏi chung chung (“Hệ thống có đủ tốt không?”).
  - Tập trung vào risk thực tế (downtime, data loss, chi phí, maintainability,…).

4.5. Quality checklist & tự đánh giá
Khi hoàn thành TỪNG CHƯƠNG, phải tự đánh giá qua checklist sau (và thể hiện trong phần cuối chương):
- [ ] Nội dung chương MECE, không trùng lặp chương khác.
- [ ] Ít nhất 1 sơ đồ kiến trúc/flow có chú thích.
- [ ] Ít nhất 1 case study thực tế (có link đến source/case study).
- [ ] Ít nhất 1 anti-pattern + 1 best practice code snippet.
- [ ] Ít nhất 3–5 nguồn tham khảo chất lượng cao.
- [ ] Có checklist áp dụng thực tế cho chương đó.
- [ ] Thuật ngữ được sử dụng nhất quán với các chương khác.
- [ ] Nội dung đủ chi tiết để kỹ sư level mid–senior có thể áp dụng vào dự án thật.
      
      
5. Testing, kiểm thử: bạn nhớ làm tốt hơn các checklist được giao và ở mỗi subtasks hãy tự đánh giá lại chất lượng của output với vai trò là 1 người đọc và chuyên gia thẩm định tài liệu kĩ thuật 
```
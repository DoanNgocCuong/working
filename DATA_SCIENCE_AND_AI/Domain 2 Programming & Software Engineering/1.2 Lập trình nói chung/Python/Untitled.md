### Giải thích “lazy init” theo cách dễ hình dung

- Hãy tưởng tượng: bạn vào quán cà phê.

- Eager init (khởi tạo sớm): Quán pha sẵn tất cả đồ uống dù chưa có khách gọi → tốn công, tốn nguyên liệu.
	- Nếu vẫn “eager init” nặng ngay lúc import, bạn gặp:
	
	- Startup chậm, block toàn bộ API.
	
	- Khởi tạo Redis/MySQL/LLM khi chưa cần dùng.
	
	- Dễ lỗi khi env chưa sẵn (VD: settings, secrets), test/unit test khó.


- Lazy init (khởi tạo lười): Chỉ pha khi có khách gọi món → tiết kiệm, nhanh mở quán.
	 “Lazy init” giúp: chỉ khởi tạo khi endpoint v2 bot thực sự được gọi, giữ API nhanh nhẹ và ổn định.

---

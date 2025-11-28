
- 10 worker process (từ replicas: 1 → replicas: 10)

- Mỗi worker xử lý 10 message song song (từ prefetch_count=1 → prefetch_count=10)

- Tổng concurrency: 10 workers × 10 messages = 100 messages cùng lúc

---

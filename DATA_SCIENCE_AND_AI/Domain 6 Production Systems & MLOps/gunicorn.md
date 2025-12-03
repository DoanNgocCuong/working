
# 1. Cách run python thuần với Unicorn và Gunicorn

[4 tools called]

Bạn đang khởi tạo FastAPI app tại `app.server:app`, nên chỉ cần truyền app đó cho Gunicorn rồi tăng số lượng worker bằng tham số `-w` (hoặc biến môi trường `WEB_CONCURRENCY`). Ví dụ chạy trực tiếp:

```bash
WEB_CONCURRENCY=4 \
gunicorn app.server:app \
  -w 4 \
  -k uvicorn.workers.UvicornWorker \
  -b 0.0.0.0:8000 \
  --timeout 120 \
  --log-level info
```

- `-w 4` (hoặc `WEB_CONCURRENCY=4`) đặt 4 worker process.
- `-k uvicorn.workers.UvicornWorker` để Gunicorn dùng event loop của Uvicorn cho FastAPI.
- Điều chỉnh `--timeout`, `--log-level`, `-b` theo nhu cầu.

Nếu muốn cấu hình cố định, bạn có thể tạo `gunicorn.conf.py` (hoặc `.py` tương tự) rồi để Gunicorn tự tải:

```python
# gunicorn.conf.py
bind = "0.0.0.0:8000"
workers = 6              # số worker mong muốn
worker_class = "uvicorn.workers.UvicornWorker"
timeout = 120
graceful_timeout = 30
loglevel = "info"
```

Khi chạy chỉ cần:

```bash
gunicorn app.server:app -c gunicorn.conf.py
```

Nhớ đảm bảo entry point thật sự là `app` như cuối file server:

```338:341:app/server.py
app = create_app()
```

Thay đổi số worker để phù hợp tài nguyên CPU/RAM của máy triển khai; thường đặt `workers = 2 * CPU + 1` rồi đo thực tế để tinh chỉnh.


---
# 2. Cách chạy docker - docker compos với 


Mình sẽ xem cấu hình trong thư mục `deploy` (Dockerfile, docker-compose, script `.sh`) để xem bạn đang chạy server bằng gì, rồi chỉ bạn chỗ chỉnh số lượng worker cho đúng chỗ.

[3 tools called]

Đúng rồi, với thư mục `deploy` hiện tại bạn **đang chạy trực tiếp bằng Uvicorn**, không dùng Gunicorn:

```18:18:deploy/Dockerfile.api
CMD ["python", "-m", "uvicorn", "app.server:app", "--host", "0.0.0.0", "--port", "30000"]
```

Muốn tăng số lượng worker, bạn có hai hướng:

### 1. Dùng luôn worker của Uvicorn (dễ nhất)

Sửa `CMD` trong `Dockerfile.api` thành:

```dockerfile
CMD ["python", "-m", "uvicorn", "app.server:app", "--host", "0.0.0.0", "--port", "30000", "--workers", "4"]
```

- Thay `4` bằng số worker bạn muốn.
- Sau đó `docker compose -f deploy/docker-compose.prod.yaml up --build` để build lại.

### 2. Chuyển sang Gunicorn + UvicornWorker (nếu muốn production kiểu “chuẩn” hơn)

Thay `CMD` trong `Dockerfile.api` thành:

```dockerfile
CMD ["gunicorn", "app.server:app", "-k", "uvicorn.workers.UvicornWorker", "-w", "4", "-b", "0.0.0.0:30000"]
```

- Ở đây `-w 4` là số worker.
- Nhớ đảm bảo trong image đã `pip install gunicorn uvicorn`.

Nếu bạn gửi nội dung `deploy/docker-compose.worker.yaml` mình có thể gợi ý thêm là nên sync số worker HTTP và số worker background như thế nào cho hợp lý.
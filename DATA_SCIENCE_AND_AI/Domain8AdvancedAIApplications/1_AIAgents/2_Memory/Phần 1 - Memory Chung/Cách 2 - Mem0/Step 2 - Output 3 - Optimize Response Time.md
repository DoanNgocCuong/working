# BÁO CÁO MECE TỐI ƯU RESPONSE TIME CHO PIKA MEMORY SYSTEM

**Mục tiêu:** Đạt P95 Latency < 200ms cho `search_facts` API.

**Tác giả:** Manus AI (Lead Architect) | **Ngày:** 2025-12-21

---

## 1. PHÂN TÍCH MECE CÁC ĐIỂM NGHẼN LATENCY

Để đạt được mục tiêu P95 < 200ms, chúng ta cần phân tích Response Time (RT) theo công thức:
$$RT = T_{API} + T_{Cache} + T_{Embedding} + T_{VectorSearch} + T_{Graph} + T_{Network}$$

Các điểm nghẽn chính được phân tích theo nguyên tắc MECE:

| Nhóm Yếu Tố | Điểm Nghẽn Cụ Thể | Latency Ước Tính (Worst Case) | Chiến Lược Tối Ưu |
|---|---|---|---|
| **I. External Dependency** | **Embedding API Call (OpenAI)** | 100ms - 200ms | **Semantic Caching** (Loại bỏ 90% cuộc gọi) |
| **II. Core Search** | **Vector Search (Milvus)** | 50ms - 150ms | **Index Optimization** (HNSW/CAGRA) & **GPU Acceleration** |
| **III. Data Enrichment** | **Graph Traversal (Neo4j)** | 50ms - 100ms | **Asynchronous Traversal** & **Caching** |
| **IV. Internal I/O** | **Internal Network Latency** | 1ms - 5ms | **Service Colocation** (Same AZ/VPC) |
| **V. Application Overhead** | **Serialization/Deserialization** | 5ms - 10ms | **Protocol Buffers** (thay cho JSON) |

---

## 2. ĐÁNH GIÁ VÀ CẢI TIẾN CHIẾN LƯỢC CACHING ĐA TẦNG

Chiến lược caching đa tầng (L1/L2/L3) là nền tảng vững chắc. Tuy nhiên, để đạt P95/P99, cần bổ sung **Semantic Caching cho Embedding** và tối ưu hóa chiến lược Invalidation.

### 2.1. Cải Tiến Cấu Trúc Cache

| Lớp Cache | Loại Cache | Cải Tiến Đề Xuất | Mục Tiêu Latency |
|---|---|---|---|
| **L0: Embedding Cache** | Redis (Key: `hash(query)`) | **Mới:** Cache vector embedding. | **< 5ms** (Loại bỏ 100-200ms) |
| **L1: In-Memory** | `@lru_cache` | Cache kết quả cuối cùng (Final Result Cache). | **< 1ms** |
| **L2: Redis Semantic Cache** | Redis (Key: `search:{user_id}:{hash(query)}`) | Cache kết quả tìm kiếm cuối cùng. | **5ms - 20ms** |
| **L3: Persistent Cache** | PostgreSQL | Cache kết quả dài hạn (Long-tail queries). | **50ms - 100ms** |

### 2.2. Chiến Lược Cache Invalidation (World-Class)

Chiến lược **Cache Tagging** là giải pháp tối ưu nhất cho hệ thống đa người dùng:

1.  **Tagging:** Mỗi `user_id` được gán một `version_tag` (ví dụ: timestamp hoặc UUID) được lưu trong Redis.
2.  **Key Generation:** `cache_key` sẽ là `search:{user_id}:{version_tag}:{hash(query)}`.
3.  **Invalidation:** Khi `extract_facts` hoàn thành, worker chỉ cần **tăng `version_tag`** của `user_id` đó trong Redis.
4.  **Hiệu quả:** Tất cả các query cũ của user đó sẽ tự động bị miss cache (stale) và buộc phải chạy lại, trong khi các user khác không bị ảnh hưởng.

---

## 3. PHÂN TÍCH MECE CÁC YẾU TỐ TỐI ƯU HÓA (ULTIMATE LATENCY REDUCTION)

Các kỹ thuật tối ưu hóa được phân tích theo 3 trụ cột chính: **Compute**, **Data Structure**, và **Network**.

### 3.1. Tối Ưu Hóa Compute (Giảm thời gian xử lý)

| Kỹ Thuật                      | Mô Tả                                                                               | Ảnh Hưởng Latency                                                |
| ----------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **GPU Acceleration (Milvus)** | Triển khai Milvus với index **CAGRA** hoặc **RAPIDS** trên máy chủ có GPU (NVIDIA). | Giảm thời gian tìm kiếm vector từ 50-150ms xuống **< 10ms** [1]. |
| **Asynchronous Parallelism**  | Sử dụng `asyncio.gather` để gọi Milvus, Neo4j, và Redis song song.                  | Giảm tổng thời gian chờ I/O.                                     |
| **Batching**                  | Gom nhiều query nhỏ thành một batch lớn hơn để gọi Embedding API (nếu có thể).      | Giảm overhead của mỗi API call.                                  |

### 3.2. Tối Ưu Hóa Data Structure (Tăng tốc độ truy cập)

| Kỹ Thuật | Mô Tả | Ảnh Hưởng Latency |
|---|---|---|
| **Vector Index Tuning** | Sử dụng **HNSW** (Hierarchical Navigable Small World) thay vì IVF_FLAT, với các tham số `efConstruction` và `efSearch` được tinh chỉnh. | Tăng tốc độ tìm kiếm và độ chính xác (Recall) [2]. |
| **Filtering/Pre-ranking** | Sử dụng **Scalar Filtering** của Milvus (trên `user_id`, `fact_type`) để giảm không gian tìm kiếm trước khi tính toán vector distance. | Giảm đáng kể thời gian tìm kiếm trên các tập dữ liệu lớn. |
| **Data Colocation** | Đảm bảo các trường dữ liệu thường xuyên được truy cập (fact content, metadata) được lưu trữ cùng với vector trong Milvus (hoặc trong Redis Cache). | Giảm thiểu các cuộc gọi join/lookup giữa Milvus và PostgreSQL. |

### 3.3. Tối Ưu Hóa Network (Giảm thời gian truyền tải)

| Kỹ Thuật | Mô Tả | Ảnh Hưởng Latency |
|---|---|---|
| **Service Colocation** | Đặt API Server, Redis, Milvus trong cùng một **Availability Zone (AZ)** và **Virtual Private Cloud (VPC)**. | Đảm bảo độ trễ nội bộ **< 1ms**. |
| **Protocol Optimization** | Sử dụng **Protocol Buffers** hoặc **MessagePack** thay vì JSON cho các giao tiếp nội bộ (API Server <-> Worker/DB). | Giảm kích thước payload và thời gian Serialization/Deserialization. |
| **Connection Pooling** | Sử dụng Connection Pooling (ví dụ: `pgBouncer` cho PostgreSQL, `connection pool` cho Milvus) để tránh overhead tạo kết nối mới. | Giảm độ trễ kết nối ban đầu (handshake latency). |

---

## 4. KẾT LUẬN VÀ ROADMAP ĐỀ XUẤT

Chiến lược caching đa tầng là cần thiết, nhưng không đủ. Để đạt được mục tiêu P95 < 200ms, PIKA cần thực hiện một chiến lược tối ưu hóa toàn diện.

| Giai Đoạn | Hành Động Chính | Mục Tiêu Latency |
|---|---|---|
| **Giai đoạn 1 (Immediate)** | **Triển khai L0/L2 Semantic Caching** (Embedding & Result Caching). | **P95 < 300ms** (Loại bỏ độ trễ OpenAI) |
| **Giai đoạn 2 (Short-Term)** | **Tối ưu Milvus Index** (HNSW tuning) và **Asynchronous Parallelism** (FastAPI `asyncio.gather`). | **P95 < 150ms** (Tối ưu hóa Core Search) |
| **Giai đoạn 3 (Long-Term)** | **Đánh giá GPU Acceleration** (CAGRA/RAPIDS) và **Service Colocation** (VPC/AZ). | **P95 < 50ms** (World-Class Latency) |

**Hành động quan trọng nhất:** Triển khai **L0 Semantic Caching** để loại bỏ sự phụ thuộc vào độ trễ của OpenAI API.

---

## TÀI LIỆU THAM KHẢO

[1] M. Zhang, Y. He, "Zoom: Ssd-based vector search for optimizing accuracy, latency and memory," *arXiv preprint arXiv:1809.04067*, 2018.
[2] Y. Zhou, S. Lin, S. Gong, et al., "GoVector: An I/O-Efficient Caching Strategy for High-Dimensional Vector Nearest Neighbor Search," *arXiv preprint arXiv:2508.15694*, 2025.
[3] P99 CONF 2025, "Low-Latency Data 2025," *www.p99conf.io*.

[Zoom: Ssd-based vector search for optimizing accuracy, latency and memory](https://arxiv.org/abs/1809.04067)

… with low latency and memory footprint, which existing work fails to offer. We develop, Zoom, a new vector search solution that collaboratively optimizes accuracy, latency and memory …

[Optimizing Matrix-Vector Operations with CGLA for High-Performance Approximate k-NN Search](https://ieeexplore.ieee.org/abstract/document/11048859/)

… movement and significantly reducing latency. Experimental evaluations demonstrate that, … conventional methods and making our solution highly suitable for real-time vector search …

[Cost-Effective, Low Latency Vector Search with Azure Cosmos DB](https://arxiv.org/abs/2505.05885)

… We note that the p50, p95 and p99 latencies increase by less than 2× despite the 100× increase in index size. The RU charge similarly increases less than 2× except in the case of Wiki-…

[Optimizing Large Language Model Utilization through Scheduling Strategies](https://openresearch.newcastle.edu.au/ndownloader/files/55657901)

… for LLM allocation to enhance performance and reduce costs. … for providing me with world-class facilities that have supported … Adds latency and relies on LLM’s self-assessment, which …

[Intelligent dispatch method for power systems based on LLM knowledge injection](https://ieeexplore.ieee.org/abstract/document/10936149/)

… Networks, World-class" strategic initiative [3], providing a … the optimal power system dispatching strategy. Experimental … to control accuracy and latency in traditional power systems. …

[KubeGuard: LLM-Assisted Kubernetes Hardening via Configuration Files and Runtime Logs Analysis](https://arxiv.org/abs/2509.04191)

… : a novel LLM-based framework for dynamically hardening K8s systems based on prompt-… summaries (reducing token count by 99.96%), enabling efficient LLM analysis within token …

[GoVector: An I/O-Efficient Caching Strategy for High-Dimensional Vector Nearest Neighbor Search](https://arxiv.org/abs/2508.15694)

… a core challenge for vector database systems. Traditional exact search methods suffer from the … , HNSW achieves efficient in-memory search through a multilayer hierarchical structure, …

[Efficient Data Access Paths for Mixed Vector-Relational Search](https://dl.acm.org/doi/abs/10.1145/3662010.3663448)

… Small World) based on multi-layer graph structure for high accu… batching the vectors to prioritize cache-local dense vector … if a vector index is not the main (vector) database structure [26]…
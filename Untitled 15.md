1. Hiện tại: stress test 100 CCU dùng = 100% CPU, use 2.5/4GB => Giảm 4 cores, 6GB => 1 - 2 cores và 4G là ngon.  

2. Hiện tại: Embedding tiêu thụ GPU thấp chỉ khoảng 10% => Tăng batch size tu 64 -> lên 128 và 256

3. I/O-bound hoặc GPU-bound workload (embedding): OMP_NUM_THREADS = CPU cores hoặc ít hơn => Do cores giảm về 2 => OMP_NUM_THREADS từ 4 -> 2
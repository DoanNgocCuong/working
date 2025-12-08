
Rất sẵn lòng! Đây là bảng "bí kíp" các chỉ số cấu hình vLLM, được giải thích siêu đơn giản như đang hướng dẫn xây dựng một nhà máy LEGO.

# 1. 🏭 Bảng "Bí Kíp" Cấu Hình Nhà Máy AI (vLLM)

Tưởng tượng GPU của bạn là một **Nhà máy lắp ráp LEGO (AI)**, và các chỉ số cấu hình là bảng điều khiển của Giám đốc nhà máy (bạn).

| Tên Chỉ Số (Config Name)       | Ý Nghĩa (Giải thích cho học sinh)                                                                                                | Ví dụ Thực tế                                                                                                                                   | Giá trị nên đặt (Cho Pika)                                             | Có ảnh hưởng đến response time như nào ? |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- | ---------------------------------------- |
| **`--gpu-memory-utilization`** | **Diện tích Kho chứa hàng**  <br>Bạn dành bao nhiêu phần trăm sân nhà máy để chứa nguyên liệu (KV Cache).                        | Nếu nhà máy rộng 100m², bạn dành 60m² làm kho thì chỉ số là 0.6.  <br>_Kho càng rộng → Chứa được càng nhiều đơn hàng cùng lúc._                 | `0.3` - `0.4`  <br>_(Model nhỏ nên kho không cần quá to, vừa đủ dùng)_ |                                          |
| **`--max-model-len`**          | **Độ dài băng chuyền tối đa**  <br>Chiều dài tối đa của một sản phẩm (câu hỏi + câu trả lời) mà dây chuyền có thể xử lý.         | Nếu băng chuyền dài 2 mét, bạn không thể lắp con rồng dài 3 mét. Nó sẽ bị cắt cụt đuôi.  <br>_Dài quá → Tốn chỗ. Ngắn quá → Không đủ viết văn._ | `512` - `1024`  <br>_(Pika nói ngắn gọn, không viết tiểu thuyết)_      |                                          |
| **`--max-num-seqs`**           | **Số làn chạy song song**  <br>Nhà máy có thể phục vụ tối đa bao nhiêu khách hàng cùng một lúc.                                  | Giống như quầy thu ngân siêu thị. Chỉ số này là số lượng quầy mở cửa.  <br>_Càng nhiều → Phục vụ càng đông, nhưng cần kho (VRAM) to._           | `256` - `512`  <br>_(Model nhỏ nên mở nhiều làn thoải mái)_            |                                          |
| **`--enable-prefix-caching`**  | **Chế độ "Copy bài mẫu"**  <br>Ghi nhớ phần mở đầu giống nhau (ví dụ: luật chơi) để không phải đọc lại từ đầu với mỗi khách mới. | Giống như giáo viên chép đề bài lên bảng. Học sinh chỉ cần chép lời giải, không phải chép lại đề.  <br>_Tiết kiệm rất nhiều thời gian!_         | `True` (Bật)  <br>_(Rất quan trọng vì Pika dùng chung system prompt)_  |                                          |
| **`--dtype`**                  | **Độ tinh xảo của gạch LEGO**  <br>Kích thước và độ chính xác của từng mảnh dữ liệu.                                             | - `float16`: Gạch tiêu chuẩn (nhẹ, nhanh).  <br>- `float32`: Gạch cao cấp (nặng gấp đôi, chính xác hơn xíu).                                    | `float16` hoặc `auto`  <br>_(Nhanh và nhẹ là ưu tiên)_                 |                                          |
| **`--enforce-eager`**          | **Chế độ "Cầm tay chỉ việc"**  <br>CPU ra lệnh từng bước nhỏ cho GPU làm. (Tắt chế độ này = Bật CUDA Graphs).                    | - Bật (`True`): Sếp đứng kè kè chỉ từng bước (chậm).  <br>- Tắt (`False`): Sếp đưa cả bản vẽ, thợ tự làm một mạch (nhanh).                      | `False` (Tắt)  <br>_(Để kích hoạt CUDA Graphs siêu tốc)_               |                                          |
| **`--max-num-batched-tokens`** | **Sức chứa tối đa của xe đẩy**  <br>Tổng số mảnh LEGO tối đa mà nhà máy có thể xử lý trong một nhịp.                             | Xe đẩy chỉ chở được 2000 mảnh. Dù có 100 đơn hàng nhỏ hay 1 đơn hàng to, tổng lại không được quá số này.                                        | `2048`  <br>_(Đủ sức cân cả trăm câu trả lời ngắn)_                    |                                          |

|                                |                                                                                                                                               |                                                                                                                                                                |                                                                |     |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- | --- |
| **`--max-num-batched-tokens`** | **Sức tải của xe nâng hàng**  <br>Tổng số lượng mảnh ghép tối đa mà một lần xe nâng có thể bốc lên để đưa vào máy làm.                        | Xe nâng chỉ chở được 2048 mảnh. Dù là 1 con rồng to (2000 mảnh) hay 10 con gà nhỏ (200 mảnh/con), tổng cộng không được quá tải trọng xe.                       | `2048` - `4096`  <br>_(Đủ lớn để xử lý nhiều yêu cầu gộp lại)_ |     |
| **`--kv-cache-dtype`**         | **Chất liệu khay đựng nguyên liệu**  <br>Bạn chọn loại khay nhựa thường (fp16) hay khay nhựa nén (fp8) để đựng các khối nhớ tạm thời.         | - `auto`: Máy tự chọn loại khay tốt nhất.  <br>- `fp8`: Khay mỏng hơn, chứa được nhiều gấp đôi nhưng hơi khó xếp.  <br>_Giúp tiết kiệm diện tích kho (VRAM)._  | `auto`  <br>_(Để vLLM tự quyết định cái nào an toàn nhất)_     |     |
| **`--disable-log-requests`**   | **Tắt loa thông báo từng đơn hàng**  <br>Bình thường mỗi khi có khách gọi món, loa sẽ "Alo! Có đơn mới!". Tắt đi để đỡ ồn và đỡ tốn điện.     | Nếu nhà máy quá đông khách, loa kêu liên tục sẽ làm điếc tai và chậm tiến độ. Tắt đi, chỉ làm thôi, không nói nhiều.  <br>_Giúp máy chạy nhanh hơn xíu xiu._   | `True` (Có cờ này)  <br>_(Tắt log để tập trung tốc độ)_        |     |
| **`--trust-remote-code`**      | **Chìa khóa vạn năng**  <br>Cho phép nhà máy mở các bản vẽ thiết kế lạ từ bên ngoài (HuggingFace) mà không cần kiểm duyệt gắt gao.            | Bạn tải bản vẽ mod của một pháp sư trên mạng về. Máy hỏi: "Tin ông này không?". Bạn bảo "Tin!" thì máy mới chịu chạy.  <br>_Bắt buộc với một số model mới/lạ._ | `True` (Có cờ này)  <br>_(Cần thiết cho SmolLM2/Phi-3)_        |     |
| **`--host 0.0.0.0`**           | **Mở cửa chính và cửa sổ**  <br>Cho phép nhận đơn đặt hàng từ bất kỳ ai, ở bất kỳ đâu (trong mạng LAN/Internet), không chỉ từ nội bộ nhà máy. | - `127.0.0.1`: Chỉ người trong nhà mới được đặt hàng.  <br>- `0.0.0.0`: Hàng xóm, người đi đường đều có thể ghé vào đặt (nếu không có bảo vệ chặn).            | `0.0.0.0`  <br>_(Để robot Pika từ máy khác gọi được)_          |     |

---
# 2. Báo Cáo Chuyên Sâu: Các Kỹ Thuật Tối Ưu Hóa Hiệu Suất Serving cho Mô Hình Ngôn Ngữ Lớn (LLM)

**Tác giả**: Manus AI
**Ngày**: 08 tháng 12 năm 2025

## Giới thiệu

Việc triển khai và phục vụ (serving) các Mô Hình Ngôn Ngữ Lớn (LLM) trong môi trường sản xuất đặt ra những thách thức đáng kể về hiệu suất và chi phí. Khi các mô hình ngày càng trở nên phức tạp, việc đảm bảo độ trễ (latency) thấp và thông lượng (throughput) cao trở thành yếu tố then chốt để mang lại trải nghiệm người dùng tốt và tối ưu hóa chi phí vận hành. Báo cáo này sẽ đi sâu vào các kỹ thuật tối ưu hóa serving hiện đại, phân tích các chỉ số hiệu suất quan trọng, và cung cấp một quy trình có hệ thống để đạt được hiệu suất phục vụ tối ưu, dựa trên các tài liệu nghiên cứu và ví dụ thực tiễn.

## 1. Các Chỉ Số Hiệu Suất Then Chốt trong LLM Serving

Để đánh giá và tối ưu hóa hiệu suất của một hệ thống phục vụ LLM, cần phải hiểu rõ các chỉ số đo lường chính. Các chỉ số này không chỉ phản ánh tốc độ của hệ thống mà còn cho thấy khả năng xử lý đồng thời nhiều yêu cầu và hiệu quả sử dụng tài nguyên phần cứng.

| Chỉ Số | Tên Tiếng Anh | Mô Tả | Tầm Quan Trọng | 
| :--- | :--- | :--- | :--- | 
| **Độ trễ token đầu tiên** | Time To First Token (TTFT) | Thời gian từ khi người dùng gửi yêu cầu cho đến khi nhận được token đầu tiên của câu trả lời. | Phản ánh khả năng phản hồi tức thì của hệ thống, rất quan trọng cho các ứng dụng tương tác thời gian thực như chatbot. | 
| **Độ trễ giữa các token** | Inter-Token Latency (ITL) | Thời gian trung bình để sinh ra mỗi token tiếp theo sau token đầu tiên. | Ảnh hưởng đến tốc độ đọc và cảm nhận về sự "trôi chảy" của câu trả lời. | 
| **Thông lượng** | Throughput | Số lượng token hoặc yêu cầu được hệ thống xử lý trong một đơn vị thời gian (thường là tokens/giây hoặc requests/giây). | Đo lường khả năng xử lý tải của hệ thống, quyết định khả năng mở rộng và chi phí trên mỗi yêu cầu. | 
| **Sử dụng bộ nhớ GPU** | GPU Memory Utilization | Tỷ lệ phần trăm bộ nhớ GPU được cấp phát và sử dụng cho việc lưu trữ trọng số mô hình và KV Cache. | Tối ưu hóa chỉ số này giúp tăng kích thước batch, từ đó tăng thông lượng mà không gây ra lỗi hết bộ nhớ (Out-of-Memory). | 

Việc tối ưu hóa thường là một sự đánh đổi giữa các chỉ số này. Ví dụ, việc tăng kích thước batch (batch size) để cải thiện thông lượng thường sẽ làm tăng độ trễ của từng yêu cầu riêng lẻ. Do đó, mục tiêu là tìm ra điểm cân bằng tối ưu phù hợp với yêu cầu cụ thể của từng ứng dụng.

## 2. Các Kỹ Thuật Tối Ưu Hóa Cốt Lõi

Quá trình tối ưu hóa serving bao gồm nhiều lớp kỹ thuật, từ nén mô hình ở cấp độ cao đến tối ưu hóa kernel ở cấp độ thấp. 

### 2.1. Lượng Tử Hóa (Quantization)

Lượng tử hóa là quá trình giảm độ chính xác số học của các trọng số mô hình, thường từ kiểu dữ liệu 32-bit (FP32) xuống 16-bit (FP16/BF16) hoặc thậm chí 8-bit và 4-bit. Kỹ thuật này giúp giảm đáng kể dung lượng lưu trữ của mô hình và yêu cầu băng thông bộ nhớ, từ đó tăng tốc độ suy luận [1].

> **Ví dụ thực tiễn**: Trong tài liệu được cung cấp, mô hình `SmolLM2-135M` sử dụng `dtype float16`, một dạng lượng tử hóa, giúp mô hình chỉ chiếm khoảng 300MB VRAM, cho phép nó chạy trên các GPU có bộ nhớ hạn chế với độ trễ rất thấp.

Các phương pháp lượng tử hóa phổ biến bao gồm:
- **AWQ (Activation-aware Weight Quantization)**: Bảo vệ một phần nhỏ các trọng số quan trọng khỏi việc lượng tử hóa để duy trì độ chính xác của mô hình.
- **GPTQ (Post-Training Quantization for GPT)**: Một phương pháp lượng tử hóa sau huấn luyện, cố gắng giảm thiểu sai số lượng tử hóa.
- **GGUF (GPT-Generated Unified Format)**: Một định dạng file được thiết kế để tải và chạy các mô hình đã được lượng tử hóa một cách hiệu quả trên CPU và GPU.

### 2.2. Cơ Chế Caching: Tối Ưu Hóa KV Cache và Prefix Caching

Trong kiến trúc Transformer, cơ chế attention yêu cầu tính toán các ma trận Key (K) và Value (V) cho mỗi token. **KV Cache** là một kỹ thuật nền tảng, lưu trữ lại các giá trị K và V đã được tính toán của các token trong chuỗi đầu vào để tái sử dụng khi sinh ra các token tiếp theo, tránh việc tính toán lại từ đầu [2].

**Prefix Caching** (hay Prompt Caching) là một bước tiến xa hơn của KV Cache. Nó lưu trữ và tái sử dụng KV cache của một tiền tố (prefix) chung qua nhiều yêu cầu khác nhau. Kỹ thuật này cực kỳ hiệu quả trong các ứng dụng có cấu trúc prompt lặp lại, chẳng hạn như chatbot với system prompt cố định hoặc các ứng dụng RAG (Retrieval-Augmented Generation).

> Theo tài liệu từ BentoML, Prefix Caching có thể giảm chi phí tính toán tới 90% và độ trễ tới 85% cho các yêu cầu có prompt dài và lặp lại [3]. Để tối đa hóa hiệu quả, các phần tĩnh của prompt nên được đặt ở đầu, trong khi các phần động (dữ liệu người dùng) nên được đặt ở cuối.

### 2.3. Batching và Scheduling: Continuous Batching và Chunked Prefill

**Continuous Batching** là một cải tiến so với static batching, cho phép thêm các yêu cầu mới vào batch đang xử lý ngay khi có tài nguyên GPU trống, thay vì phải chờ toàn bộ batch hoàn thành. Điều này giúp tăng đáng kể GPU utilization và thông lượng.

Tuy nhiên, quá trình suy luận LLM gồm hai giai đoạn với đặc tính khác nhau: **prefill** (xử lý prompt đầu vào, tính toán song song và sử dụng nhiều tài nguyên) và **decode** (sinh ra từng token một, bị giới hạn bởi băng thông bộ nhớ). Sự khác biệt này gây ra tình trạng "bong bóng" trong pipeline, làm giảm hiệu quả. **Chunked Prefill** giải quyết vấn đề này bằng cách chia giai đoạn prefill dài thành các "chunk" nhỏ hơn. Các chunk này sau đó được xếp lịch (schedule) và xử lý xen kẽ với các yêu cầu decode, giúp lấp đầy các khoảng trống tài nguyên và cải thiện đáng kể thông lượng tổng thể [4].

### 2.4. Các Chiến Lược Song Song Hóa (Parallelism)

Để chạy các mô hình có hàng chục hoặc hàng trăm tỷ tham số, việc sử dụng nhiều GPU là bắt buộc. **Tensor Parallelism (TP)** là một kỹ thuật phổ biến, chia các phép nhân ma trận lớn trong mô hình ra nhiều GPU. Mỗi GPU chỉ giữ một phần của trọng số và thực hiện một phần của phép tính, sau đó kết quả được tổng hợp lại. Mặc dù kỹ thuật này cho phép chạy các mô hình khổng lồ, nó cũng làm tăng chi phí giao tiếp (communication overhead) giữa các GPU [5]. Việc lựa chọn `tensor-parallel-size` phù hợp là một sự đánh đổi giữa dung lượng bộ nhớ trên mỗi GPU và độ trễ do giao tiếp.

### 2.5. Các Chiến Lược Decode Nâng Cao: Speculative Decoding

**Speculative Decoding** là một kỹ thuật đột phá giúp giảm độ trễ bằng cách giảm số lượng forward pass cần thiết để sinh ra một chuỗi token. Ý tưởng là sử dụng một mô hình "nháp" (draft model) nhỏ và nhanh để sinh ra một chuỗi token ứng cử viên. Sau đó, mô hình "chính" (target model) lớn và chính xác sẽ xác thực tất cả các token ứng cử viên này trong một forward pass duy nhất. Các token được xác thực sẽ được chấp nhận, và quá trình tiếp tục từ token cuối cùng được chấp nhận. Các phương pháp như EAGLE-3 đã cải tiến kỹ thuật này bằng cách tích hợp cơ chế nháp vào chính mô hình chính, loại bỏ nhu cầu về một mô hình riêng biệt [6].

### 2.6. Tối Ưu Hóa Cấp Thấp: FlashAttention và CUDA Graphs

- **FlashAttention**: Là một thuật toán attention được tối ưu hóa ở cấp độ kernel, giúp giảm đáng kể việc đọc/ghi vào bộ nhớ GPU, từ đó tăng tốc độ và giảm bộ nhớ cần thiết cho quá trình attention [7]. Hầu hết các framework serving hiện đại như vLLM đều tích hợp sẵn FlashAttention.
- **CUDA Graphs**: Cho phép ghi lại một chuỗi các hoạt động trên GPU và phát lại chúng mà không cần sự can thiệp của CPU. Điều này loại bỏ overhead từ việc gọi kernel lặp đi lặp lại, đặc biệt hiệu quả trong giai đoạn decode khi cùng một kernel được thực thi nhiều lần [8].

## 3. Triển Khai Thực Tế với vLLM

vLLM là một trong những framework serving LLM phổ biến nhất hiện nay, tích hợp nhiều kỹ thuật tối ưu hóa đã đề cập. Việc tinh chỉnh các tham số của vLLM là rất quan trọng để đạt được hiệu suất tối ưu.

| Tham Số vLLM | Chức Năng và Tác Động | 
| :--- | :--- | 
| `--gpu-memory-utilization` | Thiết lập tỷ lệ bộ nhớ GPU dành cho KV Cache. Tăng giá trị này cho phép xử lý batch lớn hơn, nhưng có thể gây lỗi OOM. | 
| `--max-num-seqs` | Số lượng yêu cầu tối đa trong một batch. Tăng giá trị này giúp tăng thông lượng nhưng cũng làm tăng độ trễ. | 
| `--max-model-len` | Độ dài chuỗi tối đa (input + output). Ảnh hưởng trực tiếp đến yêu cầu bộ nhớ cho KV Cache. | 
| `--enable-prefix-caching` | Kích hoạt tính năng Prefix Caching để tăng tốc các yêu cầu có tiền tố chung. | 
| `--enable-chunked-prefill` | Kích hoạt Chunked Prefill để cải thiện thông lượng, đặc biệt với các prompt dài. | 
| `--tensor-parallel-size` | Số lượng GPU được sử dụng cho Tensor Parallelism. | 
| `--quantization` | Chỉ định phương pháp lượng tử hóa (ví dụ: `awq`, `gptq`) để chạy các mô hình đã được nén. | 

**Phân tích ví dụ `SmolLM2-135M`**: Cấu hình được cung cấp trong tài liệu (`gpu-memory-utilization=0.3`, `max-num-seqs=512`) cho thấy một chiến lược tối ưu hóa cho mô hình rất nhỏ. Vì mô hình nhẹ, phần lớn bộ nhớ GPU có thể được dành cho KV Cache, cho phép xử lý một batch rất lớn (`512` sequences) để tối đa hóa thông lượng, phù hợp cho các tác vụ phân loại cảm xúc có thể xử lý đồng thời.

## 4. Quy Trình Tối Ưu Hóa Hệ Thống

Một quy trình tối ưu hóa có hệ thống bao gồm các bước sau:

1.  **Phân tích Yêu cầu**: Xác định rõ các mục tiêu về độ trễ (TTFT, ITL) và thông lượng (requests/giây) cho ứng dụng cụ thể.
2.  **Phân tích Workload**: Nghiên cứu đặc điểm của các yêu cầu (độ dài input/output trung bình và tối đa, tỷ lệ lặp lại của prompt).
3.  **Lựa chọn Phần cứng và Mô hình**: Dựa trên kích thước mô hình và yêu cầu hiệu suất, chọn GPU có đủ bộ nhớ (VRAM) và khả năng tính toán. Cân nhắc sử dụng các phiên bản mô hình đã được lượng tử hóa.
4.  **Cấu hình Ban đầu**: Bắt đầu với một cấu hình cơ bản, an toàn (ví dụ: `gpu-memory-utilization=0.8`, `max-num-seqs` ở mức vừa phải).
5.  **Benchmarking**: Sử dụng các công cụ benchmark để đo lường hiệu suất của hệ thống dưới các mức tải khác nhau, ghi lại các chỉ số chính.
6.  **Tinh chỉnh và Lặp lại**: Dựa trên kết quả benchmark, tinh chỉnh các tham số (ví dụ: tăng `max-num-seqs` nếu GPU utilization còn thấp, giảm nếu độ trễ quá cao) và thực hiện lại benchmark cho đến khi đạt được mục tiêu.

## 5. Kết Luận và Xu Hướng Tương Lai

Tối ưu hóa serving cho LLM là một lĩnh vực phức tạp nhưng cực kỳ quan trọng, đòi hỏi sự hiểu biết sâu sắc về cả mô hình, phần cứng và các thuật toán hệ thống. Bằng cách áp dụng một cách có hệ thống các kỹ thuật như lượng tử hóa, prefix caching, chunked prefill, và speculative decoding, các tổ chức có thể giảm đáng kể chi phí vận hành và cải thiện trải nghiệm người dùng. Các framework như vLLM đã dân chủ hóa việc tiếp cận các kỹ thuật này, nhưng việc tinh chỉnh chuyên sâu vẫn là yếu tố quyết định để đạt được hiệu suất đỉnh cao. 

Trong tương lai, các xu hướng như serving phân tán (distributed serving) với cơ chế định tuyến thông minh dựa trên prefix (prefix-aware routing) và sử dụng các tầng bộ nhớ khác nhau (tiered KV cache) để mở rộng dung lượng cache hứa hẹn sẽ tiếp tục đẩy xa hơn nữa giới hạn về hiệu suất và hiệu quả của các hệ thống phục vụ LLM.

---

### Tài liệu tham khảo

[1] Kim, D. (2024). *A Deep Dive into GPTQ and AWQ Quantization*. Medium. Truy cập tại: https://medium.com/@kimdoil1211/speeding-up-large-language-models-a-deep-dive-into-gptq-and-awq-quantization-0bb001eaabd4

[2] vLLM Project. *Automatic Prefix Caching*. vLLM Documentation. Truy cập tại: https://docs.vllm.ai/en/stable/design/prefix_caching.html

[3] BentoML. *Prefix caching | LLM Inference Handbook*. BentoML. Truy cập tại: https://bentoml.com/llm/inference-optimization/prefix-caching

[4] Moon, D. (2024). *LLM Inference Optimizations - Chunked Prefills and Decode Maximal Batching*. Medium. Truy cập tại: https://donmoon.medium.com/llm-inference-optimizations-2-chunked-prefill-764407b3a67a

[5] Hanley, E., & Rockwell, B. (2025). *vLLM Performance Tuning: The Ultimate Guide to xPU Inference Configuration*. Google Cloud Blog. Truy cập tại: https://cloud.google.com/blog/topics/developers-practitioners/vllm-performance-tuning-the-ultimate-guide-to-xpu-inference-configuration

[6] Li, J., Yu, C., & Guo, H. (2025). *An Introduction to Speculative Decoding for Reducing Latency in AI Inference*. NVIDIA Technical Blog. Truy cập tại: https://developer.nvidia.com/blog/an-introduction-to-speculative-decoding-for-reducing-latency-in-ai-inference/

[7] Dao, T. (2024). *FlashAttention-3: Fast and Accurate Attention with Fused Heads*. Tri Dao's Blog. Truy cập tại: https://tridao.me/blog/2024/flash3/

[8] vLLM Project. *CUDA Graphs*. vLLM Documentation. Truy cập tại: https://docs.vllm.ai/en/stable/design/cuda_graphs/

---

## 💡 Tóm tắt chiến thuật cho Pika:

1. **Mở nhiều làn (`max-num-seqs` cao):** Vì Pika là model tí hon, hãy tận dụng để phục vụ cả lớp học cùng lúc.
    
2. **Đừng xây băng chuyền quá dài (`max-model-len` vừa đủ):** Pika chỉ chat ngắn, đừng lãng phí tài nguyên cho những băng chuyền dài lê thê.
    
3. **Luôn bật chế độ "Copy bài mẫu" (`prefix-caching`):** Đây là bí mật để Pika phản hồi nhanh như điện khi có nhiều người hỏi cùng lúc.
    
4. **Để thợ tự làm (`enforce-eager = False`):** Đừng bắt CPU quản lý vi mô, hãy để GPU tự do chạy hết tốc lực (CUDA Graphs).


## 3.1 Ví dụ serving HuggingFaceTB/SmolLM2-135M-Instruct


```

#!/bin/bash

# ============================================================

# RUN SMOLLM2-135M EMOTION CLASSIFIER - Ultra Fast Tiny Model

# Target: GPU 0 (~12GB free VRAM - nhưng model này chỉ cần ~300MB!)

# Dự kiến latency: < 10ms (nhanh hơn Phi-3 gấp 3-5 lần)

# ============================================================

  

set -e

  

# Colors

GREEN='\033[0;32m'

YELLOW='\033[1;33m'

RED='\033[0;31m'

NC='\033[0m'

  

echo -e "${GREEN}============================================${NC}"

echo -e "${GREEN}  SMOLLM2-135M EMOTION CLASSIFIER (ULTRA)   ${NC}"

echo -e "${GREEN}============================================${NC}"

  

# Configuration

VENV_DIR="$HOME/venv_phi3"  # Tái sử dụng venv cũ

GPU_ID=0

PORT=30030

MODEL_NAME="HuggingFaceTB/SmolLM2-135M-Instruct"  # ⚡ TINY MODEL

GPU_MEMORY_UTIL=0.15  # Chỉ cần 15% VRAM (~1.8GB)

MAX_MODEL_LEN=2048

MAX_NUM_SEQS=128      # Tăng batch size vì model nhỏ

  
  

# Step 1: Check GPU

echo -e "\n${YELLOW}[1/3] Checking GPU ${GPU_ID} availability...${NC}"

FREE_MEM=$(nvidia-smi -i $GPU_ID --query-gpu=memory.free --format=csv,noheader,nounits)

echo "GPU ${GPU_ID} Free Memory: ${FREE_MEM} MiB"

  

if [ "$FREE_MEM" -lt 2000 ]; then

    echo -e "${RED}WARNING: GPU ${GPU_ID} has less than 2GB free!${NC}"

    echo "SmolLM2-135M only needs ~300MB, but 2GB is recommended for overhead."

fi

  
  

# Step 2: Activate venv

echo -e "\n${YELLOW}[2/3] Activating Python environment...${NC}"

  

if [ ! -d "$VENV_DIR" ]; then

    echo -e "${RED}ERROR: Virtual environment not found at $VENV_DIR${NC}"

    echo "Creating new venv..."

    python3.11 -m venv "$VENV_DIR"

    source "$VENV_DIR/bin/activate"

    echo "Installing vLLM..."

    pip install --upgrade pip

    pip install vllm==0.12.0 torch==2.5.1

else

    source "$VENV_DIR/bin/activate"

    echo "✓ Activated venv: $VENV_DIR"

fi

  

# Verify

if ! python -c "import vllm" 2>/dev/null; then

    echo -e "${RED}ERROR: vLLM not found!${NC}"

    exit 1

fi

  

VLLM_VERSION=$(python -c "import vllm; print(vllm.__version__)" 2>/dev/null)

echo "✓ vLLM version: $VLLM_VERSION"

  
  

# Step 3: Launch server

echo -e "\n${YELLOW}[3/3] Launching vLLM server...${NC}"

echo -e "${GREEN}============================================${NC}"

echo "Model: ${MODEL_NAME} (135M params)"

echo "GPU: ${GPU_ID}"

echo "Port: ${PORT}"

echo "Memory Utilization: ${GPU_MEMORY_UTIL} (~300MB only)"

echo "Expected Latency: < 10ms per request"

echo -e "${GREEN}============================================${NC}"

echo ""

echo "🚀 ULTRA-FAST MODE: SmolLM2 is 3.7x smaller than Qwen-0.5B"

echo "Server starting... Press Ctrl+C to stop"

echo ""

  

# Log file

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

LOG_FILE="$SCRIPT_DIR/smollm2_server.log"

  

# Launch vLLM với SmolLM2-135M

# NOTE: Model này không có sẵn AWQ version, nhưng FP16 đã đủ nhanh!

CUDA_VISIBLE_DEVICES=$GPU_ID python -m vllm.entrypoints.openai.api_server \

    --model "$MODEL_NAME" \

    --dtype float16 \

    --host 0.0.0.0 \

    --port $PORT \

    --gpu-memory-utilization $GPU_MEMORY_UTIL \

    --max-model-len $MAX_MODEL_LEN \

    --max-num-seqs $MAX_NUM_SEQS \

    --enable-prefix-caching \

    --trust-remote-code \

    --disable-log-requests \

    2>&1 | tee "$LOG_FILE"

  
  

# ALTERNATIVE: Nếu muốn quantize thủ công để nhanh hơn nữa

# Uncomment dòng dưới và comment block trên

# CUDA_VISIBLE_DEVICES=$GPU_ID python -m vllm.entrypoints.openai.api_server \

#     --model "prithivMLmods/SmolLM2-135M-GGUF" \

#     --quantization gguf \

#     --dtype auto \

#     --host 0.0.0.0 \

#     --port $PORT \

#     --gpu-memory-utilization $GPU_MEMORY_UTIL \

#     --max-model-len $MAX_MODEL_LEN \

#     --max-num-seqs $MAX_NUM_SEQS \

#     --enable-prefix-caching \

#     --trust-remote-code \

#     2>&1 | tee "$LOG_FILE"

  

```

  
  

```

curl --location 'http://103.253.20.30:30030/v1/chat/completions' \

--header 'Content-Type: application/json' \

--data '{

    "model": "HuggingFaceTB/SmolLM2-135M-Instruct",

    "messages":  [

        {

            "role": "system",

            "content": "You are now intention detection. Given user'\''s input, detect the suitable emotion and the need of celebrate for it.\nUser input in format\nprevious Question: string\nprevious Answer: string\nResponse to check: string to check\n\nYou will extract the '\''Response to check'\'' and check:\n\n1. For emotion, pick from the list below:\n- happy, happy_2: when intention about happiness\n- calm: when intentionis to comfort\n- excited, excited_2: when expressing the exciting emotion\n- playful,playful_2,playful_3: intention about playing some fun activity\n- no_problem: intention when telling something is fine\n- encouraging,encouraging_2: intention when tell to try to do something\n- curious: intention when show curiousity\n- surprised: when being shocked or surprised\n- proud,proud_2: when showing proud\n- thats_right, thats_right_2: when telling something is correct\n- sad: when sad\n- angry: showing anger\n- worry: when show worriness\n- afraid: when feel scared\n- noisy: When intention about can'\''t hear properly\n- thinking: intention about thinking\n\n2. For learn_score, if related to english learning\n- true: if question is about english learning, repeating something, and answer correctly\n- false: for other case include positive and negative\n\n\n return in single line json format.\n{\"emotion\":\"<emotion name>\",\"learn_score\":true/flase}"

        },

        {

            "role": "user",

            "content": "Previous Question: Tớ buồn quá. \nPrevious Answer: i think a yummy\nResponse to check: Nghe vui quá! Bể Hả, cậu có muốn chơi trò kể tên các loại trái cây bằng tiếng Anh không? Name fruits in English!"

        }

    ],

    "temperature": 0,

    "max_tokens": 50

  }'

```

  
  

```

{

    "id": "chatcmpl-8ca7b299c9702fde",

    "object": "chat.completion",

    "created": 1765172321,

    "model": "HuggingFaceTB/SmolLM2-135M-Instruct",

    "choices": [

        {

            "index": 0,

            "message": {

                "role": "assistant",

                "content": "\"Tớ buồn quá. Nghe vui quá!\" Bể Hả, cậu có muốn chơi trò kể tên các",

                "refusal": null,

                "annotations": null,

                "audio": null,

                "function_call": null,

                "tool_calls": [],

                "reasoning": null,

                "reasoning_content": null

            },

            "logprobs": null,

            "finish_reason": "length",

            "stop_reason": null,

            "token_ids": null

        }

    ],

    "service_tier": null,

    "system_fingerprint": null,

    "usage": {

        "prompt_tokens": 428,

        "total_tokens": 478,

        "completion_tokens": 50,

        "prompt_tokens_details": null

    },

    "prompt_logprobs": null,

    "prompt_token_ids": null,

    "kv_transfer_params": null

}

```

  

Ket qua 170ms ()

  

Thấp hơn qwen0.6B = 50-80ms tren H100, tren cung server là 150ms

va thap hon Groq oss-20b/120b gpt mà có 150ms


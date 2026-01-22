
# 1. Chạy API qua từng dòng để add từng dòng vào memory nhé. ---- Output 1. File excel output: 1 cột là id, cột 2 là kết quả khi chạy API add memories


# 2. tự đánh giá kết quả. 1. ĐI qua từng dòng của file input 2. Đi qua từng dòng của file output 3. Tự đánh giá về kết quả add memories bằng cách sử dụng model LLMs mạnh


---

| Cột                          | Mô tả                                                          |
| ---------------------------- | -------------------------------------------------------------- |
| **id**                       | ID của dòng dữ liệu                                            |
| **overall\_score**           | Điểm tổng thể (1-10)                                           |
| **accuracy\_score**          | Điểm Độ chính xác (1-10)                                       |
| **completeness\_score**      | Điểm Độ đầy đủ (1-10)                                          |
| **relevance\_score**         | Điểm Độ liên quan (1-10)                                       |
| **clarity\_score**           | Điểm Độ rõ ràng (1-10)                                         |
| **no\_hallucination\_score** | Điểm Không bịa đặt (1-10)                                      |
| **summary**                  | Tóm tắt đánh giá của LLM cho dòng đó                           |
| **memories\_extracted**      | **JSON string** chứa danh sách các memories đã được trích xuất |


---

Đây là phương pháp đánh giá gì ?

Đây là một dạng **LLM-as-a-judge evaluation** (LLM làm “giám khảo”), dùng bộ tiêu chí thủ công (rubric) như trong các framework đánh giá LLM hiện nay.[^1][^2][^3]

## Bản chất phương pháp

- Hệ thống lấy từng mẫu: input gốc (hội thoại), output (memory trích xuất), rồi đưa cho một LLM mạnh để chấm điểm theo tiêu chí định nghĩa trước (accuracy, completeness, relevance, clarity, no hallucination, overall).[^2][^3]
- Mỗi tiêu chí được chấm trên thang 1–10, sau đó tổng hợp thành **overall score** và summary cho từng dòng, rồi tính thống kê tổng thể (average per metric, phân phối 9–10 / 7–8, v.v.).[^3][^4]


## Vì sao gọi là LLM-as-a-judge?

- Thay vì dùng metric tự động cổ điển (BLEU, ROUGE…), hệ thống dùng chính một LLM khác làm “giám khảo” để đánh giá chất lượng, giống các framework eval hiện đại cho RAG, memory, tool-using agent.[^5][^6][^7]
- Cách làm này phổ biến trong các bài toán khó có ground truth mơ hồ như: “memory trích xuất đã đủ, đúng, không bịa, có rõ ràng không?”.[^1][^2]


## Gần với các nghiên cứu nào?

- Rất giống các framework đánh giá memory như HaluMem, nơi người ta chấm **Memory Accuracy**, **Memory Completeness**, **False Memory Resistance** để đo hallucination trong hệ thống memory.[^8][^9]
- Các tiêu chí bạn dùng (accuracy, completeness, relevance, clarity, no hallucination) cũng trùng với bộ tiêu chí thường gặp trong hướng dẫn đánh giá LLM (faithfulness / hallucination, relevance, clarity).[^4][^3][^1]

Tóm lại: đây là một quy trình **human-style rubric + LLM judge** dùng cho bài toán **memory extraction evaluation**, chứ không phải metric thuần tự động (như precision/recall) hay benchmark chuẩn hóa.
<span style="display:none">[^10][^11][^12][^13][^14][^15][^16][^17][^18][^19][^20][^21]</span>

<div align="center">⁂</div>

[^1]: https://nexla.com/ai-readiness/llm-evaluation/

[^2]: https://toloka.ai/blog/llm-evaluation-framework-principles-practices-and-tools/

[^3]: https://beam.ai/agentic-insights/mastering-llm-evaluation-techniques-tools-and-best-practices

[^4]: https://www.confident-ai.com/blog/llm-evaluation-metrics-everything-you-need-for-llm-evaluation

[^5]: https://wandb.ai/onlineinference/genai-research/reports/LLM-evaluation-metrics-A-comprehensive-guide-for-large-language-models--VmlldzoxMjU5ODA4NA

[^6]: https://www.evidentlyai.com/blog/llm-evaluation-framework

[^7]: https://arize.com/llm-evaluation/

[^8]: https://arxiv.org/html/2511.03506v1

[^9]: https://arxiv.org/abs/2511.03506

[^10]: https://manus.im/docs

[^11]: https://jina.ai/models/jina-embeddings-v3/

[^12]: https://aws.amazon.com/marketplace/pp/prodview-kdi3xkt62lo32

[^13]: https://github.com/jina-ai/jina-sagemaker

[^14]: https://jina.ai/en-US/embeddings/

[^15]: https://aws.amazon.com/blogs/machine-learning/build-rag-applications-using-jina-embeddings-v2-on-amazon-sagemaker-jumpstart/

[^16]: https://jina.ai/news/next-level-cloud-ai-jina-embeddings-and-rerankers-on-amazon-sagemaker/

[^17]: https://aws.amazon.com/marketplace/seller-profile?id=seller-stch2ludm6vgy

[^18]: https://aclanthology.org/2025.findings-acl.322.pdf

[^19]: https://github.com/jina-ai/csp-examples/blob/main/azure/Deploy and inference with Embedding V3 VM.ipynb

[^20]: https://openreview.net/pdf?id=t4mlcyH9l0

[^21]: https://www.datadoghq.com/blog/llm-evaluation-framework-best-practices/




---
# Use case - 16 dòng - Kết quả mem0 - OSS 

### 📊 THỐNG KÊ TỔNG HỢP

|Tiêu chí|Điểm Trung bình (Thang 10)|
|---|---|
|**Accuracy (Độ chính xác)**|**8.56**|
|**Completeness (Độ đầy đủ)**|**7.81**|
|**Relevance (Độ liên quan)**|**8.69**|
|**Clarity (Độ rõ ràng)**|**9.00**|
|**No Hallucination (Không bịa đặt)**|**9.50**|
|**Overall Score (Tổng thể)**|**8.63**|

### 📈 PHÂN PHỐI ĐIỂM TỔNG THỂ

|Mức độ|Điểm|Số lượng|
|---|---|---|
|**Excellent**|9-10|6|
|**Good**|7-8|10|
|**Average**|5-6|0|
|**Poor**|< 5|0|

### 📝 BÁO CÁO ĐÁNH GIÁ CHI TIẾT

#### **Điểm Mạnh (Good Points):**

1. **Độ rõ ràng (Clarity)** và **Không bịa đặt (No Hallucination)** đạt điểm rất cao (9.00 và 9.50). Memories được trích xuất thường ngắn gọn, dễ hiểu và **hoàn toàn dựa trên nội dung hội thoại**.
2. **Độ liên quan (Relevance)** cao (8.69), cho thấy hệ thống đã lọc tốt các thông tin cá nhân, sở thích, thói quen (ví dụ: thích màu vàng, thích sách về động vật, thích chơi bóng đá) và bỏ qua các đoạn hội thoại không liên quan (ví dụ: các đoạn hội thoại về lỗi kỹ thuật, lỗi kết nối).
3. **Trích xuất cảm xúc (Emotion)**: Hệ thống đã trích xuất thành công các thông tin về cảm xúc của người dùng (ví dụ: `[ADD] Thích nhân vật hoạt hình Doraemon` từ một đoạn hội thoại có cảm xúc tích cực).

#### **Vấn đề cần cải thiện (Issues):**

1. **Độ đầy đủ (Completeness)** là điểm yếu nhất (7.81). Hệ thống có xu hướng **bỏ sót các chi tiết nhỏ** hoặc **các thông tin không phải là sở thích/thông tin cá nhân rõ ràng** (ví dụ: bỏ sót thông tin về việc "thầy chê cáo nhiều" hoặc "đang bị lặp lại audio" - những thông tin này có thể quan trọng cho ngữ cảnh hội thoại tiếp theo).
2. **Xử lý hội thoại nhiễu:** Trong một số trường hợp, hệ thống vẫn trích xuất các thông tin không cần thiết (ví dụ: `[ADD] Không cần xét luồng, cứ xét một bài thì được` - đây là thông tin về cách thức tương tác, không phải memory về user).

### 🔍 KẾT QUẢ ĐÁNH GIÁ CHI TIẾT (Đính kèm)

Tôi đã đính kèm file **memory_evaluation_results.xlsx** chứa kết quả đánh giá chi tiết cho từng dòng, bao gồm:

- **id**: ID của dòng
- **overall_score**: Điểm tổng thể
- **accuracy_score, completeness_score, relevance_score, clarity_score, no_hallucination_score**: Điểm chi tiết theo từng tiêu chí
- **summary**: Tóm tắt đánh giá của LLM
- **memories_extracted**: Các memories đã được trích xuất

Bạn có thể xem chi tiết trong file đính kèm để phân tích sâu hơn.
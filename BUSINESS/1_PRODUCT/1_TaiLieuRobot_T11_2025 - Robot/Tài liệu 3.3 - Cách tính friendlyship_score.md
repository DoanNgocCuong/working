
```
**a. Thu thập các chỉ số từ `daily_metrics`:**

* `total_turns`: Tổng số lượt trò chuyện trong ngày. => BE bắt (vì nếu AI bắt thì bắt kiểu gì)

* `user_initiated_questions`: Số lần người dùng chủ động hỏi Pika. (Cái này làm sao để mà tính, lồng LLMs vào luồng học ? )

* `followup_topics_count`: Tên chủ đề mới do người dùng gợi ý.=> Là cái gì, và BE bắt hay sao.

* `session_emotion`: Cảm xúc chủ đạo trong ngày ('interesting', 'boring', 'neutral', 'angry', 'happy','sad'). => Đo cảm xúc mỗi bài hay mỗi ngày ?

* `new_memories_count`: Số ký ức mới được tạo. => tính toán ở module Mem0???.

* `topic_details`: Chi tiết tương tác cho từng topic (số turn, số câu hỏi). => ??? Làm sao để extract.

Bạn có recommend cách tính score nào dễ dàng cho các bên ko?
```
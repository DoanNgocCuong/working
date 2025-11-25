# Tài liệu Triển khai Kỹ thuật: Module Context Handling - Friendlyship Management

**Version:** 1.0
**Date:** 25/11/2025
**Author:** Manus AI

## 1. Tổng quan và Bối cảnh (Overview and Context)

Tài liệu này đặc tả chi tiết về mặt kỹ thuật cho việc xây dựng và tích hợp module **Context Handling**, với trọng tâm là quản lý trạng thái tình bạn (Friendship) và lựa chọn Agent (Talk/Game/Greeting) trong hệ sinh thái sản phẩm Pika. Module này là một phần của **Container 3: Context Handling** trong kiến trúc tổng thể, chịu trách nhiệm thu thập, xử lý, và duy trì tất cả dữ liệu liên quan đến người dùng và mối quan hệ của họ với Pika.

### 1.1. Mục tiêu Product

- **Tăng Retention và Engagement:** Tạo ra một mối quan hệ cá nhân hóa, sâu sắc và lâu dài giữa người dùng và Pika, khiến người dùng cảm thấy được thấu hiểu và quay trở lại thường xuyên.
- **Cá nhân hóa Trải nghiệm:** Chuyển đổi từ trải nghiệm "một cho tất cả" sang "một cho mỗi người", nơi các hoạt động, lời chào và chủ đề trò chuyện được điều chỉnh dựa trên lịch sử tương tác và mức độ thân thiết.
- **Tạo ra các khoảnh khắc "Aha!":** Khiến người dùng bất ngờ và thích thú khi Pika "nhớ" lại các chi tiết, sở thích, hoặc các sự kiện trong quá khứ, tạo ra một kết nối cảm xúc thực sự.

### 1.2. Thay đổi so với Thiết kế ban đầu

Dựa trên yêu cầu mới, luồng cập nhật điểm tình bạn (`friendship_score`) sẽ được thay đổi từ mô hình xử lý hàng loạt cuối ngày (batch processing) sang **mô hình xử lý theo thời gian thực (real-time processing)**. 

> **Yêu cầu cốt lõi:** *"Sau khi kết thúc 1 cuộc hội thoại phía BE gửi user_id kèm log cho phía AI. Phía AI xử lý log luôn và tính điểm daily_score và code API phía BE để update điểm friendlyship_score."*

Điều này có nghĩa là `friendship_score` sẽ được cập nhật liên tục sau mỗi phiên tương tác, mang lại phản hồi tức thì về mức độ thân thiết và cho phép hệ thống điều phối (Orchestration) có được dữ liệu mới nhất để ra quyết định.

## 2. Thiết kế Kiến trúc Module

Để đáp ứng yêu cầu xử lý real-time, kiến trúc của module sẽ bao gồm ba thành phần chính: **Backend (BE) Service**, **AI Scoring Service**, và **Friendship Database**.

```mermaid
sequenceDiagram
    participant User as User
    participant BE as Backend Service
    participant AI as AI Scoring Service
    participant DB as Friendship Database

    User->>BE: End Conversation
    BE->>AI: POST /v1/scoring/calculate-friendship (user_id, conversation_log)
    activate AI
    AI-->>AI: 1. Phân tích log (tính total_turns, emotion, etc.)
    AI-->>AI: 2. Tính toán friendship_score_change
    deactivate AI
    AI->>BE: POST /v1/users/{user_id}/update-friendship (friendship_score_change, topic_metrics_change, ...)
    activate BE
    BE->>DB: 1. Read current friendship_status
    BE-->>BE: 2. Calculate new scores (friendship_score, topic_scores)
    BE-->>BE: 3. Update friendship_level, streak_day, etc.
    BE->>DB: 3. Write updated friendship_status
    deactivate BE
    BE-->>AI: Response: {success: true}
    AI-->>BE: Response: {success: true}

```
*Sơ đồ 1: Luồng cập nhật Friendship Score theo thời gian thực*

### Luồng hoạt động:
1.  **Kết thúc hội thoại:** Người dùng hoàn thành một phiên trò chuyện.
2.  **BE gửi yêu cầu:** Backend Service gửi một yêu cầu (POST) đến AI Scoring Service, đính kèm `user_id` và toàn bộ `conversation_log` của phiên vừa kết thúc.
3.  **AI tính toán:** AI Scoring Service nhận log, phân tích và tính toán ra một "điểm thay đổi" (`friendship_score_change`) cùng các chỉ số liên quan khác (ví dụ: sự thay đổi của `topic_score`).
4.  **AI gọi BE để cập nhật:** AI Service gọi một API do BE cung cấp để gửi "điểm thay đổi" này.
5.  **BE cập nhật vào DB:** BE nhận điểm thay đổi, đọc bản ghi `friendship_status` hiện tại từ Database, tính toán các giá trị mới, và ghi đè bản ghi đã cập nhật trở lại vào Database.

## 3. Định nghĩa Database Schema và Data Structure

Cấu trúc dữ liệu sẽ được lưu trong một cơ sở dữ liệu SQL (Postgress). Cấu trúc này dựa trên `Tài liệu 1` nhưng có điều chỉnh để phù hợp với logic cập nhật real-time.

**Collection/Table:** `friendship_status`

| Tên trường | Kiểu dữ liệu | Mô tả | Ghi chú | 
| :--- | :--- | :--- | :--- |
| `user_id` | String | (Primary Key) Mã định danh duy nhất của người dùng. | Bắt buộc. |
| `friendship_score` | Float | Điểm số tổng thể đo lường mức độ thân thiết. | Cập nhật sau mỗi phiên. |
| `friendship_level` | String | Cấp độ tình bạn: `STRANGER`, `ACQUAINTANCE`, `FRIEND`. | Tự động cập nhật dựa trên `friendship_score`. |
| `last_interaction_date` | ISO 8601 Timestamp | Dấu thời gian của lần tương tác cuối cùng. | Cập nhật sau mỗi phiên. |
| `streak_day` | Integer | Số ngày tương tác liên tiếp. | Cập nhật sau mỗi phiên. |
| `topic_metrics` | Object / Map | Lưu trữ điểm và lịch sử tương tác cho từng chủ đề. | Cập nhật sau mỗi phiên. |
| `dynamic_memory` | Array of Objects | Danh sách các "ký ức chung" giữa Pika và người dùng. | Thêm mới khi có sự kiện đáng nhớ. |

**Lưu ý:** Trường `daily_metrics` trong thiết kế ban đầu sẽ không còn cần thiết, vì các chỉ số giờ đây được xử lý và tích luỹ trực tiếp vào các trường chính sau mỗi phiên, thay vì được thu thập tạm thời và xử lý cuối ngày.

### Ví dụ bản ghi:

```json
{
  "user_id": "user_123",
  "friendship_score": 785.5, // Đã được cập nhật sau phiên gần nhất
  "friendship_level": "ACQUAINTANCE",
  "last_interaction_date": "2025-11-25T18:30:00Z",
  "streak_day": 6, // Tăng lên vì hôm qua cũng tương tác
  "topic_metrics": {
    "agent_movie": {
      "topic_score": 52.0, // Đã tăng sau phiên nói về phim
      "total_turns": 65,
      "last_talked_date": "2025-11-25T18:25:00Z"
    }
  },
  "dynamic_memory": [
    {
      "memory_id": "mem_003",
      "content": "Thích xem phim của đạo diễn Hayao Miyazaki.",
      "related_topic": "agent_movie",
      "timestamp": "2025-11-25T18:28:00Z"
    }
  ]
}
```

## 4. Thiết kế API Endpoints

Sẽ có 2 API chính được định nghĩa để phục vụ cho module này.

### 4.1. API 1: Tính toán Friendship Score (BE -> AI)


>    **a. Thu thập các chỉ số từ `daily_metrics`:**
    *   `total_turns`: Tổng số lượt trò chuyện trong ngày.
    *   `user_initiated_questions`: Số lần người dùng chủ động hỏi Pika.
    *   `followup_topics_count`: Tên chủ đề mới do người dùng gợi ý.
    *   `session_emotion`: Cảm xúc chủ đạo trong ngày ('interesting', 'boring', 'neutral', 'angry', 'happy','sad').
    *   `new_memories_count`: Số ký ức mới được tạo. 
    *   `topic_details`: Chi tiết tương tác cho từng topic (số turn, số câu hỏi).

> Logic Mapping Friendship vs Kho



API này cho phép BE yêu cầu AI phân tích một cuộc hội thoại và trả về các điểm số cần cập nhật.

- **Endpoint:** `POST /v1/scoring/calculate-friendship`
- **Service:** AI Scoring Service
- **Mô tả:** Nhận log hội thoại, tính toán và trả về các thay đổi về điểm tình bạn và các chỉ số liên quan.
- **Request Body:**

  ```json
  {
    "user_id": "user_123",
    "conversation_log": [
      {"speaker": "user", "turn_id": 1, "text": "Hello Pika!"},
      {"speaker": "pika", "turn_id": 2, "text": "Hi there! How are you?"},
      // ... thêm các turn khác
    ],
    "session_metadata": {
        "emotion": "interesting", // Do AI tự phân tích hoặc BE gửi sang
        "new_memories_created": 2
    }
  }
  ```

- **Response Body (Success 200):**

  ```json
  {
    "friendship_score_change": 35.0,
    "topic_metrics_update": {
        "agent_movie": {
            "score_change": 7.0,
            "turns_increment": 15
        }
    },
    "new_memories": [
        {
          "content": "Thích xem phim của đạo diễn Hayao Miyazaki.",
          "related_topic": "agent_movie"
        }
    ]
  }
  ```

### 4.2. API 2: Cập nhật Friendship Status (AI -> BE)

API này cho phép AI gửi các điểm số đã tính toán để BE cập nhật vào cơ sở dữ liệu.

- **Endpoint:** `POST /v1/users/{user_id}/update-friendship`
- **Service:** Backend Service
- **Mô tả:** Nhận các thay đổi về điểm số và chỉ số từ AI, sau đó cập nhật vào bản ghi `friendship_status` của người dùng trong DB.
- **Path Parameter:** `user_id` (String, required)
- **Request Body:** (Giống hệt Response Body của API 1)

- **Response Body (Success 200):**

  ```json
  {
    "success": true,
    "message": "Friendship status updated successfully."
  }
  ```

### 4.3. API 3: Lấy danh sách Agent đề xuất (BE -> AI/Orchestration)

API này phục vụ cho việc lấy danh sách các hoạt động (Greeting, Talk, Game) được cá nhân hóa cho người dùng khi bắt đầu một phiên mới.

- **Endpoint:** `GET /v1/users/{user_id}/suggested-activities`
- **Service:** AI Orchestration Service (hoặc một service riêng cho việc lựa chọn)
- **Mô tả:** Dựa trên `friendship_status` của người dùng, chọn ra một danh sách các hoạt động phù hợp.
- **Query Parameters:**
  - `type`: (String, optional) Loại agent cần lấy, ví dụ: `greeting`, `talk`, `game`. Nếu không có, trả về cả gói.
  - `count`: (Integer, optional) Số lượng cần lấy.
- **Response Body (Success 200):**

  ```json
  {
    "greeting_agent": {
        "agent_id": "greeting_streak_milestone_5_days",
        "type": "greeting"
    },
    "suggested_agents": [
        {"agent_id": "talk_agent_movie_preference", "type": "talk"},
        {"agent_id": "game_agent_drawing_challenge", "type": "game"},
        {"agent_id": "talk_agent_school_life", "type": "talk"},
        {"agent_id": "talk_agent_follow_up_pet_milu", "type": "talk"}
    ]
  }
  ```

## 5. Logic Xử lý Scoring và Selection

### 5.1. Logic tính điểm (Scoring Logic - Real-time)

Logic này được thực thi trong **AI Scoring Service** sau mỗi cuộc hội thoại, dựa trên `Tài liệu 2` nhưng được điều chỉnh cho phù hợp.

1.  **Thu thập chỉ số từ `conversation_log`:**
    *   `total_turns`: Tổng số lượt trò chuyện trong phiên.
    *   `user_initiated_questions`: Số lần người dùng chủ động hỏi Pika.
    *   `session_emotion`: Cảm xúc chủ đạo của phiên (ví dụ: 'interesting', 'boring').
    *   `new_memories_count`: Số ký ức mới được tạo trong phiên.
    *   `topic_details`: Chi tiết tương tác cho từng topic (số turn, số câu hỏi).

2.  **Tính toán `friendship_score_change`:**
    *   `base_score = total_turns * 0.5`
    *   `engagement_bonus = (user_initiated_questions * 3)`
    *   `emotion_bonus`: +15 cho 'interesting', -15 cho 'boring'.
    *   `memory_bonus = new_memories_count * 5`
    *   **`friendship_score_change`** = `base_score + engagement_bonus + emotion_bonus + memory_bonus`

3.  **Tính toán `topic_metrics_update`:**
    *   Với mỗi topic trong `topic_details`, tính `score_change` = (`turns` * 0.5 + `user_questions` * 3) và `turns_increment` = `turns`.

### 5.2. Logic lựa chọn Agent (Selection Logic)

Logic này được thực thi trong **AI Orchestration Service** (API 3) và tuân thủ chặt chẽ theo `Tài liệu 3`.

1.  **Tải dữ liệu và Xác định Phase:** Lấy `friendship_status` mới nhất của user, xác định `Phase` (Stranger, Acquaintance, Friend) từ `friendship_score`.
2.  **Lọc Kho Hoạt động:** Giới hạn các kho Greeting, Talk, Game dựa trên `Phase`.
3.  **Chọn Greeting:** Dựa trên các quy tắc ưu tiên (sinh nhật, quay lại sau thời gian dài, cảm xúc phiên trước, v.v.).
4.  **Chọn 4 Talk/Game:** Sử dụng phương pháp `Weighted Candidate Selection`:
    *   Tạo danh sách ứng viên từ sở thích (`topic_score` cao), khám phá (ít tương tác), cảm xúc, và game.
    *   Lắp ráp danh sách cuối cùng, cân bằng tỷ lệ Talk:Game, và chống lặp.
5.  **Trả về kết quả:** Gửi danh sách `agent_id` đã được lựa chọn.

### 5.3 Visulize Logic Chọn lựa Agent 

#### Mermaid Diagrams: Logic Chọn Talk/Game-Agent Đầu Ngày

######## 1. Flowchart Tổng Quan - Quy Trình Lựa Chọn Hoàn Chỉnh

```mermaid
flowchart TD
    Start([Người dùng mở app đầu ngày]) --> Input[Input: user_id]
    Input --> Step1[Bước 1: Tải dữ liệu<br/>& Xác định Phase]
    
    Step1 --> LoadData[Tải friendship_status]
    LoadData --> DeterminePhase{Xác định Phase<br/>dựa trên friendship_score}
    
    DeterminePhase -->|score < 500| Phase1[Phase 1: STRANGER]
    DeterminePhase -->|500 ≤ score ≤ 3000| Phase2[Phase 2: ACQUAINTANCE]
    DeterminePhase -->|score > 3000| Phase3[Phase 3: FRIEND]
    
    Phase1 --> Step2[Bước 2: Lọc Kho Hoạt Động]
    Phase2 --> Step2
    Phase3 --> Step2
    
    Step2 --> FilterPools[Giới hạn kho theo Phase:<br/>- Greeting Pool<br/>- Talk Agent Pool<br/>- Game/Activity Pool]
    
    FilterPools --> Step3[Bước 3: Chọn Greeting]
    Step3 --> GreetingLogic[Priority-Based Selection]
    GreetingLogic --> GreetingSelected[greeting_id được chọn]
    
    GreetingSelected --> Step4[Bước 4: Chọn 4 Talk/Game]
    Step4 --> BuildCandidates[4a. Tạo danh sách ứng viên]
    
    BuildCandidates --> Pref[Ứng viên sở thích<br/>2 Agent topic_score cao nhất]
    BuildCandidates --> Explore[Ứng viên khám phá<br/>1 Agent ít tương tác]
    BuildCandidates --> Emotion[Ứng viên cảm xúc<br/>Nếu lastday_emotion tiêu cực]
    BuildCandidates --> Game[Ứng viên Game<br/>Từ kho Game của Phase]
    
    Pref --> Assemble[4b. Lắp ráp danh sách cuối cùng]
    Explore --> Assemble
    Emotion --> Assemble
    Game --> Assemble
    
    Assemble --> ApplyRatio[Áp dụng tỷ lệ Talk:Activity<br/>theo Phase]
    ApplyRatio --> AntiDup[Áp dụng bộ lọc chống lặp]
    AntiDup --> WeightPriority[Ưu tiên theo trọng số]
    
    WeightPriority --> Select4[Chọn 4 activity_id]
    
    Select4 --> Step5[Bước 5: Trả về kết quả]
    Step5 --> Output[Output: 1 greeting_id<br/>+ 4 activity_id]
    Output --> End([Kết thúc])
    
    style Start fill:####e1f5e1
    style End fill:####ffe1e1
    style Phase1 fill:####fff4e1
    style Phase2 fill:####e1f0ff
    style Phase3 fill:####f0e1ff
    style Output fill:####e1ffe1
```

---

######## 2. Flowchart Chi Tiết - Xác Định Phase

```mermaid
flowchart LR
    Start([Input: user_id]) --> Load[Tải friendship_status<br/>từ Database]
    Load --> GetScore[Lấy friendship_score]
    
    GetScore --> Check{Kiểm tra<br/>friendship_score}
    
    Check -->|< 500| P1[Phase 1: STRANGER<br/>━━━━━━━━━━<br/>Kho Greeting: V1<br/>Talk: Bề mặt<br/>Game: Đơn giản]
    Check -->|500-3000| P2[Phase 2: ACQUAINTANCE<br/>━━━━━━━━━━<br/>Kho Greeting: V1+V2<br/>Talk: +Trường học, Bạn bè<br/>Game: +Cá nhân hóa]
    Check -->|> 3000| P3[Phase 3: FRIEND<br/>━━━━━━━━━━<br/>Kho Greeting: V1+V2+V3<br/>Talk: +Gia đình, Lịch sử<br/>Game: +Dự án chung]
    
    P1 --> Output([Phase đã xác định])
    P2 --> Output
    P3 --> Output
    
    style Start fill:####e1f5e1
    style Output fill:####e1ffe1
    style P1 fill:####fff4e1
    style P2 fill:####e1f0ff
    style P3 fill:####f0e1ff
```

---

######## 3. Flowchart Chi Tiết - Chọn Greeting (Priority-Based)

```mermaid
flowchart TD
    Start([Bắt đầu chọn Greeting]) --> Input[Input: Phase + friendship_status]
    
    Input --> Check1{Kiểm tra:<br/>user.birthday<br/>== hôm nay?}
    
    Check1 -->|Có| G1[✓ Chọn Greeting:<br/>S2 Birthday]
    Check1 -->|Không| Check2{Kiểm tra:<br/>last_interaction_date<br/>> 7 ngày?}
    
    Check2 -->|Có| G2[✓ Chọn Greeting:<br/>S4 Returning After<br/>Long Absence]
    Check2 -->|Không| Check3{Kiểm tra:<br/>lastday_emotion<br/>== sad?}
    
    Check3 -->|Có| G3[✓ Chọn Greeting:<br/>Agent hỏi cảm xúc]
    Check3 -->|Không| Check4{Kiểm tra:<br/>last_day_follow_up_topic<br/>tồn tại?}
    
    Check4 -->|Có| G4[✓ Chọn Greeting:<br/>Agent follow up topic]
    Check4 -->|Không| Default[Chọn ngẫu nhiên<br/>từ kho Greeting của Phase<br/>ưu tiên chưa dùng gần đây]
    
    G1 --> Output([greeting_id])
    G2 --> Output
    G3 --> Output
    G4 --> Output
    Default --> Output
    
    style Start fill:####e1f5e1
    style Output fill:####e1ffe1
    style G1 fill:####ffe1e1
    style G2 fill:####ffe1e1
    style G3 fill:####ffe1e1
    style G4 fill:####ffe1e1
    style Default fill:####fff4e1
```

---

######## 4. Flowchart Chi Tiết - Tạo Danh Sách Ứng Viên

```mermaid
flowchart TD
    Start([Bắt đầu tạo<br/>danh sách ứng viên]) --> Input[Input: Phase +<br/>friendship_status]
    
    Input --> Parallel{Tạo ứng viên<br/>từ nhiều nguồn}
    
    Parallel --> Source1[Nguồn 1: Sở thích]
    Parallel --> Source2[Nguồn 2: Khám phá]
    Parallel --> Source3[Nguồn 3: Cảm xúc]
    Parallel --> Source4[Nguồn 4: Game]
    
    Source1 --> S1Process[Lấy 2 Agent có<br/>topic_score cao nhất<br/>từ topic_metrics]
    
    Source2 --> S2Process[Lấy 1 Agent ngẫu nhiên<br/>từ kho Talk của Phase<br/>trong top 10 topic<br/>total_turns thấp nhất]
    
    Source3 --> S3Check{lastday_emotion<br/>tiêu cực?}
    S3Check -->|Có| S3Add[Thêm Game/Talk<br/>vui vẻ, hài hước]
    S3Check -->|Không| S3Skip[Bỏ qua]
    
    Source4 --> S4Process[Thêm các Game<br/>từ kho Game của Phase]
    
    S1Process --> Merge[Gộp tất cả ứng viên]
    S2Process --> Merge
    S3Add --> Merge
    S3Skip --> Merge
    S4Process --> Merge
    
    Merge --> CandidateList([Danh sách ứng viên<br/>hoàn chỉnh])
    
    style Start fill:####e1f5e1
    style CandidateList fill:####e1ffe1
    style S1Process fill:####e1f0ff
    style S2Process fill:####fff4e1
    style S3Add fill:####ffe1f0
    style S4Process fill:####f0e1ff
```

---

######## 5. Flowchart Chi Tiết - Lắp Ráp Danh Sách Cuối Cùng

```mermaid
flowchart TD
    Start([Danh sách ứng viên]) --> Input[Input: Candidate List +<br/>Phase]
    
    Input --> Step1[Bước 1: Áp dụng tỷ lệ Talk:Activity]
    
    Step1 --> Ratio{Xác định tỷ lệ<br/>theo Phase}
    
    Ratio -->|Phase 1| R1[Tỷ lệ 40:60<br/>VD: 2 Talk + 2 Game<br/>hoặc 1 Talk + 3 Game]
    Ratio -->|Phase 2| R2[Tỷ lệ theo Phase 2<br/>cân bằng Talk/Game]
    Ratio -->|Phase 3| R3[Tỷ lệ theo Phase 3<br/>linh hoạt hơn]
    
    R1 --> Step2[Bước 2: Áp dụng bộ lọc chống lặp]
    R2 --> Step2
    R3 --> Step2
    
    Step2 --> Filter1[Kiểm tra: Greeting + Talk<br/>không cùng hỏi về cảm xúc]
    Filter1 --> Filter2[Kiểm tra: Không cùng<br/>hỏi về 1 topic]
    Filter2 --> Filter3[Loại bỏ các ứng viên<br/>trùng lặp]
    
    Filter3 --> Step3[Bước 3: Ưu tiên theo trọng số]
    
    Step3 --> Weight[Sắp xếp ưu tiên:<br/>1. Sở thích cao<br/>2. Ký ức liên quan<br/>3. Cảm xúc phù hợp<br/>4. Khám phá mới]
    
    Weight --> Select[Chọn 4 activity_id<br/>từ danh sách đã sắp xếp]
    
    Select --> Output([Output:<br/>4 activity_id])
    
    style Start fill:####e1f5e1
    style Output fill:####e1ffe1
    style R1 fill:####fff4e1
    style R2 fill:####e1f0ff
    style R3 fill:####f0e1ff
    style Weight fill:####ffe1e1
```

---

######## 6. Sequence Diagram - Toàn Bộ Quy Trình

```mermaid
sequenceDiagram
    participant User as Người dùng
    participant App as Ứng dụng
    participant Service as Selection Service
    participant DB as Database
    participant GreetingPool as Greeting Pool
    participant ActivityPool as Talk/Game Pool
    
    User->>App: Mở app đầu ngày
    App->>Service: Gọi với user_id
    
    rect rgb(230, 245, 255)
        Note over Service,DB: BƯỚC 1: Tải dữ liệu & Xác định Phase
        Service->>DB: Query friendship_status(user_id)
        DB-->>Service: Trả về friendship_status
        Service->>Service: Tính Phase từ friendship_score
    end
    
    rect rgb(255, 245, 230)
        Note over Service,ActivityPool: BƯỚC 2: Lọc Kho Hoạt Động
        Service->>GreetingPool: Lọc theo Phase
        GreetingPool-->>Service: Kho Greeting khả dụng
        Service->>ActivityPool: Lọc theo Phase
        ActivityPool-->>Service: Kho Talk/Game khả dụng
    end
    
    rect rgb(230, 255, 230)
        Note over Service,GreetingPool: BƯỚC 3: Chọn Greeting
        Service->>Service: Kiểm tra điều kiện ưu tiên
        alt Birthday
            Service->>GreetingPool: Lấy S2 (Birthday)
        else Long Absence
            Service->>GreetingPool: Lấy S4 (Returning)
        else Sad Emotion
            Service->>GreetingPool: Lấy Agent hỏi cảm xúc
        else Follow-up Topic
            Service->>GreetingPool: Lấy Agent follow-up
        else Default
            Service->>GreetingPool: Lấy ngẫu nhiên
        end
        GreetingPool-->>Service: greeting_id
    end
    
    rect rgb(255, 230, 245)
        Note over Service,ActivityPool: BƯỚC 4: Chọn 4 Talk/Game
        Service->>DB: Query topic_metrics
        DB-->>Service: Trả về topic_metrics
        Service->>Service: Tạo danh sách ứng viên
        Service->>Service: Áp dụng tỷ lệ Talk:Activity
        Service->>Service: Áp dụng bộ lọc chống lặp
        Service->>Service: Sắp xếp theo trọng số
        Service->>ActivityPool: Lấy 4 activity_id
        ActivityPool-->>Service: 4 activity_id
    end
    
    rect rgb(245, 230, 255)
        Note over Service,App: BƯỚC 5: Trả về kết quả
        Service->>Service: Lắp ráp danh sách cuối cùng
        Service-->>App: [greeting_id, activity_id_1, ..., activity_id_4]
        App-->>User: Hiển thị trải nghiệm cá nhân hóa
    end
```

---

######## 7. State Diagram - Trạng Thái Phase Progression

```mermaid
stateDiagram-v2
    [*] --> Stranger: User mới<br/>score = 0
    
    Stranger: Phase 1: STRANGER
    Stranger: score < 500
    Stranger: ━━━━━━━━━━
    Stranger: Greeting: V1
    Stranger: Talk: Bề mặt
    Stranger: Game: Đơn giản
    
    Acquaintance: Phase 2: ACQUAINTANCE
    Acquaintance: 500 ≤ score ≤ 3000
    Acquaintance: ━━━━━━━━━━
    Acquaintance: Greeting: V1+V2
    Acquaintance: Talk: +Trường học, Bạn bè
    Acquaintance: Game: +Cá nhân hóa
    
    Friend: Phase 3: FRIEND
    Friend: score > 3000
    Friend: ━━━━━━━━━━
    Friend: Greeting: V1+V2+V3
    Friend: Talk: +Gia đình, Lịch sử
    Friend: Game: +Dự án chung
    
    Stranger --> Acquaintance: Tương tác tích cực<br/>score ≥ 500
    Acquaintance --> Friend: Tương tác sâu sắc<br/>score > 3000
    
    Acquaintance --> Stranger: Không tương tác lâu<br/>score < 500
    Friend --> Acquaintance: Giảm tương tác<br/>score ≤ 3000
    
    note right of Stranger
        Nội dung an toàn,
        khám phá ban đầu
    end note
    
    note right of Acquaintance
        Cá nhân hóa tăng,
        chủ đề đa dạng hơn
    end note
    
    note right of Friend
        Mối quan hệ sâu sắc,
        nội dung đặc biệt
    end note
```

---

######## 8. Class Diagram - Cấu Trúc Dữ Liệu

```mermaid
classDiagram
    class SelectionService {
        +selectDailyActivities(user_id)
        -loadFriendshipStatus(user_id)
        -determinePhase(friendship_score)
        -filterPools(phase)
        -selectGreeting(phase, friendship_status)
        -selectActivities(phase, friendship_status)
    }
    
    class FriendshipStatus {
        +String user_id
        +Float friendship_score
        +String friendship_level
        +DateTime last_interaction_date
        +Integer streak_day
        +Object topic_metrics
        +String lastday_emotion
        +String last_day_follow_up_topic
        +Array dynamic_memory
    }
    
    class Phase {
        <<enumeration>>
        STRANGER
        ACQUAINTANCE
        FRIEND
        +getGreetingPool()
        +getTalkPool()
        +getGamePool()
        +getTalkActivityRatio()
    }
    
    class GreetingSelector {
        +selectGreeting(phase, status)
        -checkBirthday()
        -checkLongAbsence()
        -checkEmotion()
        -checkFollowUpTopic()
        -selectRandom()
    }
    
    class ActivitySelector {
        +selectActivities(phase, status)
        -buildCandidateList()
        -getPreferenceCandidates()
        -getExploreCandidates()
        -getEmotionCandidates()
        -getGameCandidates()
        -applyRatio()
        -applyAntiDuplication()
        -applyWeighting()
    }
    
    class CandidateList {
        +Array preference_candidates
        +Array explore_candidates
        +Array emotion_candidates
        +Array game_candidates
        +merge()
        +filter()
        +sort()
    }
    
    class DailyActivityList {
        +String greeting_id
        +Array activity_ids
        +validate()
    }
    
    SelectionService --> FriendshipStatus: uses
    SelectionService --> Phase: determines
    SelectionService --> GreetingSelector: delegates
    SelectionService --> ActivitySelector: delegates
    ActivitySelector --> CandidateList: creates
    SelectionService --> DailyActivityList: returns
    Phase --> GreetingSelector: configures
    Phase --> ActivitySelector: configures
```

---

######## Tổng Kết

Các diagram trên mô tả toàn bộ logic chọn Talk/Game-Agent đầu ngày từ nhiều góc độ:

1. **Flowchart tổng quan**: Cái nhìn toàn cảnh về 5 bước chính
2. **Flowchart xác định Phase**: Chi tiết cách phân loại người dùng
3. **Flowchart chọn Greeting**: Logic ưu tiên dựa trên điều kiện đặc biệt
4. **Flowchart tạo ứng viên**: Cách xây dựng danh sách từ nhiều nguồn
5. **Flowchart lắp ráp**: Quy trình lọc và chọn cuối cùng
6. **Sequence diagram**: Tương tác giữa các thành phần theo thời gian
7. **State diagram**: Sự chuyển đổi giữa các Phase
8. **Class diagram**: Cấu trúc dữ liệu và quan hệ giữa các class


## 6. Integration Flow và Workflow

Sự tích hợp của module này vào hệ thống lớn được thể hiện qua hai luồng chính.

### 6.1. Luồng Cập nhật Trạng thái (Status Update Flow)

Đây là luồng chạy ngầm sau mỗi tương tác của người dùng, đảm bảo dữ liệu `friendship_status` luôn được cập nhật.

1.  **Trigger:** `Conversation_End` event.
2.  **BE:** Gói `user_id` và `log`.
3.  **BE -> AI:** Gọi `POST /v1/scoring/calculate-friendship`.
4.  **AI:** Xử lý và tính toán điểm thay đổi.
5.  **AI -> BE:** Gọi `POST /v1/users/{user_id}/update-friendship`.
6.  **BE:** Cập nhật `friendship_status` trong **Friendship Database**.

### 6.2. Luồng Lựa chọn Hoạt động (Activity Selection Flow)

Đây là luồng được kích hoạt khi người dùng bắt đầu một phiên mới, quyết định "hôm nay Pika sẽ nói gì?"

1.  **Trigger:** `Session_Start` event (người dùng mở app).
2.  **BE:** Nhận diện `user_id`.
3.  **BE -> AI Orchestration:** Gọi `GET /v1/users/{user_id}/suggested-activities`.
4.  **AI Orchestration:**
    a. Đọc `friendship_status` từ **Friendship Database**.
    b. Thực thi **Selection Logic**.
    c. Trả về danh sách `agent_id`.
5.  **BE:** Nhận danh sách, lấy nội dung chi tiết của các Agent từ kho và hiển thị cho người dùng, bắt đầu với Greeting Agent.

---

**Kết luận:** Tài liệu này cung cấp một kế hoạch triển khai kỹ thuật toàn diện cho module **Context Handling - Friendlyship Management**, chuyển đổi hệ thống sang mô hình cập nhật thời gian thực để tạo ra một trải nghiệm người dùng linh hoạt và cá nhân hóa hơn. Các API và cấu trúc dữ liệu được định nghĩa rõ ràng để đảm bảo sự phối hợp nhịp nhàng giữa Backend, AI và Database. 

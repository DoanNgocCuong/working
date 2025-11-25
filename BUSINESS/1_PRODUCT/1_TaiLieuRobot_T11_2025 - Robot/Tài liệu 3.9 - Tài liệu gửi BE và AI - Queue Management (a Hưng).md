1. Khi kết thúc cuộc hội thoại, phía BE bắn cho AI 1 payload để thông báo : user_id đã hoàn tất conversation_id. 

AI 6h-24h sau sẽ call tới API : user_id, conversation_id để lấy thông tin: user_id, conversation_log, memory (nếu có)

```
# Get data dựa vào conversation_id và user_id
curl -X GET http://localhost:8000/

Đảm bảo log trả đủ: 
- conversation, 
- learning_type:workflow/greeting / learn agent / talk agent / game gent (trả rõ type)
- total_time: thời gian trẻ nói
  
API format thì tuỳ ý ạ, anh Việt gửi em format example sớm nhá ạ. 

```

2. Trong mỗi 6h, phía AI tự động chạy cron-job để update score và update bài học và đánh trạng thái là đã update. 

Đêm đến phía a Hưng - AI (Queue Management) chỉ cần gọi suggest để lấy danh sách bài Agent: Greeting Agent, Talk Agent, Game/Activity Agent . 

```
curl -X POST http://localhost:8000/v1/activities/suggest \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user_123"
  }'
```


```
{
  "user_id": "user_123",
  "friendship_level": "ACQUAINTANCE",
  "computed_at": "2025-11-25T18:30:00Z",
  "greeting_agent": {
    "id": 8,
    "friendship_level": "ACQUAINTANCE",
    "agent_type": "GREETING",
    "agent_id": "greeting_memory_recall",
    "agent_name": "Memory Recall",
    "agent_description": "Nhắc lại ký ức chung với user",
    "weight": 2.0,
    "is_active": true,
    "reason": "High affinity with past memories"
  },
  "talk_agents": [
    {
      "id": 10,
      "friendship_level": "ACQUAINTANCE",
      "agent_type": "TALK",
      "agent_id": "talk_movie_preference",
      "agent_name": "Movie Preference",
      "agent_description": "Nói về phim yêu thích",
      "weight": 1.2,
      "is_active": true,
      "reason": "Highest topic_score (59.0)",
      "topic_score": 59.0,
      "total_turns": 12
    },
    {
      "id": 11,
      "friendship_level": "ACQUAINTANCE",
      "agent_type": "TALK",
      "agent_id": "talk_dreams",
      "agent_name": "Dreams Talk",
      "agent_description": "Nói về ước mơ",
      "weight": 1.0,
      "is_active": true,
      "reason": "Exploration candidate - low interaction",
      "topic_score": 8.5,
      "total_turns": 2
    }
  ],
  "game_agents": [
    {
      "id": 12,
      "friendship_level": "ACQUAINTANCE",
      "agent_type": "GAME_ACTIVITY",
      "agent_id": "game_20questions",
      "agent_name": "20 Questions",
      "agent_description": "Trò chơi 20 câu hỏi",
      "weight": 1.0,
      "is_active": true,
      "reason": "Engagement booster"
    },
    {
      "id": 13,
      "friendship_level": "ACQUAINTANCE",
      "agent_type": "GAME_ACTIVITY",
      "agent_id": "game_story_building",
      "agent_name": "Story Building",
      "agent_description": "Xây dựng câu chuyện chung",
      "weight": 1.5,
      "is_active": true,
      "reason": "Creative engagement"
    }
  ]
}
```
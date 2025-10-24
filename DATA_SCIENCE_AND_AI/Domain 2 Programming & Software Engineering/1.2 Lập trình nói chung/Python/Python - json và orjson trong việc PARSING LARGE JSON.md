# 1. Vấn đề: tại sao mất tới 0.7s chỉ để load cái conversation - get session 0.4s ???? thế 0.3 s còn lại ở đâu ?

```

robot-v2.load-conversation: 0.75s
├── robot-v2.redis.session.get: 0.41s (55%)
└── Other operations: 0.34s (45%) ← VẤN ĐỀ Ở ĐÂY!
```

## 🔍 **PHÂN TÍCH PERFORMANCE BOTTLENECK - 0.7s Load Conversation**

Dựa vào trace performance, tôi thấy có vấn đề nghiêm trọng về performance! Hãy phân tích chi tiết:

### 📊 **1. Performance Breakdown**

```
robot-v2.load-conversation: 0.75s
├── robot-v2.redis.session.get: 0.41s (55%)
└── Other operations: 0.34s (45%) ← ĐÂY LÀ VẤN ĐỀ!
```

**0.34s còn lại là gì?** Đây là thời gian xử lý trong `_load_conversation()` method!

### 🔍 **2. Phân tích code `_load_conversation`**

[3 tools called]

Bây giờ hãy xem `session_store.get_session()`:

[1 tool called]

Bây giờ hãy xem `AsyncRedisAdapter`:

[1 tool called]

## 🚨 **TÌM THẤY VẤN ĐỀ! Performance Bottleneck Analysis**

### 📊 **1. Performance Breakdown Chi Tiết**

```
robot-v2.load-conversation: 0.75s
├── robot-v2.redis.session.get: 0.41s (55%)
└── Other operations: 0.34s (45%) ← VẤN ĐỀ Ở ĐÂY!
```

### 🔍 **2. Phân Tích 0.34s Còn Lại**

**0.34s này bao gồm:**

#### **2.1 JSON Parsing Overhead:**

```python
# Trong get_session()
value = await adapter.get(_session_key(conversation_id))  # 0.41s
if not value: 
    return None 
try: 
    return json.loads(value)  # ← 0.34s CHỦ YẾU Ở ĐÂY!
except json.JSONDecodeError:
    # Error handling
```

#### **2.2 Session Data Size:**

- **Conversation session** có thể rất lớn (scenario, history, record, etc.)
- **JSON parsing** của data lớn rất chậm
- **Memory allocation** cho dict lớn

### 🎯 **3. Root Causes**

#### **3.1 Large Session Data:**

```python
# Session structure (rất lớn)
{
    "conversation_id": "...",
    "bot_id": 123,
    "user_id": "...",
    "input_slots": {...},
    "bot_config": {
        "scenario": [...],  # ← Có thể rất lớn
        "system_prompt": "...",
        "generation_params": {...}
    },
    "record": {
        "LOOP_COUNT": [{} for _ in range(len(scenario))],  # ← Array lớn
        "HISTORY_QUESTION": [...],
        "TOOL_SCORE": {...}
    },
    "history": [...]  # ← Chat history có thể rất dài
}
```

#### **3.2 JSON Parsing Bottleneck:**

```python
# json.loads() với data lớn
return json.loads(value)  # ← 0.34s cho parsing
```

### **Kết luận: 0.34s còn lại chủ yếu là JSON parsing overhead với session data lớn. Cần optimize JSON pars**

@v2_robot_workflow/ Check xem code cũ này đang lưu Redis theo cách nào??
Code mới **@robot_v2_services.py**  đang gặp vấn đề redis bị lâu và parser JSON cũng lâu như đã nói bên trên


# 2. Thử giải pháp 1: orjson, cùng 1 conversation_id cùng 1 redis và khác đầu API, 1 đầu API là json, 1 đầu API orjson


![](image/Pasted%20image%2020251024145141.png)

![](image/Pasted%20image%2020251024145150.png)
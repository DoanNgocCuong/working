## 1. Cách để log ra object thay vì history dài:


```python
logger.info(f"[RUN_ROBOT_WORKFLOW] DEBUG - State: {state}")
```

Thành:

```python
logger.info(f"[RUN_ROBOT_WORKFLOW] DEBUG - State keys: {list(state.__dict__.keys()) if hasattr(state, '__dict__') else 'No __dict__'}")
```

Bây giờ thay vì log toàn bộ state object (có thể chứa conversation history rất dài), nó sẽ chỉ log ra danh sách các keys của state, giúp bạn debug mà không bị spam log với nội dung conversation siêu dài.

Log mới sẽ hiển thị dạng như: `[RUN_ROBOT_WORKFLOW] DEBUG - State keys: ['conversation_id', 'user_input', 'messages', 'record', 'scenario', ...]`
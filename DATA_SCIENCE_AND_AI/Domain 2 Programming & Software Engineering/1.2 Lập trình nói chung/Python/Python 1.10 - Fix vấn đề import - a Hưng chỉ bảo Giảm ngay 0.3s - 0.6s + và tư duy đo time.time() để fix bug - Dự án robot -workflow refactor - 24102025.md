# Fix vấn đề import - a Hưng chỉ bảo: Giảm ngay 0.3s - 0.6s + và tư duy đo time.time() để fix bug 


## Trước fix: 

v2_session_store.py
```python
import orjson 
from typing import Any, Optional
from app.common.redis.adapter import AsyncRedisAdapter
from app.common.log import setup_logger
from langfuse import observe
```

```python
from app.common.redis import v2_session_store as session_store

    @observe(name="robot-v2.load-conversation", capture_input=False, capture_output=False)
    async def _load_conversation(self, conversation_id: str) -> Optional[Dict]:
        """
        Tải thông tin hội thoại từ Redis dựa vào conversation_id.

        Args:
            conversation_id (str): Id của hội thoại cần truy vấn.

        Returns:
            Dict hoặc None: object chứa thông tin hội thoại (nếu tìm thấy), ngược lại là None.
        """
        start_time = time.time()
        redis_client = self.redis
        logging.info(f"DEBUG - RobotV2Service._load_conversation - Time taken to get redis: {time.time() - start_time} seconds")
        start_time = time.time()
        session = await session_store.get_session(redis_client, conversation_id)
        logging.info(f"DEBUG - RobotV2Service._load_conversation - Time taken to get session: {time.time() - start_time} seconds")
        return session
```

2025-10-24 16:07:39,200 - root - INFO - DEBUG - RobotV2Service._load_conversation - Time taken to get redis: 0.0 seconds
2025-10-24 16:07:39,774 - root - INFO - DEBUG - RobotV2Service._load_conversation - Time taken to get session: 0.5731911659240723 seconds
2025-10-24 16:07:40,350 - root - INFO - DEBUG - RobotV2Service.webhook - Time taken to load conversation: 1.1509146690368652 seconds

## Sau fix: 

```python
from app.common.redis.v2_session_store import get_session

    @observe(name="robot-v2.load-conversation", capture_input=False, capture_output=False)
    async def _load_conversation(self, conversation_id: str) -> Optional[Dict]:
        """
        Tải thông tin hội thoại từ Redis dựa vào conversation_id.

        Args:
            conversation_id (str): Id của hội thoại cần truy vấn.

        Returns:
            Dict hoặc None: object chứa thông tin hội thoại (nếu tìm thấy), ngược lại là None.
        """
        start_time = time.time()
        redis_client = self.redis
        logging.info(f"DEBUG - RobotV2Service._load_conversation - Time taken to get redis: {time.time() - start_time} seconds")
        start_time = time.time()
        session = await get_session(redis_client, conversation_id)
        logging.info(f"DEBUG - RobotV2Service._load_conversation - Time taken to get session: {time.time() - start_time} seconds")
        return session
```
=> 

2025-10-24 16:12:01,185 - root - INFO - DEBUG - RobotV2Service._load_conversation - Time taken to get redis: 0.0 seconds
2025-10-24 16:12:01,707 - root - INFO - DEBUG - RobotV2Service._load_conversation - Time taken to get session: 0.5215826034545898 seconds
2025-10-24 16:12:01,708 - root - INFO - DEBUG - RobotV2Service.webhook - Time taken to load conversation: 0.5256209373474121 seconds
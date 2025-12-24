 "[PRODUCTION STAGE: OPTIMIZE LANGFUSE: USE: capture_input=False, capture_output=False, nếu log thì dùng metadata log thay vì log full input hoặc output
>> REDIS: 0.3-0.7s ?? (giảm xuống 0.01-0.03s) chủ yếu do: Langfuse trace với capture_input=False, capture_output=False. Bên cạnh đó là do dùng orjson + tối ưu: cache riêng scenario và ko cần cache history (do nó dùng CUR_ACTION chứ có dùng History để gen intent llms méo đâu)"

---
```
 "[Small Update]: Thêm bot_id và conversation_id vào metadata của observe trace (trace trên Langfuse dùng metadata)
>> --
>> Chiến lược: thay vì để capture_in và capture_out True và load data nhiều 
>> => Thì chỉ log rất ít vào metadata

```

1. https://github.com/IsProjectX/robot-lesson-workflow/commit/07741e2eec5e59c6c67bca4dfae2bc04104bcdf6
2. 
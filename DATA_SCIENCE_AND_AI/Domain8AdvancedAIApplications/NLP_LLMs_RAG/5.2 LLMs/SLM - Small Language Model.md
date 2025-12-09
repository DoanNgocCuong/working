https://huggingface.co/QuantFactory/SmolLM2-135M-GGUF

HuggingFaceTB/SmolLM2-135M-Instruct (đã test 50-170ms)
Ngang với: Qwen/Qwen2.5-0.5B-Instruct-AWQ


| Model                                                                                                       | Size | Quantization | Model Size | Context    | Link Download                                                                                                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ----------------------------------------------------------------------------------------------------------- | ---- | ------------ | ---------- | ---------- | --------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Qwen3-0.6B](https://github.com/QwenLM/Qwen3)                                                               | 0.6B | Base FP16    | ~1.2GB     | 32K tokens | [GitHub Qwen3](https://github.com/QwenLM/Qwen3)                                                                 | - [16 query heads và 8 key/value heads (GQA)](https://skywork.ai/blog/models/qwen3-0-6b-fp8-free-chat-online-skywork-ai/) - giảm 50% KV cache memory<br>    <br>- 32,768 token context window natively<br>    <br>- Hỗ trợ **thinking mode** và **non-thinking mode** trong cùng một model<br>    <br>- [Grouped Query Attention architecture](https://skywork.ai/blog/models/qwen3-0-6b-fp8-free-chat-online-skywork-ai/) tối ưu cho inference speed |
| [Qwen3-0.6B-FP8](https://skywork.ai/blog/models/qwen3-0-6b-fp8-free-chat-online-skywork-ai/)                | 0.6B | FP8          | ~600MB     | 32K tokens | [Skywork AI](https://skywork.ai/blog/models/qwen3-0-6b-fp8-free-chat-online-skywork-ai/)                        |                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Qwen3-0.6B-GGUF Q4_K_M                                                                                      | 0.6B | GGUF INT4    | ~180-220MB | 32K tokens | [Ollama](https://ollama.com/library/qwen3) (via ollama pull qwen3:0.6b)                                         |                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| [Qwen3-ViT-0.5B](https://huggingface.co/models?other=base_model%3Aquantized%3AAname-Tommy%2FQwen3-ViT-0.5B) | 0.5B | Various      | ~150-300MB | -          | [HuggingFace Search](https://huggingface.co/models?other=base_model%3Aquantized%3AAname-Tommy%2FQwen3-ViT-0.5B) |                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

| Model                                                                                    | Size | Quantization    | Latency | Accuracy           | MMLU Score | Link                                                                        |
| ---------------------------------------------------------------------------------------- | ---- | --------------- | ------- | ------------------ | ---------- | --------------------------------------------------------------------------- |
| [Qwen2.5-1.5B-Instruct-AWQ](https://huggingface.co/Qwen/Qwen2.5-1.5B-Instruct-AWQ)       | 1.5B | AWQ INT4        | 30-40ms | **+5-7% vs 0.5B**  | 60%        | [HF Link](https://huggingface.co/Qwen/Qwen2.5-1.5B-Instruct-AWQ)            |
| [Phi-3.5-mini-INT4-ONNX](https://huggingface.co/nvidia/Phi-3.5-mini-Instruct-ONNX-INT4)  | 3.8B | INT4 AWQ (ONNX) | 40-50ms | **+8-10% vs 0.5B** | 69%        | [NVIDIA](https://huggingface.co/nvidia/Phi-3.5-mini-Instruct-ONNX-INT4)     |
| [Gemma-2-2B-IT-AWQ-INT4](https://huggingface.co/Esperanto/gemma-2b-it-kvc-AWQ-int4-onnx) | 2B   | AWQ INT4 (ONNX) | 35-45ms | **+6-8% vs 0.5B**  | 56%        | [Esperanto](https://huggingface.co/Esperanto/gemma-2b-it-kvc-AWQ-int4-onnx) |

---

 # model: "Qwen/Qwen2.5-0.5B-Instruct-GPTQ-Int4"



```
Accuracy Ranking:
1. GGUF (best accuracy)
2. GPTQ
3. AWQ

Speed Ranking:
1. AWQ (fastest) ✅
2. GPTQ
3. GGUF (slowest)
```

---
```
CUDA_VISIBLE_DEVICES=0 python -m vllm.entrypoints.openai.api_server \
    --model 'Qwen/Qwen2.5-0.5B-Instruct-AWQ' \
    --host 0.0.0.0 \
    --port 30030 \
    --quantization awq \
    --dtype half \
    --gpu-memory-utilization 0.3 \
    --max-model-len 512 \
    --max-num-seqs 16 \
    --max-num-batched-tokens 512 \
    --enable-prefix-caching \
    --enable-chunked-prefill \
    --swap-space 4 \
    --trust-remote-code \
    --disable-log-requests


Esperanto/gemma-2b-it-kvc-AWQ-int4-onnx
```


# SO sánh Qwen 0.5B với 1.5B 
response time

curl --location 'http://103.253.20.30:30030/v1/chat/completions' \
--header 'Content-Type: application/json' \
--data '{
    "model": "Qwen/Qwen2.5-1.5B-Instruct-AWQ",
    "messages": [
      {
        "role": "system",
        "content": "CLASSIFY: '\''yes'\'' if Pika'\''s response confirms a correct factual answer. Otherwise, '\''no'\''. Output ONLY '\''yes'\'' or '\''no'\''."
      },
      {
        "role": "user", 
        "content": "Previous Question: Fantastic! Cụm từ tiếp theo cậu nha. Khi chào tạm biệt bạn bè, tớ hay dùng cụm này nè. Bắt đầu chậm trước nha See you soon \n Previous Answer: đúng rùi \n Previous Answer: See you soon \n Pika'\''s response need check: I think this lesson is interesting"
      }
    ],
    "temperature": 0,
    "max_tokens": 1, 
    "stop": ["\n", " ", ".", ","]
  }'
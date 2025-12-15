<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Deep Research: Giải pháp Memory cho AI Agents (MECE Framework)

Báo cáo này cung cấp một nghiên cứu sâu (Deep Research) về hệ thống bộ nhớ (Memory) cho AI Agents, được cấu trúc theo nguyên tắc MECE (Mutually Exclusive, Collectively Exhaustive - Không trùng lặp, Không bỏ sót).

Nội dung tập trung vào các kiến trúc SOTA (State-of-the-Art) giai đoạn cuối 2024 - 2025, phù hợp với góc nhìn kỹ thuật của một AI Engineer (MLOps/System Design).

***

### Executive Summary: The Shift to "Agentic Memory"

Trước 2024, bộ nhớ AI chủ yếu là **RAG tĩnh** (Read-Only). Đến 2025, xu hướng chuyển dịch mạnh mẽ sang **Agentic Memory** (Read-Write-Reflect). Sự thay đổi cốt lõi là Agent không chỉ *truy xuất* mà còn *chủ động quản lý* bộ nhớ của mình: tự quyết định cái gì cần nhớ (Commit), cái gì cần quên (Prune), và cái gì cần học thành kỹ năng (Proceduralization).

***

### PHẦN 1: MECE FRAMEWORK - KIẾN TRÚC BỘ NHỚ TOÀN DIỆN

Chúng ta sẽ phân loại giải pháp theo **Chức năng Nhận thức (Cognitive Function)**, tương ứng với các lớp trong kiến trúc phần mềm.

#### 1. Working Memory (Bộ nhớ Làm việc - Short-term)

Đây là "RAM" của Agent, xử lý ngữ cảnh tức thời trong phiên làm việc hiện tại.

* **Cơ chế:** Quản lý Context Window của LLM.
* **Giải pháp Kỹ thuật:**
    * **Sliding Window \& Token Buffering:** Giữ $N$ token gần nhất, cắt bỏ phần cũ.
    * **Summary-based Buffer:** Dùng một LLM phụ để tóm tắt hội thoại cũ và chèn vào đầu context (recursive summarization).
    * **Attention Sinks (StreamingLLM):** Giữ lại các token "sink" (thường là token đầu tiên) để duy trì sự ổn định của attention map ngay cả khi context trôi đi, cho phép hội thoại vô tận mà không cần fine-tuning lại.
    * **Managed State (LangGraph):** Lưu trữ trạng thái biến (variables state) thay vì raw text. Ví dụ: `current_step: "analyzing_financial_report"`.


#### 2. Episodic Memory (Bộ nhớ Sự kiện - Long-term Explicit)

Đây là "Hard Drive" chứa nhật ký hoạt động và trải nghiệm quá khứ. Agent nhớ lại "mình đã làm gì" và "kết quả ra sao".

* **Cơ chế:** Vector Search (Semantic) + Time-series.
* **Giải pháp Kỹ thuật:**
    * **Vector RAG:** Lưu trữ logs/chats dưới dạng vector embeddings (OpenAI, VoyageAI) trong Vector DB (Pinecone, Milvus, Qdrant).
    * **Clustering \& Community Detection:** Gom nhóm các sự kiện tương đồng để Agent hiểu patterns (ví dụ: "Người dùng này thường hỏi về stock vào thứ 2").
    * **Hierarchical Retrieval (MemGPT):** Phân trang bộ nhớ. Chỉ đưa vào context những "trang" bộ nhớ liên quan nhất thay vì toàn bộ lịch sử.


#### 3. Semantic Memory (Bộ nhớ Ngữ nghĩa - World Model)

Đây là "Knowledge Graph" chứa sự thật (facts), mối quan hệ và kiến thức về thế giới/domain.

* **Cơ chế:** Knowledge Graph (KG) hoặc Hybrid (Graph + Vector).
* **Giải pháp Kỹ thuật:**
    * **GraphRAG (Microsoft):** Trích xuất thực thể (Entities) và mối quan hệ (Relationships) từ dữ liệu văn bản, tạo thành đồ thị. Giúp Agent trả lời các câu hỏi tổng hợp (global questions) tốt hơn RAG thường.[^1][^2]
    * **Dynamic Knowledge Graphs (Zep/Mem0):** Khác với GraphRAG tĩnh, các hệ thống này cho phép Agent *update* graph real-time (ví dụ: User đổi ý định -> Update node `User_Preference` từ "Conservative" sang "Aggressive").[^3][^4]
    * **Triple Store:** Lưu trữ dạng (Subject, Predicate, Object) trong Neo4j hoặc NetworkX.


#### 4. Procedural Memory (Bộ nhớ Thủ tục - Skills \& Tools)

Đây là "Skill Library" chứa kỹ năng, code, và quy trình thực hiện task. Đây là phần quan trọng nhất để Agent tự động hóa (Automation).

* **Cơ chế:** Code-as-Memory, Dynamic Tool Loading.
* **Giải pháp Kỹ thuật:**
    * **Skill Library (Voyager Style):** Lưu trữ các hàm Python (executable code) đã thành công vào Vector DB. Khi gặp task tương tự, Agent *retrieve* code cũ và chạy lại thay vì viết mới từ đầu.[^5][^6]
    * **Dynamic Tool Selection (Memp/Adaptive Minds):** Thay vì load tất cả 100 tools vào context (gây nhiễu), Agent dùng vector search để tìm top-5 tools phù hợp nhất với task hiện tại và chỉ load định nghĩa của chúng.[^7][^8]
    * **Prompt/Chain Caching:** Lưu trữ các chuỗi suy luận (Chain-of-Thought) đã thành công để tái sử dụng (Memoization).


#### 5. Parametric Memory (Bộ nhớ Ẩn - Implicit)

Kiến thức được "nướng" (baked) vào trong trọng số mô hình.

* **Cơ chế:** Fine-tuning, Model Merging.
* **Giải pháp Kỹ thuật:**
    * **LoRA-as-Tools (Adaptive Minds):** Thay vì RAG, Agent chọn một LoRA Adapter chuyên biệt (ví dụ: "Finance LoRA") và *swap* nó vào base model tại runtime để xử lý task chuyên sâu.[^8][^9]
    * **Continual Learning (Replay Buffers):** Fine-tune nhẹ liên tục trên dữ liệu mới mà không bị catastrophic forgetting (sử dụng kỹ thuật Replay buffer).

***

### PHẦN 2: SO SÁNH CÁC GIẢI PHÁP MEMORY "OS" HÀNG ĐẦU (2025)

Hiện nay đã xuất hiện các "Memory OS" đóng gói các lớp trên thành giải pháp trọn gói. Dưới đây là bảng so sánh kỹ thuật:


| Tiêu chí | **Mem0 (Mem0g)** [^10][^3] | **Zep** [^4][^2] | **GraphRAG (Microsoft)** [^1] | **LangGraph Checkpointer** |
| :-- | :-- | :-- | :-- | :-- |
| **Loại hình** | Graph-based Memory Layer | Long-term Memory Service | Static Retrieval Engine | State Persistence |
| **Cơ chế Update** | **Read-Write**: 2-Phase (Extraction -> LLM quyết định Add/Update/Delete) | **Real-time**: Stream dữ liệu vào, tự động update graph bất đồng bộ. | **Batch-Only**: Phải chạy index lại toàn bộ dataset để update graph. | **Snapshot**: Lưu trạng thái tại từng bước (Step). |
| **Độ trễ (Latency)** | Thấp (Optimized for Agent) | Rất thấp (Sub-second retrieval) | Cao (Phù hợp offline analysis) | N/A (Internal State) |
| **Cấu trúc dữ liệu** | Hybrid (Vector + Graph Relations) | Temporal Knowledge Graph (Graph + Time) | Hierarchical Graph Communities | Key-Value State Schema |
| **Use Case tối ưu** | Personal Assistant, User Profiling | Chatbot quy mô lớn, Enterprise Data | Tóm tắt tài liệu lớn, Global Analysis | Luồng Agent phức tạp (Multi-step) |
| **Cost** | Rẻ hơn 90% token so với full context | Cao hơn (Managed Service) | Cao (Chi phí build graph lớn) | Rất thấp |

**Deep Dive: Cơ chế "Write" của Mem0 (Sự khác biệt lớn nhất)**[^3]
Mem0 không chỉ ném dữ liệu vào Vector DB. Nó thực hiện quy trình **CRUD thông minh**:

1. **Extraction:** Lấy hội thoại mới.
2. **Comparison:** So sánh với Memory cũ.
3. **Decision:** LLM quyết định hành động:
    * `ADD`: Thông tin mới hoàn toàn -> Tạo node.
    * `UPDATE`: Thông tin thay đổi (VD: Sở thích đổi) -> Sửa node cũ.
    * `DELETE`: Thông tin sai/cũ -> Xóa.
    * `NOOP`: Thông tin trùng lặp -> Bỏ qua.

***

### PHẦN 3: KIẾN NGHỊ TRIỂN KHAI (DÀNH CHO AI ENGINEER)

Dựa trên profile của bạn (Fintech, System Design), đây là kiến trúc khuyến nghị để xây dựng "Financial Analyst Agent":

#### 1. Tech Stack

* **Orchestration:** LangGraph (quản lý luồng, state).
* **Working Memory:** LangGraph State (quản lý biến số dư tài khoản, mã stock đang check).
* **Semantic Memory (Financial Data):**
    * Dùng **Zep** hoặc **Mem0** để lưu User Profile (khẩu vị rủi ro, lịch sử đầu tư).
    * Dùng **GraphRAG** (chạy batch hàng đêm) trên các báo cáo tài chính (PDF) để Agent có thể trả lời các câu hỏi vĩ mô ("So sánh chiến lược vốn của 5 ngân hàng lớn nhất").
* **Procedural Memory (Tools):**
    * Lưu các Python function (DCF Calculation, Technical Analysis) vào Vector DB (Qdrant/Pinecone).
    * Dùng cơ chế **Dynamic Tool Loading**: Khi user hỏi "Định giá VCB", hệ thống query vector DB tìm tool `dcf_valuation.py` và load vào context.


#### 2. Workflow mẫu (Pythonic Pseudocode)

```python
# Ví dụ triển khai Procedural Memory đơn giản
from langchain.vectorstores import Qdrant
from langchain.agents import Tool

class AgentBrain:
    def __init__(self):
        self.skill_library = Qdrant.from_documents(all_tools_docs) # Procedural Memory
        self.user_memory = Mem0() # Semantic/Episodic Memory

    def process_query(self, user_id, query):
        # 1. Retrieve Context (Semantic)
        user_profile = self.user_memory.search(query, user_id=user_id)
        
        # 2. Retrieve Skills (Procedural) - Dynamic Tool Loading
        relevant_tools_docs = self.skill_library.similarity_search(query, k=3)
        active_tools = [load_tool_from_doc(doc) for doc in relevant_tools_docs]
        
        # 3. Parametric (Optional) - Load Expert Adapter
        if "financial report" in query:
             model.load_adapter("finance_lora") # Parametric Memory switch

        # 4. Execute
        agent = create_agent(model, tools=active_tools, context=user_profile)
        return agent.run(query)
```


### Kết luận

Để đạt được sự tự do tâm trí và xây dựng hệ thống bền vững (X10), đừng chỉ dựa vào Context Window (ngày càng rẻ nhưng không nhớ lâu). Hãy đầu tư xây dựng **Lớp Bộ nhớ (Memory Layer)** tách biệt.

* Bắt đầu với **Mem0** (Open source, dễ integrate) cho User Memory.
* Dùng **Vector DB** chuẩn cho Skill Library.
* Dùng **GraphRAG** cho việc phân tích dữ liệu tài chính phức tạp.
<span style="display:none">[^11][^12][^13][^14][^15][^16][^17][^18][^19][^20][^21][^22][^23][^24][^25][^26][^27][^28][^29][^30][^31][^32][^33][^34][^35][^36][^37][^38][^39][^40][^41][^42][^43][^44]</span>

<div align="center">⁂</div>

[^1]: https://towardsdatascience.com/graphrag-in-action-from-commercial-contracts-to-a-dynamic-q-a-agent-7d4a6caa6eb5/

[^2]: https://help.getzep.com/docs/building-searchable-graphs/zep-vs-graph-rag

[^3]: https://memo.d.foundation/breakdown/mem0

[^4]: https://www.getzep.com/mem0-alternative/

[^5]: https://voyager.minedojo.org

[^6]: https://arxiv.org/abs/2305.16291

[^7]: https://sparkco.ai/blog/enterprise-guide-to-dynamic-tool-loading-agents

[^8]: https://arxiv.org/html/2510.15416v1

[^9]: https://www.rohan-paul.com/p/dynamic-and-continual-learning-in

[^10]: https://github.com/Decentralised-AI/mem0-Memory-for-AI-Agents-SOTA-in-AI-Agent-Memory

[^11]: https://fpt-is.com/goc-nhin-so/deep-research-la-gi/

[^12]: https://www.facebook.com/groups/miaigroup/posts/2016092149162047/

[^13]: https://viblo.asia/p/mem0-kien-truc-long-term-memory-cho-he-thong-ai-agent-G24B88pOLz3

[^14]: https://techdata.ai/ai-agent-memory-trien-khai-nhu-the-nao-cho-dung/

[^15]: https://tuyendung.evotek.vn/su-dung-co-so-du-lieu-vector-va-bo-nho-theo-giai-doan-trong-ai-agent/

[^16]: https://artium.ai/insights/memory-in-multi-agent-systems-technical-implementations

[^17]: https://natesnewsletter.substack.com/p/the-definitive-guide-to-ai-agents

[^18]: https://vneconomy.vn/techconnect/review-tinh-nang-deep-research-cua-openai-giua-con-sot-deepseek.htm

[^19]: https://www.linkedin.com/posts/ivan-nardini_aimemory-llms-agents-activity-7326725381309227008-9cU1

[^20]: https://collabnix.com/multi-agent-and-multi-llm-architecture-complete-guide-for-2025/

[^21]: https://arxiv.org/pdf/2502.12110.pdf

[^22]: https://www.ultralytics.com/vi/blog/the-role-of-deep-research-models-in-ai-advancements

[^23]: https://aclanthology.org/2025.emnlp-main.1318.pdf

[^24]: https://www.lindy.ai/blog/ai-agent-architecture

[^25]: https://arxiv.org/html/2502.12110v11

[^26]: https://fukai.news/p/fukai-01-deep-research-ai-agent-bat-dau-huu-ich

[^27]: https://arxiv.org/abs/2505.00675

[^28]: https://www.labellerr.com/blog/what-are-ai-agents-a-comprehensive-guide/

[^29]: https://openreview.net/forum?id=FiM0M8gcct

[^30]: https://dev.to/bredmond1019/building-intelligent-ai-agents-with-memory-a-complete-guide-5gnk

[^31]: https://blog.promptlayer.com/top-skills-to-build-ai-agents-in-2025/

[^32]: https://docs.langchain.com/oss/python/langgraph/memory

[^33]: https://qdrant.tech/articles/agentic-builders-guide/

[^34]: https://botpress.com/blog/skills-to-build-ai-agent

[^35]: https://www.linkedin.com/posts/kumaran-ponnambalam-961a344_m-e-m-p-memp-exploring-agent-procedural-activity-7363938826504818689-zj2b

[^36]: https://arxiv.org/html/2412.17944v1

[^37]: https://fleek.xyz/blog/learn/ai-agent-skills/

[^38]: https://www.reddit.com/r/MachineLearning/comments/13sc0pp/voyager_an_llmpowered_learning_agent_in_minecraft/

[^39]: https://github.com/peytontolbert/DynamicAgent

[^40]: https://dev.to/yigit-konur/mem0-the-comprehensive-guide-to-building-ai-with-persistent-memory-fbm

[^41]: https://memgraph.com/blog/build-agentic-graphrag-ai

[^42]: https://arxiv.org/html/2504.19413v1

[^43]: https://www.gocodeo.com/post/continuous-learning-in-agentic-ai-pipelines-for-adaptation

[^44]: https://arxiv.org/pdf/2504.19413.pdf


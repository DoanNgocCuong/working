```bash
“I want to perform a MECE-based codebase audit and architecture reconstruction to understand the entire system from high-level structure down to detailed execution flows.”
```


```bash
You are a senior software architect. 
Your task is to quickly understand a new GitHub repository from high-level to detailed view.

Analyze this repository using a MECE-based, top-down approach:
1. High-Level Overview  
   - What is the purpose of the project?
   - What problems does it solve?
   - What are the main modules?

2. MECE Module Decomposition  
   - Identify key directories and group them into MECE categories 
     (e.g., Core Logic, API Layer, Data Layer, Agent/AI, Utilities).
   - Explain each group with purpose + responsibilities.

3. Architecture Reconstruction  
   - Generate a High-Level Architecture diagram description.
   - Generate a Component Diagram (main components + interactions).
   - Identify external APIs, services, databases, libraries.

4. Execution Flow Mapping  
   - Describe the main flows (e.g., request → processing → output).
   - Provide Sequence Diagrams for the most important use cases.
   - Explain how data moves through the system.

5. Hotspot Identification  
   - Locate the most important files/classes/functions.
   - Summarize their roles, inputs, outputs.
   - Identify any logic hubs or orchestration entry points.

6. Code Quality & Structure Assessment  
   - Evaluate clarity, modularity, naming, separation of concerns.
   - Identify risks, smells, or complexity hotspots.
   - Suggest improvements.

7. Deliver a concise "15-Minute Understanding Summary"  
   - A compressed explanation that lets a new engineer 
     understand the entire system quickly.

Repository to analyze:
[INSERT GITHUB URL HERE]

Follow a top-down approach, avoid overwhelming details at first.
Explain clearly, professionally, and structurally.

```


---

# 🔥 Tại sao prompt này là “best practice”?

Vì nó:

- Áp dụng **MECE** → không sót, không trùng
    
- Dùng **top-down reading** → hiểu nhanh từ tổng quan → chi tiết
    
- Bắt buộc AI tạo **architecture diagrams**, **flow diagrams**, **module map**
    
- Buộc AI tìm **hotspots**, **core logic**, **orchestration entry point**
    
- Cho bạn phần **summary 15 phút** để onboard team
    
- Dành cho cả dự án AI/agent/microservice/web backend
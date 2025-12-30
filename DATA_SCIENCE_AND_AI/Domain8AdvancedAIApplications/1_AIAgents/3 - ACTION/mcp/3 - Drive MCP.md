

|MCP Server|Maintainer|Read|Write/Update|Auto-convert Docs/Sheets|Vị thế|Phù hợp cho bạn|
|---|---|---|---|---|---|---|
|**Anthropic Google Drive MCP (reference)**[](https://www.pulsemcp.com/servers/gdrive)​|Anthropic|✅|❌ (chủ yếu read-only)|✅ (Docs/Sheets → text/CSV)[](https://playbooks.com/mcp/modelcontextprotocol-gdrive)​|Reference, rất phổ biến[](https://modelcontextprotocol.io/faqs)​|Safe default, đọc/tóm tắt tài liệu fin, research|
|**`mcp-gdrive` (community full-feature)**[](https://skywork.ai/skypage/en/The-Ultimate-Guide-to-the-mcp-gdrive-MCP-Server-for-AI-Engineers/1971371440002363392)​|Community (evolved from Anthropic)|✅|✅ (nhiều bản hỗ trợ Sheets write)[](https://skywork.ai/skypage/en/The-Ultimate-Guide-to-the-mcp-gdrive-MCP-Server-for-AI-Engineers/1971371440002363392)​|✅ (nhấn mạnh là key feature)[](https://skywork.ai/skypage/en/The-Ultimate-Guide-to-the-mcp-gdrive-MCP-Server-for-AI-Engineers/1971371440002363392)​|SOTA về tính năng, nhiều guide chuyên sâu[](https://skywork.ai/skypage/en/The-Ultimate-Guide-to-the-mcp-gdrive-MCP-Server-for-AI-Engineers/1971371440002363392)​|Khi muốn agent đọc + ghi vào Drive/Sheets (workflow tài chính)|
|**`gdrive-mcp-server` (felores)**[](https://github.com/felores/gdrive-mcp-server)​|felores|✅|❌ (read-focused)|✅|Được đánh giá là efficient, high‑performance read bridge[](https://skywork.ai/skypage/en/google-drive-ai-mcp-servers/1981200041361985536)​|Khi cần cầu nối đọc Drive rất ổn định, hiệu năng cao|

https://github.com/taylorwilsdon/google_workspace_mcp


## GENSPARK: ✅ MCP Server được đề xuất

**[Google Workspace MCP Server](https://github.com/taylorwilsdon/google_workspace_mcp)** của Taylor Wilsdon - Đây là giải pháp **hoàn chỉnh nhất** với:

- ✅ **OAuth 2.1 multi-user support** (chính xác là điều bạn cần!)
- ✅ Drive + Sheets + Docs + Gmail + Calendar + Forms + Tasks + Chat
- ✅ FastMCP framework với stateless HTTP mode
- ✅ Bearer token authentication
- ✅ Production-ready
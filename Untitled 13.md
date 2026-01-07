# Vấn đề ban đầu: gặp phải là 


  ```bash
# Test MCP protocol endpoint
curl -X POST http://localhost:8000/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"initialize","id":1,"params":{}}'
  ```

```
{
    "jsonrpc": "2.0",
    "id": "server-error",
    "error": {
        "code": -32600,
        "message": "Not Acceptable: Client must accept both application/json and text/event-stream"
    }
}
```

```bash

  

curl -X POST http://localhost:8000/mcp \

  -H "Content-Type: application/json" \

  -H "Accept: application/json, text/event-stream" \

  -d '{

    "jsonrpc": "2.0",

    "method": "initialize",

    "id": 1,

    "params": {

      "protocolVersion": "2024-11-05",

      "capabilities": {},

      "clientInfo": {

        "name": "test-client",

        "version": "1.0.0"

      }

    }

  }'

  

```
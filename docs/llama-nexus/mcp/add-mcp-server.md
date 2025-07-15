---
sidebar_position: 3
---

# Connect Another MCP Server

As an MCP client, **Llama-Nexus** supports connecting to multiple MCP servers. This article provides an example of how to add an additional MCP server.

## Modify `config.toml`

Assuming you have already [installed Llama-Nexus](../quick-start.md#install-the-llama-nexus-software) on your machine, you can add a new MCP server by editing the `config.toml` file:

```bash
vi config.toml
```

Add a new MCP server entry using the following format:

```
[[mcp.server.tool]]
name      = "test-search"
transport = "stream-http"
url       = "http://YOUR_IP_ADDRESS/mcp"
enable    = true
```

> Make sure enable = true to activate the MCP service.

Next, [start the Llama-Nexus service and register an LLM that supports function calls with the MCP server.](../mcp/quick-start-with-mcp.md).

Once registered successfully, you can send requests using the OpenAI-compatible API. For example:

```
curl -X POST http://localhost:3389/v1/chat/completions \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{"messages":[{"role":"system", "content": "You are a helpful assisstant.If the user asks a question, you will use the  tool call to answer user question."},{"role":"user", "content": "Send an email to vivian@gamil.com and told her the dinner is cancelled, becuase I have a business meeting."}]}'
```

If you're using an email MCP server like Gmail, the email will be sent automactailly to the designated email address.

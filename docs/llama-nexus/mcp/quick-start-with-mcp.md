---
sidebar_position: 1
---

# Quick start with MCP servers

One of the key features of Llama-Nexus is its built-in MCP Client, which allows you to use Llama-Nexus for MCP-related tasks just like Claude Desktop and Cursor.

This tutorial shows how to set up real-time weather functionality with Llama-Nexus, a Weather MCP server, and an LLM that supports tool calls. The MCP server should be running successfully before you start Llama-Nexus.

## Prerequisites

- OpenWeather API key (obtain from [openweathermap.org](https://openweathermap.org))
- macOS or Linux environment
- Network access for remote server setup (if using remote option)

## 1. Set Up Your MCP Server

```bash
curl -LO https://github.com/cardea-mcp/cardea-mcp-servers/releases/download/0.8.0/cardea-mcp-servers-unknown-linux-gnu-x86_64.tar.gz
tar xvf cardea-mcp-servers-unknown-linux-gnu-x86_64.tar.gz
```
> Download for your platform: https://github.com/cardea-mcp/cardea-mcp-servers/releases/tag/0.8.0

Set the environment variables:

```bash
export OPENWEATHERMAP_API_KEY=YOUR_KEY
export RUST_LOG=debug
export LLAMA_LOG=debug
```

Run the MCP server (accessible from external connections):

```bash
./cardea-weather-mcp-server --transport stream-http --socket-addr 0.0.0.0:8002
```

**Important**: Ensure port 8002 is open in your firewall/security group settings if you're running on a cloud machine.

## 2. Set Up the Inference Server

### Install llama-nexus

Download and extract llama-nexus:

```bash
curl -LO https://github.com/LlamaEdge/llama-nexus/releases/download/0.6.0/llama-nexus-apple-darwin-aarch64.tar.gz
tar xvf llama-nexus-apple-darwin-aarch64.tar.gz
```

> Download for your platform: https://github.com/LlamaEdge/llama-nexus/releases/tag/0.6.0

### Configure llama-nexus

Edit the `config.toml` file to specify the gateway server port:

```toml
[server]
host = "0.0.0.0" # The host to listen on
port = 9095      # The port to listen on
```

Configure the Weather MCP server connection:


```toml
[[mcp.server.tool]]
name      = "Cardea-weather"
transport = "stream-http"
url       = "http://YOUR-IP-ADDRESS:8002/mcp"
enable    = true
```
> You can configure multiple MCP servers in the `config.toml` file by adding additional `[[mcp.server.tool]]` sections.

### Start llama-nexus

```bash
nohup ./llama-nexus --config config.toml &
```

### Register Downstream API Servers

Register an LLM chat API server for the `/chat/completions` endpoint:

```bash
curl --location 'http://localhost:9095/admin/servers/register' \
--header 'Content-Type: application/json' \
--data '{
    "url": "https://0xb2962131564bc854ece7b0f7c8c9a8345847abfb.gaia.domains/v1",
    "kind": "chat"
}'
```

> If your API server requires an API key access, you can add an `api-key` field in the registration request.

## 3. Test the Setup

Test the inference server by requesting the `/chat/completions` API endpoint, which is OpenAI-compatible:

```bash
curl -X POST http://localhost:9095/v1/chat/completions \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{"messages":[{"role":"system", "content": "You are a helpful assistant. You will use the tool to solve user problems."},{"role":"user", "content": "What is the weather in Singapore?"}]}'
```

Expected output:

```json
{
  "id": "chatcmpl-cf63660e-3494-472c-b4d0-6cda72e1f8e9",
  "object": "chat.completion",
  "created": 1751602566,
  "model": "Qwen3-4B",
  "choices": [
    {
      "index": 0,
      "message": {
        "content": "The current temperature in Singapore is 30.13°C. Would you like to know the weather forecast for the next few days as well?",
        "role": "assistant"
      },
      "finish_reason": "stop",
      "logprobs": null
    }
  ],
  "usage": {
    "prompt_tokens": 62,
    "completion_tokens": 32,
    "total_tokens": 94
  }
}
```

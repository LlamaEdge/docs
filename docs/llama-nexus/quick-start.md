---
sidebar_position: 2
---

# Manage Your API Services with Llama-Nexus

This tutorial shows you how to use **Llama-Nexus** to manage multiple OpenAI-compatible API services.

## Install the Llama-Nexus software

The following command installs the Linux on x86 version of llama-nexus.

```
curl -LO https://github.com/LlamaEdge/llama-nexus/releases/download/0.5.0/llama-nexus-unknown-linux-gnu-x86_64.tar.gz

tar xvf llama-nexus-unknown-linux-gnu-x86_64.tar
```

> Download for your platfrom here: https://github.com/LlamaEdge/llama-nexus/releases/tag/0.5.0

## Start Llama-Nexus

```
./llama-nexus --config config.toml
```

By default, Llama-Nexus listens on port `3389`.

## Register a chat service

Assuming you already have an OpenAI-compatible API server for your LLM, let‘s register it with Llama-Nexus.

If you'd like to run a model locally, refer to the [Quick Start with LLM](../ai-models/llm/quick-start-llm.md) guide.

Register the LLM chat API server for the `/chat/completions` endpoint:

```bash
curl --location 'http://localhost:3389/admin/servers/register' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR_API_KEY_GOES_HERE' \
--data '{
    "url": "http://localhost:8080",
    "kind": "chat"
}'
```

To register multiple services, repeat the request with different URLs.

> For other types of models, please check the [Register](./register.md) guide

## Call the service

Since Llama-Nexus is OpenAI-compatible, you can use any OpenAI-compatible client or the API spec to call your services:

```bash
curl -X POST http://localhost:9095/v1/chat/completions \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{"messages":[{"role":"system", "content": "You are a helpful assistant."},{"role":"user", "content": "What is the weather in Singapore?"}]}'
```

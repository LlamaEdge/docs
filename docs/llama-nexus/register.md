---
sidebar_position: 3
---

# Register API services

You can add almost any OpenAI-compatible API services to the Llama-Nexus gateway.
In this chapter, we demonstrate how various LlamaEdge API servers could be registered 
under a single Llama Nexus gateway. This gateway will be able to provide all OpenAI API
endpoints supported by these registered API servers. 

## Prerequisites

- Llama Nexus server running (default port: 3389)
- Target services running and accessible
- Services implementing OpenAI-compatible APIs

## Service Registration

### Chat Service

Register a chat completion service that handles `/v1/chat/completions` requests:

```bash
curl --location 'http://localhost:3389/admin/servers/register' \
--header 'Content-Type: application/json' \
--data '{
    "url": "http://localhost:8080",
    "kind": "chat"
}'
```

### Embedding Service

Register an embedding service that handles `/v1/embeddings` requests:

```bash
curl --location 'http://localhost:3389/admin/servers/register' \
--header 'Content-Type: application/json' \
--data '{
    "url": "https://0x448f0405310a9258cd5eab5f25f15679808c5db2.gaia.domains",
    "kind": "embeddings"
}'
```

### Image Generation Service

Register an image generation service that handles `/v1/images/generations` requests:

```bash
curl --location 'http://localhost:3389/admin/servers/register' \
--header 'Content-Type: application/json' \
--data '{
    "url": "http://localhost:10010",
    "kind": "image"
}'
```

### Speech to Text Service

Register a transcription service that handles `/v1/audio/transcriptions` requests:

```bash
curl --location 'http://localhost:3389/admin/servers/register' \
--header 'Content-Type: application/json' \
--data '{
    "url": "http://localhost:10010",
    "kind": "transcribe"
}'
```

### Text to Speech Service

Register a text-to-speech service that handles `/v1/audio/speech` requests:

```bash
curl --location 'http://localhost:3389/admin/servers/register' \
--header 'Content-Type: application/json' \
--data '{
    "url": "http://localhost:10010",
    "kind": "tts"
}'
```

## Notes

- All services must implement OpenAI-compatible APIs
- URLs can be local (`http://localhost:port`) or remote (`https://domain.com`)
- Services are automatically load-balanced if multiple instances of the same kind are registered

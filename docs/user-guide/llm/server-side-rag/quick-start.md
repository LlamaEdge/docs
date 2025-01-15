---
sidebar_position: 1
---

# Long-term memory for the LLM

The LLM app requires both long-term and short-term memories. Long-term memory includes factual knowledge, historical facts, background stories etc. They are best added to the context as complete chapters instead of small chunks of text to maintain the internal consistency of the knowledge.  

[RAG](https://blogs.nvidia.com/blog/what-is-retrieval-augmented-generation/) 
is an important technique to inject contextual knowledge into an LLM application. It improves accuracy and reduces the hallucination of LLMs.
An effective RAG application combines real-time and user-specific short-term memory (chunks) with stable long-term memory (chapters) in the prompt context. 

Since the application's long-term memory is stable (even immutable), we package it in a vector database tightly coupled with the LLM. The client app assembles the short-term memory in the prompt and is supplemented with the long-term memory on the LLM server. We call the approach "server-side RAG".

> The long context length supported by modern LLMs are especially well suited for long term knowledge that are best represented by chapters of text.

The LlamaEdge API server provides application components that developers can reuse to 
supplement the LLM with long-term memories. 
We have built this feature into the [rag-api-server](https://github.com/LlamaEdge/rag-api-server) project. 
The result is an OpenAI
compatible LLM service that is grounded by long-term knowledge on the server side. The client application
can simply chat with it or provide realtime / short-term memory since the LLM is already aware of the 
domain or background.

## Prerequisites

Install the [WasmEdge Runtime](https://github.com/WasmEdge/WasmEdge), our cross-platform LLM runtime.

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install_v2.sh | bash -s
```

Download the pre-built binary for the LlamaEdge API server with RAG support.

```
curl -LO https://github.com/LlamaEdge/rag-api-server/releases/latest/download/rag-api-server.wasm
```

And the chatbot web UI for the API server.

```
curl -LO https://github.com/second-state/chatbot-ui/releases/latest/download/chatbot-ui.tar.gz
tar xzf chatbot-ui.tar.gz
rm chatbot-ui.tar.gz
```

Download a chat model and an embedding model.

```
# The chat model is Llama3 8b chat
curl -LO https://huggingface.co/second-state/Llama-3-8B-Instruct-GGUF/resolve/main/Meta-Llama-3-8B-Instruct-Q5_K_M.gguf

# The embedding model is nomic-embed-text-v1.5
curl -LO https://huggingface.co/second-state/Nomic-embed-text-v1.5-Embedding-GGUF/resolve/main/nomic-embed-text-v1.5-f16.gguf
```

The embedding model is a special kind of LLM that turns sentences into vectors. The vectors can then be stored in a vector database and searched later. When the sentences are from a body of text that represents a knowledge domain, that vector database becomes our RAG knowledge base.

## Prepare a vector database

By default, we use Qdrant as the vector database. You can start a Qdrant instance on your server using Docker. The following command starts it in the background.

```
mkdir qdrant_storage
mkdir qdrant_snapshots

nohup docker run -d -p 6333:6333 -p 6334:6334 \
    -v $(pwd)/qdrant_storage:/qdrant/storage:z \
    -v $(pwd)/qdrant_snapshots:/qdrant/snapshots:z \
    qdrant/qdrant
```

Delete the `default` collection if it exists.

```
curl -X DELETE 'http://localhost:6333/collections/default'
```

Next, download a knowledge base, which is in the form of a vector snapshot. For example, here is an vector snapshot
created from a guidebook for Paris. It is a 768-dimension vector collection created by the embedding model [nomic-embed-text](https://huggingface.co/second-state/Nomic-embed-text-v1.5-Embedding-GGUF), which you have already downloaded.

```
curl -LO https://huggingface.co/datasets/gaianet/paris/resolve/main/paris_768_nomic-embed-text-v1.5-f16.snapshot
```

> You can create your own vector snapshots using tools discussed in the next several chapters.

Import the vector snapshot file into the local Qdrant database server's `default` collection.

```
curl -s -X POST http://localhost:6333/collections/default/snapshots/upload?priority=snapshot \
    -H 'Content-Type:multipart/form-data' \
    -F 'snapshot=@paris_768_nomic-embed-text-v1.5-f16.snapshot'
```

## Start the API server

Let's start the LlamaEdge RAG API server on port 8080. By default, it connects to the local Qdrant server.

```
wasmedge --dir .:. \
   --nn-preload default:GGML:AUTO:Meta-Llama-3-8B-Instruct-Q5_K_M.gguf \
   --nn-preload embedding:GGML:AUTO:nomic-embed-text-v1.5-f16.gguf \
   rag-api-server.wasm -p llama-3-chat,embedding --web-ui ./chatbot-ui \
     --model-name Meta-Llama-3-8B-Instruct-Q5_K_M,nomic-embed-text-v1.5-f16 \
     --ctx-size 8192,8192 \
     --batch-size 128,8192 \
     --rag-prompt "Use the following context to answer the question.\n----------------\n" \
     --log-prompts --log-stat
```

The CLI arguments are self-explanatory.
Notice that those arguments are different from the [llama-api-server.wasm](https://github.com/LlamaEdge/LlamaEdge/tree/main/api-server) app.

* The `--nn-proload` loads two models we just downloaded. The chat model is named `default` and the embedding model is named `embedding` .
* The `rag-api-server.wasm` is the API server app. It is written in Rust using LlamaEdge SDK, and is already compiled to cross-platform Wasm binary.
* The `--model-name` specifies the names of those two models so that API calls can be routed to specific models.
* The `--ctx-size` specifies the max input size for each of those two models listed in `--model-name`.
* The `--batch-size` specifies the batch processing size for each of those two models listed in `--model-name`. This parameter has a large impact on the RAM use of the API server.
* The `--rag-prompt` specifies the system prompt that introduces the context of the vector search and returns relevant context from qdrant.

There are a few optional `--qdrant-*` arguments you could use.

* The `--qdrant-url` is the API URL to the Qdrant server that contains the vector collection. It defaults to `http://localhost:6333`.
* The `--qdrant-collection-name` is the name of the vector collection that contains our knowledge base. It defaults to `default`.
* The `--qdrant-limit` is the maximum number of text chunks (search results) we could add to the prompt as the RAG context. It defaults to `3`.
* The `--qdrant-score-threshold` is the minimum score a search result must reach for its corresponding text chunk to be added to the RAG context. It defaults to `0.4`.

## Chat with supplemental knowledge

Just go to `http://localhost:8080/` from your web browser, and you will see a chatbot UI web page. You can now
ask any question about Paris and it will answer based on the Paris guidebook in the Qdrant database!

> This is a local web server serving a local LLM with contextual knowledge from a local vector database. Nothing leaves your computer!

Or, you can access it via the API. 

```
curl -X POST http://localhost:8080/v1/chat/completions \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{"messages":[{"role":"system", "content": "You are a helpful assistant."}, {"role":"user", "content": "Where is Paris?"}]}'

{
  "id":"18511d0f-b760-437f-a87f-8e95645822a0",
  "object":"chat.completion",
  "created":1711519741,
  "model":"Meta-Llama-3-8B-Instruct-Q5_K_M",
  "choices":[{"index":0,
    "message":{"role":"assistant","content":"Based on the provided context, Paris is located in the north-central part of France, situated along the Seine River. According to the text, people were living on the site of the present-day city by around 7600 BCE, and the modern city has spread from the island (the Île de la Cité) and far beyond both banks of the Seine."},
  "finish_reason":"stop"}],"usage":{"prompt_tokens":387,"completion_tokens":80,"total_tokens":467}
}
```

## Next steps

Now it is time to build your own LLM API server with long-term memory! You can start by using the same embedding model but with a different document. 

Good luck!

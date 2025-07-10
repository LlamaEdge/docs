---
sidebar_position: 1
---

# Working with embedding models

Embedding models compute vectors from text inputs. The vectors can then be used as search index
for semantic search in a vector database.

### Step 1: Install WasmEdge

First off, you'll need WasmEdge, a high-performance, lightweight, and extensible WebAssembly (Wasm) runtime optimized for server-side and edge computing. To install WasmEdge along with the necessary plugin for AI inference, open your terminal and execute the following command:

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install_v2.sh | bash -s
```

This command fetches and runs the WasmEdge installation script, which automatically installs WasmEdge and the WASI-NN plugin, essential for running LLM models like Llama 3.1.

### Step 2: Download the embedding model

Next, you'll need to obtain a model file. For this tutorial, we're focusing on the **GTW Qwen2 1.5B** model, which is a top rated text embedding model from Qwen. It generates vectors of 1536 dimensions. The steps are generally applicable to other models too. Use the following command to download the model file.

```
curl -LO https://huggingface.co/second-state/gte-Qwen2-1.5B-instruct-GGUF/resolve/main/gte-Qwen2-1.5B-instruct-Q5_K_M.gguf
```

### Step 3: Download a portable API server app

Next, you need an application that can build an OpenAI compatible API server for the model.
The [LlamaEdge api server app](https://github.com/LlamaEdge/LlamaEdge/tree/main/llama-api-server) is a lightweight and cross-platform Wasm app that works on any device
you might have. Just download the compiled binary app.

```
curl -LO https://github.com/second-state/LlamaEdge/releases/latest/download/llama-api-server.wasm
```

> The LlamaEdge apps are written in Rust and compiled to portable Wasm. That means they can run across devices and OSes without any change to the binary apps. You can simply download and run the compiled wasm apps regardless of your platform.

### Step 4: Start the API server

Start the API server with the following command. Notice that the context size of this particular embedding model is 
32k and the prompt template is `embedding`.

```
wasmedge --dir .:. --nn-preload default:GGML:AUTO:gte-Qwen2-1.5B-instruct-Q5_K_M.gguf llama-api-server.wasm --model-name gte-qwen2-1.5b --ctx-size 32768 --batch-size 8192 --ubatch-size 8192 --prompt-template embedding
```

### Step 5: Use the /embeddings API 

You can now send embedding requests to it using the OpenAI-compatible `/embeddings` API endpoint.

```
curl http://localhost:8080/v1/embeddings \
  -H "Content-Type: application/json" \
  -d '{
    "input": "The food was delicious and the waiter..."
  }'
```

The response is.

```
{"object":"list","data":[{"index":0,"object":"embedding","embedding":[0.02968290634,0.04592291266,0.05229084566,-0.001912750886,-0.01647545397,0.01744602434,0.008423444815,0.01363539882,-0.005849621724,-0.004947130103,-0.02326701023,0.1068811566,0.01074867789, ... 0.005662892945,-0.01796873659,0.02428019233,-0.0333112292]}],"model":"gte-qwen2-1.5b","usage":{"prompt_tokens":9,"completion_tokens":0,"total_tokens":9}}
```


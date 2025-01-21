---
sidebar_position: 3
---

# Start an OpenAI compatible API server

LlamaEdge support running LLMs along with embbedding models, allowing you to start a drop-in replacement for OpenAI API.

### Step 1: Install WasmEdge

First off, you'll need WasmEdge. To install WasmEdge along with the necessary plugin for AI inference, open your terminal and execute the following command:

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install_v2.sh | bash -s
```

This command fetches and runs the WasmEdge installation script, which automatically installs WasmEdge and the WASI-NN plugin, essential for running LLM models like Llama 3.1 and Nomix-embed models.

### Step 2: Download the LLM Model and Embedding Model

Next, you'll need to obtain a model file. For this tutorial, we're focusing on the **Llama 3.2 1B model finetuned for instruction following and Nomic embed model**, but the steps are generally applicable to other models too. Use the following command to download the model files.

```
# The chat model is Llama 3.2 1b chat
curl -LO https://huggingface.co/second-state/Llama-3.2-1B-Instruct-GGUF/resolve/main/Llama-3.2-1B-Instruct-Q5_K_M.gguf

# The embedding model is nomic-embed-text-v1.5
curl -LO https://huggingface.co/second-state/Nomic-embed-text-v1.5-Embedding-GGUF/resolve/main/nomic-embed-text-v1.5-f16.gguf
```

This command downloads the Llama-3.2-1B-Instruct model and nomic-embed-text-v1.5 model from Huggingface, an AI model hosting platform.

### Step 3: Download a Portable OpenAI Compatible Server

To start an OpenAI-compatible API server, you need the [LlamaEdge API server](https://github.com/LlamaEdge/LlamaEdge/tree/main/api-server) app.

```
curl -LO https://github.com/second-state/LlamaEdge/releases/latest/download/llama-api-server.wasm
```

The `llama-api-server.wasm` is a web server with an OpenAI-compatible API.

> The LlamaEdge apps are written in Rust and compiled to portable Wasm. That means they can run across devices and OSes without any change to the binary apps. You can simply download and run the compiled wasm apps regardless of your platform.

### Step 4: Start the API Server

With everything set up, it's time to run the models as follows.

```
wasmedge --dir .:. \
   --nn-preload default:GGML:AUTO:Llama-3.2-1B-Instruct-Q5_K_M.gguf \
   --nn-preload embedding:GGML:AUTO:nomic-embed-text-v1.5-f16.gguf \
   llama-api-server.wasm -p llama-3-chat,embedding \
     --model-name Llama-3.2-1B-Instruct-Q5_K_M,nomic-embed-text-v1.5-f16 \
     --ctx-size 8192,8192 \
     --batch-size 128,8192 \
     --log-prompts --log-stat
```

This command executes the chat application, allowing you to start interacting with the Llama 3 8B model. Here, `wasmedge` is the command to run the WasmEdge runtime, `--nn-preload` specifies the model to use with the WASI-NN plugin, and `-p` sets the prompt template for the chat.

### Step 5: Send an API Request 

Now you have a drop-in replacement for OpenAI API. You can integrate it with any agents/frameworks based on OpenAI.

|Config option | Value |
|-----|--------|
| API endpoint URL | http://localhost:8080/v1 |
| Model Name (for LLM) | Llama-3.2-1B-Instruct-Q5_K_M |
| Model Name (for Text embedding) | nomic-embed-text-v1.5-f16 |
| API key | Empty or any value |


Congratulations! Next, you can integrate your APT server with [OpenAI ecosystem apps](/docs/category/ecosystem-apps).


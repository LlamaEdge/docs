---
sidebar_position: 1
---

# Start an LlamaEdge API service

Since LlamaEdge provides an OpenAI-compatible API service, it can be a drop-in replacement for OpenAI in almost all LLM applications and frameworks. 
Checkout the articles in this section for instructions and examples for how to use locally hosted LlamaEdge API services in popular LLM apps.

But first, you will need to start an [LlamaEdge API server](https://github.com/LlamaEdge/LlamaEdge/tree/main/api-server). But the steps are a little different from just a chatbot.

## Step 1: Install WasmEdge

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install_v2.sh | bash -s
```

## Step 2: Download an LLM model

```
curl -LO https://huggingface.co/second-state/Llama-3-8B-Instruct-GGUF/resolve/main/Meta-Llama-3-8B-Instruct-Q5_K_M.gguf
```

## Step 3: Download an embedding model

```
curl -LO https://huggingface.co/second-state/All-MiniLM-L6-v2-Embedding-GGUF/resolve/main/all-MiniLM-L6-v2-ggml-model-f16.gguf
```

It is used by many agent and RAG apps to convert text-based knowledge into vectors for easy search and retrieval.

## Step 4: Start the API server!

```
wasmedge --dir .:. \
    --nn-preload default:GGML:AUTO:Meta-Llama-3-8B-Instruct-Q5_K_M.gguf \
    --nn-preload embedding:GGML:AUTO:all-MiniLM-L6-v2-ggml-model-f16.gguf \
    llama-api-server.wasm \
    --model-alias default \
    --model-name llama-3-8b-chat \
    --prompt-template llama-3-chat \
    --ctx-size 8192 \
    --embedding-model-alias embedding \
    --embedding-model-name all-minilm-l6-v2 \
    --embedding-ctx-size 256
```

You can learn more about these CLI options [here](https://github.com/LlamaEdge/LlamaEdge/tree/main/api-server).

* Options for the chat LLM model (i.e., the `Meta-Llama-3-8B-Instruct-Q5_K_M.gguf`)
    * The `--model-alias` matches the chat LLM model in `--nn-preload`, which is `default` in this case.
    * The `--model-name` can be any string, and you will need it in API calls when the client wants to select a model to chat.
    * The `--prompt-template` and `--ctx-size` are settings for the chat LLM model.
* Options for the embedding model (i.e., the `all-MiniLM-L6-v2-ggml-model-f16.gguf`)
    * The `--embedding-model-alias` matches the chat LLM model in `--nn-preload`, which is `embedding` in this case.
    * The `--embedding-model-name` can be any string, and you will need it in API calls.
    * The `--embedding-ctx-size` is the max context size for the input to the embedding model.

That's it. You can now test the API server by sending it a request.
Notice that model name `llama-3-8b-chat` matches what you specified in the `llama-api-server.wasm` command.

```
curl -X POST http://0.0.0.0:8080/v1/chat/completions -H 'accept:application/json' -H 'Content-Type: application/json' -d '{"messages":[{"role":"system", "content":"You are a helpful AI assistant"}, {"role":"user", "content":"What is the capital of France?"}], "model":"llama-3-8b-chat"}'
```

You should receive a JSON message that contains a reply to the question in the response.

## OpenAI replacement

Now, you can ready to use this API server in OpenAI ecosystem apps as a drop-in replacement for the OpenAI API!
In general, for any OpenAI tool, you could just replace the following.

|Config option | Value | Note |
|-----|--------|-------|
| API endpoint URL | `http://localhost:8080/v1` | If the server is accessible from the web, you could use the public IP and port |
| Model Name (for LLM) | `llama-3-8b-chat` | The value specified in the `--model-name` option |
| Model Name (for Text embedding) | `all-minilm-l6-v2` | The value specified in the `--embedding-model-name` option |
| API key | Empty | Or any value if the app does not permit empty string |



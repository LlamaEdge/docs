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
curl -LO https://huggingface.co/second-state/Meta-Llama-3.1-8B-Instruct-GGUF/resolve/main/Meta-Llama-3.1-8B-Instruct-Q5_K_M.gguf
```

## Step 3: Download an embedding model

```
curl -LO https://huggingface.co/gaianet/Nomic-embed-text-v1.5-Embedding-GGUF/resolve/main/nomic-embed-text-v1.5.f16.gguf
```

It is used by many agent and RAG apps to convert text-based knowledge into vectors for easy search and retrieval.

## Step 4: Start the API server!

```
wasmedge --dir .:. \
    --nn-preload default:GGML:AUTO:Meta-Llama-3.1-8B-Instruct-Q5_K_M.gguf \
    --nn-preload embedding:GGML:AUTO:nomic-embed-text-v1.5.f16.gguf \
    llama-api-server.wasm \
    --model-alias default,embedding \
    --model-name llama-3-8b-chat,nomic-embed \
    --prompt-template llama-3-chat,embedding \
    --batch-size 128,8192 \
    --ctx-size 8192,8192
```

You can learn more about these CLI options [here](https://github.com/LlamaEdge/LlamaEdge/tree/main/api-server).

* The `--model-alias` specifies which of the preloaded models is for chat and embedding respectively. In this case
  * The alias `default` corresponds to `Meta-Llama-3.1-8B-Instruct-Q5_K_M.gguf`
  * The alias `embedding` corresponds to `nomic-embed-text-v1.5.f16.gguf`
* The `--model-name` can be any string, and you will need it in API calls when the client wants to select a model to interact with. The two values correspond to the `default` and `embedding` model respectively.
* The `--prompt-template` specifies the prompt template name for the chat model, and it uses `embedding` for the prompt template name for the embedding model.
* The `--ctx-size` specifies the context window size for the `default` and `embedding` model respectively.
* The `--batch-size` specifies the batch job size for the `default` and `embedding` model respectively.

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
| Model Name (for LLM) | `llama-3-8b-chat` | The first value specified in the `--model-name` option |
| Model Name (for Text embedding) | `nomic-embed` | The second value specified in the `--model-name` option |
| API key | Empty | Or any value if the app does not permit empty string |

## The OpenAI Python library

You can install the [official OpenAI Python library](https://pypi.org/project/openai/) as follows.

```
pip install openai
```

When you create an OpenAI client using the library, you can pass in the API endpoint point as the `base_url`.

```
import openai

client = openai.OpenAI(base_url="http://localhost:8080/v1", api_key="")
```

Alternatively, you could set an environment variable at the OS level.

```
export OPENAI_API_BASE=http://localhost:8080/v1
```

Then, when you make API calls from the `client`, make sure that the `model` is set to the model name
available on your node.

```
response = client.chat.completions.create(
    model="llama-3-8b-chat",
    messages=[
        {"role": "system", "content": "You are a strategic reasoner."},
            {"role": "user", "content": "What is the purpose of life?"}
        ],
        temperature=0.7,
        max_tokens=500
    ]
)
```

That's it! You can now take any application built with the official OpenAI Python library and use your own
LlamaEdge device as its backend!


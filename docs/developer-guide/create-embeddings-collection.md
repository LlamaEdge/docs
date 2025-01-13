---
sidebar_position: 5
---

# Create knowledge embeddings using the API server

The LlamaEdge API server project demonstrates how to support OpenAI style APIs to upload, chunck, and create embeddings for a text document. In this guide, I will show you how to use those API endpoints as a developer.

> This article is intended to demonstrate capabilities of the open source API server example. You should review the API server source code to learn how those features are implemented. If you are running an RAG application with the API server, check out [this guide](../user-guide/server-side-rag/quick-start).

## Build the API server

Check out the source code and build it using Rust `cargo` tools.

```
git clone https://github.com/LlamaEdge/LlamaEdge

cd LlamaEdge/api-server
cargo build --target wasm32-wasip1 --release
```

The `llama-api-server.wasm` file is in the `target` directory.

```
cp target/wasm32-wasip1/release/llama-api-server.wasm . 
```

## Download models

We will need an LLM and a specialized embedding model. While the LLM technically can create embeddings, specialized embedding models can do it much much better.

```
# The chat model is Llama2 7b chat
curl -LO https://huggingface.co/second-state/Llama-2-7B-Chat-GGUF/resolve/main/Llama-2-7b-chat-hf-Q5_K_M.gguf

# The embedding model is all-MiniLM-L6-v2
curl -LO https://huggingface.co/second-state/All-MiniLM-L6-v2-Embedding-GGUF/resolve/main/all-MiniLM-L6-v2-ggml-model-f16.gguf
```

## Start the API server

We will now start the API server with both models. The LLM is named `default` and the embedding model is named `embedding`. They each have an external facing model name in the `--model-name` argument.

```
wasmedge --dir .:. \
   --nn-preload default:GGML:AUTO:Llama-2-7b-chat-hf-Q5_K_M.gguf \
   --nn-preload embedding:GGML:AUTO:all-MiniLM-L6-v2-ggml-model-f16.gguf \
   llama-api-server.wasm -p llama-2-chat,embedding --web-ui ./chatbot-ui \
     --model-name Llama-2-7b-chat-hf-Q5_K_M,all-MiniLM-L6-v2-ggml-model-f16 \
     --ctx-size 4096,384 \
     --log-prompts --log-stat
```

## Create the embeddings

First, we use the `/files` API to upload a file `paris.txt` to the API server.

```
curl -X POST http://127.0.0.1:8080/v1/files -F "file=@paris.txt"
```

If the command is successful, you should see the similar output as below in your terminal.

```
{
    "id": "file_4bc24593-2a57-4646-af16-028855e7802e",
    "bytes": 2161,
    "created_at": 1711611801,
    "filename": "paris.txt",
    "object": "file",
    "purpose": "assistants"
}
```

Next, take the `id` and request the `/chunks` API to chunk the file `paris.txt` into smaller pieces. The reason is that each embedding vector can only hold limited amount of information. The embedding model can "understand" the file content, and determine the optimistic places to break up the text into chunks.

```
curl -X POST http://localhost:8080/v1/chunks \
    -H 'accept:application/json' \
    -H 'Content-Type: application/json' \
    -d '{"id":"file_4bc24593-2a57-4646-af16-028855e7802e", "filename":"paris.txt"}'
```

The following is an example return with the generated chunks.

```
{
    "id": "file_4bc24593-2a57-4646-af16-028855e7802e",
    "filename": "paris.txt",
    "chunks": [
        "Paris, city and capital of France, ..., for Paris has retained its importance as a centre for education and intellectual pursuits.",
        "Paris’s site at a crossroads ..., drawing to itself much of the talent and vitality of the provinces."
    ]
}
```

Finally, use the `/embeddings` API to generate the embedding vectors. Make sure that you pass in the embedding model name.

```bash
curl -X POST http://localhost:8080/v1/embeddings \
    -H 'accept:application/json' \
    -H 'Content-Type: application/json' \
    -d '{"model": "all-MiniLM-L6-v2-ggml-model-f16", "input":["Paris, city and capital of France, ..., for Paris has retained its importance as a centre for education and intellectual pursuits.", "Paris’s site at a crossroads ..., drawing to itself much of the talent and vitality of the provinces."]}'
```

The embeddings returned are like below.

```json
{
    "object": "list",
    "data": [
        {
            "index": 0,
            "object": "embedding",
            "embedding": [
                0.1428378969,
                -0.0447309874,
                0.007660218049,
                ...
                -0.0128974719,
                -0.03543198109,
                0.03974733502,
                0.00946635101,
                -0.01531364303
            ]
        },
        {
            "index": 1,
            "object": "embedding",
            "embedding": [
                0.0697753951,
                -0.0001159032545,
                0.02073983476,
                ...
                0.03565846011,
                -0.04550019652,
                0.02691745944,
                0.02498772368,
                -0.003226313973
            ]
        }
    ],
    "model": "all-MiniLM-L6-v2-ggml-model-f16",
    "usage": {
        "prompt_tokens": 491,
        "completion_tokens": 0,
        "total_tokens": 491
    }
}
```

## Next step

Once you have the embeddings in a JSON file, you can store them into a vector database. It will probably require you to write a script to combine each vector point with its corresponding source text, and then upsert into the database's vector collection. This step will be specific to the vector database and RAG strategy you choose.



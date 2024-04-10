---
sidebar_position: 3
---

# Server-side RAG with LlamaEdge

RAG is an important technique to inject new and updated knowledge into an LLM application. It improves accuracy and reduces the hallucination of LLMs, especially in specific domains covered by the RAG knowledge base. In the past, most RAG setups are very complex requiring the orchestration of multiple services and components in heavyweight frameworks in Python. 

LlamaEdge, on the other hand, provides a Rust-based development platform that enables developers to combine RAG logic into the chat API server! The compact and efficient single-binary API server is cross-platform and runs on any GPU or AI accelerator device, which allows it to be easily deployed across the cloud and the edge. 

>Since the LlamaEdge API server encapsulates the RAG logic behind the OpenAI-compatible web service API, you can use any OpenAI-compatible frontend to interact with it. When you ask a question through the API, the answer is already RAG-enhanced. No client-side RAG (i.e., to index and search the RAG vector store in the client) is needed.

The LlamaEdge API server is a powerful demo of the LlamaEdge development platform. It showcases how to use the LlamaEdge SDK to build a generic RAG application. You can, of course, implement your own RAG logic using the LlamaEdge SDK and build your own RAG API server! In this tutorial, we will explain how to use the default RAG features built into our standard API server.

## Prerequisites

Install the [WasmEdge Runtime](https://github.com/WasmEdge/WasmEdge), our cross-platform LLM runtime.

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- --plugins wasi_nn-ggml
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
# The chat model is Llama2 7b chat
curl -LO https://huggingface.co/second-state/Llama-2-7B-Chat-GGUF/resolve/main/Llama-2-7b-chat-hf-Q5_K_M.gguf

# The embedding model is all-MiniLM-L6-v2
curl -LO https://huggingface.co/second-state/All-MiniLM-L6-v2-Embedding-GGUF/resolve/main/all-MiniLM-L6-v2-ggml-model-f16.gguf
```

The embedding model is a special kind of LLM that turns sentences into vectors. The vectors can then be stored in a vector database and searched later. When the sentences are from a body of text that represents a knowledge domain, that vector database becomes our RAG knowledge base.

## Start a vector database

By default, we use Qdrant as the vector database. You can start a Qdrant instance on your server using Docker. The following command starts it in the background.

```
mkdir qdrant_storage
mkdir qdrant_snapshots

nohup docker run -d -p 6333:6333 -p 6334:6334 \
    -v $(pwd)/qdrant_storage:/qdrant/storage:z \
    -v $(pwd)/qdrant_snapshots:/qdrant/snapshots:z \
    qdrant/qdrant
```

## Start the API server

Let’s start the LlamaEdge RAG API server on port 8080. By default, it connects to the local Qdrant server.

```
wasmedge --dir .:. \
   --nn-preload default:GGML:AUTO:Llama-2-7b-chat-hf-Q5_K_M.gguf \
   --nn-preload embedding:GGML:AUTO:all-MiniLM-L6-v2-ggml-model-f16.gguf \
   rag-api-server.wasm -p llama-2-chat --web-ui ./chatbot-ui \
     --model-name Llama-2-7b-chat-hf-Q5_K_M,all-MiniLM-L6-v2-ggml-model-f16 \
     --ctx-size 4096,384 \
     --rag-prompt "Use the following context to answer the question.\n----------------\n" \
     --log-prompts --log-stat
```

The CLI arguments are self-explanatory.

* The `--nn-proload` loads two models we just downloaded. The chat model is named `default` and the embedding model is named `embedding` .
* The `llama-api-server.wasm` is the API server app. It is written in Rust using LlamaEdge SDK, and is already compiled to cross-platform Wasm binary.
* The `--model-name` specifies the names of those two models so that API calls can to routed to specific models.
* The `--ctx-size` specifies the max input size for each of those two models listed in `--model-name`.
* The `--rag-prompt` specifies the system prompt that introduces the context if the vector search returns relevant context from qdrant.

There are a few optional `--qdrant-*` arguments you could use.

* The `--qdrant-url` is the API URL to the Qdrant server that contains the vector collection. It defaults to `http://localhost:6333`.
* The `--qdrant-collection-name` is the name of the vector collection that contains our knowledge base. It defaults to `default`.
* The `--qdrant-limit` is the max number of text chunks (search results) we could add to the prompt as the RAG context. It defaults to `3`.
* The `--qdrant-score-threshold` is minimum score a search result must reach for its corresponding text chunk to be added to the RAG context. It defaults to `0.4`.

## Create knowledge embeddings

The LlamaEdge RAG API server provides an API endpoint `/create/rag` that takes a text file, segments it into small chunks, turns the chunks into embeddings (i.e., vectors), and then stores the embeddings into the Qdrant database.
Here we submit a travel guide for Paris France.

```
curl -LO https://huggingface.co/datasets/gaianet/paris/raw/main/paris.txt

curl -X POST http://127.0.0.1:8080/v1/create/rag -F "file=@paris.txt"
```

Now, the Qdrant database has a vector collection called `default` which contains embeddings from the Paris guide. You can see the stats of the vector collection as follows.

```
curl 'http://localhost:6333/collections/default'
```

Of course, the `/create/rag` API is rather primitive in chunking documents and creating embeddings. For many use cases, you should [create your own embedding vectors](#use-your-own-embedding-algos).

> The `/create/rag` is a simple combination of [several more basic API endpoints](../developer-guide/create-embeddings-collection.md) provided by the API server. You can learn more about them in the developer guide.

## Chat with supplemental RAG knowledge

The LlamaEdge RAG API server takes every new user request, searches relevant embeddings based on the request, and then adds search results to the prompt.

For example, if you ask the question “Where is Paris?”, the actual prompt to the LLM will contain 3 paragraphs of text that are relevant to the question. 


```
<s>[INST] <<SYS>>
You are a helpful assistant.
Use the following pieces of context to answer the user's question.
If you don't know the answer, just say that you don't know, don't try to make up an answer.
----------------
"For centuries Paris has been one of the world’s most important and attractive cities. It is appreciated for the opportunities it offers for business and commerce, for study, for culture, and for entertainment; its gastronomy, haute couture, painting, literature, and intellectual community especially enjoy an enviable reputation. Its sobriquet “the City of Light” (“la Ville Lumière”), earned during the Enlightenment, remains appropriate, for Paris has retained its importance as a centre for education and intellectual pursuits."

"Under Hugh Capet (ruled 987–996) and the Capetian dynasty the preeminence of Paris was firmly established, and Paris became the political and cultural hub as modern France took shape. France has long been a highly centralized country, and Paris has come to be identified with a powerful central state, drawing to itself much of the talent and vitality of the provinces."

"Paris, city and capital of France, situated in the north-central part of the country. People were living on the site of the present-day city, located along the Seine River some 233 miles (375 km) upstream from the river’s mouth on the English Channel (La Manche), by about 7600 BCE. The modern city has spread from the island (the Île de la Cité) and far beyond both banks of the Seine." <</SYS>>

Where is Paris? [/INST]
```


You can try the application from its web UI. 

http://localhost:8080/

Or, you can access it via the API. 


```
curl -X POST http://localhost:8080/v1/chat/completions \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{"messages":[{"role":"system", "content": "You are a helpful assistant."}, {"role":"user", "content": "Where is Paris?"}], "model":"Llama-2-7b-chat-hf-Q5_K_M"}'

{
  "id":"18511d0f-b760-437f-a87f-8e95645822a0",
  "object":"chat.completion",
  "created":1711519741,
  "model":"Llama-2-7b-chat-hf-Q5_K_M",
  "choices":[{"index":0,
    "message":{"role":"assistant","content":"Based on the provided context, Paris is located in the north-central part of France, situated along the Seine River. According to the text, people were living on the site of the present-day city by around 7600 BCE, and the modern city has spread from the island (the Île de la Cité) and far beyond both banks of the Seine."},
  "finish_reason":"stop"}],"usage":{"prompt_tokens":387,"completion_tokens":80,"total_tokens":467}
}
```

## Use your own embedding algos

You could build your own embeddings database. By chunking the documents yourself, you will probably get better results. You can also then share the database snapshot with others. 

Delete the `default` collection if it exists. 


```
curl -X DELETE 'http://localhost:6333/collections/default'
```


Create a new collection called `default`. Notice that it is 384 dimensions. That is the output vector size of the embedding model `all-MiniLM-L6-v2`. If you are using a different embedding model, you should use a dimension that fits the model. 


```
curl -X PUT 'http://localhost:6333/collections/default' \
  -H 'Content-Type: application/json' \
  --data-raw '{
    "vectors": {
      "size": 384,
      "distance": "Cosine",
      "on_disk": true
    }
  }'
```

Download a program to chunk a document and create embeddings. 

```
curl -LO https://github.com/YuanTony/chemistry-assistant/raw/main/rag-embeddings/create_embeddings.wasm
```

It chunks the document based on empty lines. So, you MUST prepare your source document this way — to segment the document into sections of around 200 words with empty lines. See the example document here. You can check out the [Rust source code here](https://github.com/YuanTony/chemistry-assistant/tree/main/rag-embeddings) and modify it if you need to use a different chunking strategy. 

Next, you can run the program by passing a collection name, vector dimension, and the source document. Make sure that Qdrant is running on your local machine. The model is preloaded under the name `embedding`. The wasm app then uses the `embedding` model to create the 384-dimension vectors from [paris.txt](https://huggingface.co/datasets/gaianet/paris/raw/main/paris.txt) and saves them into the `default` collection.

```
curl -LO https://huggingface.co/datasets/gaianet/paris/raw/main/paris.txt

wasmedge --dir .:. \
  --nn-preload embedding:GGML:AUTO:all-MiniLM-L6-v2-ggml-model-f16.gguf \
  create_embeddings.wasm embedding default 384 paris.txt
```

You can create a snapshot of the collection, which can be shared and loaded into a different Qdrant database. You can find the snapshot file in the `qdrant_snapshots` directory.

```
curl -X POST 'http://localhost:6333/collections/default/snapshots'
```

If you already have a snapshot file `paris.snapshot`, you can import it as follows. [Learn more about creating and restoring Qdrant snapshots](https://qdrant.tech/documentation/tutorials/create-snapshot/).

```
curl -X POST 'http://localhost:6333/collections/default/snapshots/upload?priority=snapshot' \
    -H 'Content-Type:multipart/form-data' \
    -F 'snapshot=@paris.snapshot'
```

Finally, start your LlamaEdge API server again. Go to http://localhost:8080/ again and have it answer questions based on your new knowledge base!

## Next steps

Now it is time to build your own RAG-enabled API server! You can start by using the same embedding model but with a different document. Make sure that you segment the document into chucks of less than 200 words each, and separate each chunk with an empty line. You can experiment with different chucking strategies to evaluate how to come up with the best search results when the user asks new questions. 

Next, you can experiment with a different embedding model. Notice that each embedding model has a different context length (ie max length for each chunk) and vector size. You must also use the same embedding model to generate embeddings and perform RAG search / chat. 

Good luck!

---
sidebar_position: 4
---

# Use the API server

The LlamaEdge RAG API server provides an API endpoint `/create/rag` that takes a text file, segments it into small chunks, turns the chunks into embeddings (i.e., vectors), and then stores the embeddings into the Qdrant database.
It provides an easy way to quick generate embeddings from a body text into a Qdrant database collection.

## Prerequisites

You will need to follow [this guide](quick-start) to start a Qdrant database instance and a local `llama-api-server.wasm` server.

Delete the `default` collection if it exists. 

```
curl -X DELETE 'http://localhost:6333/collections/default'
```

## Step by step example

In this example, we will use a text document `paris.txt`, and simply submit it to the LlamaEdge API server.

```
curl -LO https://huggingface.co/datasets/gaianet/paris/raw/main/paris.txt

curl -X POST http://127.0.0.1:8080/v1/create/rag -F "file=@paris.txt"
```

Now, the Qdrant database has a vector collection called `default` which contains embeddings from the Paris guide. You can see the stats of the vector collection as follows.

```
curl 'http://localhost:6333/collections/default'
```

Of course, the `/create/rag` API is rather primitive in chunking documents and creating embeddings. For many use cases, you should [create your own embedding vectors](text).

> The `/create/rag` is a simple combination of [several more basic API endpoints](../../developer-guide/create-embeddings-collection.md) provided by the API server. You can learn more about them in the developer guide.

Have fun!

---
sidebar_position: 9
---

# Agent Zero

[Agent Zero](https://github.com/frdel/agent-zero) is a general purpose AI agent application. You can simply ask it to accomplish tasks on the command line. 
It is designed to be dynamic, organically growing, and learning as users use it. It leverages your computer as a tool to accomplish your tasks.

## Prerequisites

Follow [this guide](https://llamaedge.com/docs/user-guide/get-started-with-llamaedge) to run an open-source LLM locally. In this example, we will use [Llama-3.1-8B](https://huggingface.co/second-state/Meta-Llama-3.1-8B-Instruct-GGUF) as the chat model and nomic-text-embed as the embedding model. 

```
curl -LO https://huggingface.co/second-state/Meta-Llama-3.1-8B-Instruct-GGUF/resolve/main/Meta-Llama-3.1-8B-Instruct-Q5_K_M.gguf
curl -LO https://huggingface.co/second-state/Nomic-embed-text-v1.5-Embedding-GGUF/resolve/main/nomic-embed-text-v1.5-Q5_K_M.gguf
```

Then start the LlamaEdge API server with the Llama-3.1-8B model `Meta-Llama-3.1-8B-Instruct-Q5_K_M` and nomic embed model `nomic-embed-text-v1.5.f16`.

```
wasmedge --dir .:. \
    --nn-preload default:GGML:AUTO:Meta-Llama-3.1-8B-Instruct-Q5_K_M.gguf \
    --nn-preload embedding:GGML:AUTO:nomic-embed-text-v1.5.f16.gguf \
    llama-api-server.wasm \
    --model-alias default,embedding \
    --model-name Meta-Llama-3.1-8B-Instruct-Q5_K_M,nomic-embed-text-v1.5.f16 \
    --prompt-template llama-3-chat,embedding \
    --batch-size 128,8192 \
    --ctx-size 32768,8192
```
> You can also start a Gaia node for the Llama-3.1-8B model and the nomic embed model. You can then use the node's API URL endpoint and model name in this app.


**You will also need to make sure your Docker engine is running.** Because the Agent Zero framework will leverage Docker to execute the generated code.

## Configure the agent

First, we will need to get the source code of a Gaia-compatible version of Agent Zero.

```
git clone https://github.com/JYC0413/agent-zero-gaia.git
cd agent-zero-gaia
```

Then, let's install the required dependencies.

```
pip install -r requirements.txt
```

Next, let's configure the gaia node and other parameters.

```
cp example.env .env
```

You will need to configure the following items:

* `CHAT_MODEL_BASE_URL`: URL for the LLM API base URL. E.g., `http://localhost:8080/v1`
* `CHAT_MODEL_NAME`: Name of the chat model to be used. E.g., `Meta-Llama-3.1-8B-Instruct-Q5_K_M`
* `CHAT_API_KEY`: An API key to access the LLM services. You can enter several random characters here. E.g., `LlamaEdge`
* `EMBEDDING_MODEL_BASE_URL`: URL for the embedding model API base URL. E.g., `http://localhost:8080/v1`
* `EMBEDDING_MODEL_NAME`: Name of the embedding model name. E.g., `nomic-embed-text-v1.5.f16`
* `EMBEDDING_API_KEY`: An API key to access the embedding services. You can enter several random characters here. E.g., `LlamaEdge`

## Run the agent

Finally, let's run the Agent Zero application backed by the Llama 3.1 8b model.

```
python main.py
```

You can interact with Agent Zero via the command line. You will see the Agent Zero framework will call the related tools and execute some code to complete your task.

### Example 1

Get time in Berlin and Seattle

![](agent-zero-01.png)

![](agent-zero-02.png)

Save the above result in a file using node.js

![](agent-zero-03.png)

### Example 2

Install [mc](https://midnight-commander.org/). do not use `-y` flag

![](agent-zero-04.png)

### Example 3

Run `top`, show the system load

![](agent-zero-05.png)

### Example 4

Memorize my openai api key - 563434643464

![](agent-zero-06.png)

### Example 5

Extract an mp3 audio track from a mp4 video file using ffmpeg. You will need to put an mp4 file (e.g., `video.mp4`) into the `work_dir` and tell the agent about it.

![](agent-zero-07.png)

![](agent-zero-08.png)

![](agent-zero-09.png)




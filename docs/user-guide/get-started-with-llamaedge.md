---
sidebar_position: 2
---

# Getting started with LlamaEdge


Let's dive into a simple and practical tutorial on getting started with LlamaEdge, focusing on how to use a Command Line Interface (CLI) installer to run a model, along with some useful WasmEdge commands. This guide can be adjusted and applied to run Llama 2 series of models, tailored to give you a hands-on approach to running your large language model with LlamaEdge.


### Step 1: Installing WasmEdge

First off, you'll need WasmEdge, a high-performance, lightweight, and extensible WebAssembly (Wasm) runtime optimized for server-side and edge computing. To install WasmEdge along with the necessary plugin for AI inference, open your terminal and execute the following command:


```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- --plugin wasi_nn-ggml
```


This command fetches and runs the WasmEdge installation script, which automatically installs WasmEdge and the WASI-NN plugin, essential for running LLM models like Llama 2.


### Step 2: Downloading the Model

Next, you'll need to obtain a model file. For this tutorial, we're focusing on the **Llama 2-13B model**, but the steps are generally applicable to other models too. Use the following command to download the model file:


```
curl -LO https://huggingface.co/second-state/Llama-2-13B-Chat-GGUF/resolve/main/llama-2-13b-chat.Q5_K_M.gguf
```


This command downloads the Llama 2-13B model from Hugging Face, a platform hosting various LLM models.


### Step 3: Downloading the Wasm Application

To interact with the model, you'll need a Wasm application. For a chat application, use:


```
curl -LO https://github.com/second-state/LlamaEdge/releases/latest/download/llama-chat.wasm
```


This downloads a portable Wasm file that lets you chat with the Llama 2 model directly from the command line.


### Step 4: Running the Model

With everything set up, it's time to run the model:


```
wasmedge --dir .:. --nn-preload default:GGML:AUTO:llama-2-13b-chat.Q5_K_M.gguf llama-chat.wasm -p llama-2-chat
```


This command executes the chat application, allowing you to start interacting with the Llama 2 13b model. Here, `wasmedge` is the command to run the WasmEdge runtime, `--nn-preload` specifies the model to use with the WASI-NN plugin, and `-p` sets the prompt template for the chat.


### Optional: Creating an OpenAI-Compatible API Service

If you're interested in creating a web API service that's compatible with OpenAI, download the API server application:


```
curl -LO https://github.com/second-state/LlamaEdge/releases/latest/download/llama-api-server.wasm
```


Then, start the API server:


```
wasmedge --dir .:. --nn-preload default:GGML:AUTO:llama-2-13b-chat.Q5_K_M.gguf llama-api-server.wasm -p llama-2-chat
```


You can interact with this API using tools like `curl` from another terminal:


```
curl -X POST http://0.0.0.0:8080/v1/chat/completions -H 'accept:application/json' -H 'Content-Type: application/json' -d '{"messages":[{"role":"system", "content":"You are a helpful AI assistant"}, {"role":"user", "content":"What is the capital of France?"}], "model":"llama-2-13b-chat"}'
```


That's it! You've just learned how to set up and run a Llama model using LlamaEdge. These steps demonstrate the ease and flexibility of deploying AI models across various devices without worrying about complex dependencies.
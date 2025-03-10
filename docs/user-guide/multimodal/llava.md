---
sidebar_position: 1
---

# Quick start with the Llava mdoels

[Llava-v1.6-Vicuna-7B](https://huggingface.co/liuhaotian/llava-v1.6-vicuna-7b) is open-source community's answer to OpenAI's multimodal GPT-4-V. It is also known as a Visual Language Model for its ability to handle visual images and language in a conversation.  This guide shows you how to set up and run Llava-v1.6-Vicuna-7B using the LlamaEdge Llama API server server, which provides an OpenAI-compatible API interface.

### Step 1: Install WasmEdge

First off, you'll need WasmEdge, a high-performance, lightweight, and extensible WebAssembly (Wasm) runtime optimized for server-side and edge computing. To install WasmEdge along with the necessary plugin for AI inference, open your terminal and execute the following command:

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install_v2.sh | bash -s
```

### Step 2: Download the LLM model

Next, you'll need to obtain model files: the **Llava-v1.6-Vicuna-7B model** and the **mmproj model**.

```
curl -LO https://huggingface.co/second-state/Llava-v1.6-Vicuna-7B-GGUF/resolve/main/llava-v1.6-vicuna-7b-Q5_K_M.gguf
curl -LO https://huggingface.co/second-state/Llava-v1.6-Vicuna-7B-GGUF/resolve/main/llava-v1.6-vicuna-7b-mmproj-model-f16.gguf
```

### Step 3: Download a portable chatbot app

Next, you need an application that can load the model and provide a UI to interact with the model.
The [LlamaEdge api server app](https://github.com/LlamaEdge/LlamaEdge/tree/main/llama-api-server) is a lightweight and cross-platform Wasm app that works on any device
you might have. Just download the compiled binary app.

```
curl -LO https://github.com/second-state/LlamaEdge/releases/latest/download/llama-api-server.wasm
```

> The LlamaEdge apps are written in Rust and compiled to portable Wasm. That means they can run across devices and OSes without any change to the binary apps. You can simply download and run the compiled wasm apps regardless of your platform.

### Step 4: Chat with the chatbot UI 

The `llama-api-server.wasm` is a web server with an OpenAI-compatible API. You still need HTML files for the chatbot UI.
Download and unzip the HTML UI files as follows.

```
curl -LO https://github.com/LlamaEdge/chatbot-ui/releases/latest/download/chatbot-ui.tar.gz
tar xzf chatbot-ui.tar.gz
rm chatbot-ui.tar.gz
```

Then, start the web server.


```
wasmedge --dir .:. --nn-preload default:GGML:AUTO:llava-v1.6-vicuna-7b-Q5_K_M.gguf llama-api-server.wasm -p vicuna-llava -c 4096 --llava-mmproj llava-v1.6-vicuna-7b-mmproj-model-f16.gguf -m llava-v1.6-vicuna-7b
```

Go to `http://localhost:8080` on your computer to access the chatbot UI on a web page! You can upload an imange and chat with the model based on the image.

Congratulations! You have now started an multimodal app on your own device.

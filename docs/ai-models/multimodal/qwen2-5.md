---
sidebar_position: 2
---

# Quick start with the Qwen 2.5 VL model

[Qwen 2.5 VL](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct) is the latest vision-language model from the Qwen series, designed to handle a wide range of complex multimodal tasks. It excels at understanding visual content such as text, charts, and layouts, and can act as an intelligent agent capable of interacting with tools and devices.


### Step 1: Install WasmEdge

First off, you'll need WasmEdge, a high-performance, lightweight, and cross-platform LLM runtime. 

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install_v2.sh | bash -s
```


### Step 2: Download the LLM model

Next, you'll need to obtain model files: the **Qwen2.5-VL-7B-Instruct model** and the **mmproj model**.

```
curl -LO https://huggingface.co/second-state/Qwen2.5-VL-7B-Instruct-GGUF/resolve/main/Qwen2.5-VL-7B-Instruct-Q5_K_M.gguf
curl -LO https://huggingface.co/second-state/Qwen2.5-VL-7B-Instruct-GGUF/resolve/main/Qwen2.5-VL-7B-Instruct-vision.gguf
```

### Step 3: Download a portable API server app 

Next, you need an application that can build and OpenAI-compatible API server for the Qwen 2.5 models
The [LlamaEdge api server app](https://github.com/LlamaEdge/LlamaEdge/tree/main/llama-api-server) is a lightweight and cross-platform Wasm app that works on any device
you might have. Just download the compiled binary app.

```
curl -LO https://github.com/second-state/LlamaEdge/releases/latest/download/llama-api-server.wasm
```

> The version of the `llama-api-server.wasm` should be v0.18.5 and above.

### Step 4: Chat with the chatbot UI 

The `llama-api-server.wasm` is a web server with an OpenAI-compatible API. You still need HTML files for the chatbot UI. It's optional and you can use `curl` to send an API request.
Download and unzip the HTML UI files as follows.

```
curl -LO https://github.com/LlamaEdge/chatbot-ui/releases/latest/download/chatbot-ui.tar.gz
tar xzf chatbot-ui.tar.gz
rm chatbot-ui.tar.gz
```

Then, start the web server.


```
wasmedge --dir .:. \
  --nn-preload default:GGML:AUTO:Qwen2.5-VL-7B-Instruct-Q5_K_M.gguf \
  llama-api-server.wasm \
  --model-name Qwen2.5-VL-7B-Instruct \
  --prompt-template qwen2-vision \
  --llava-mmproj Qwen2.5-VL-7B-Instruct-vision.gguf \
  --ctx-size 4096
```

> The above command line can work on a Macbook with 16GB memory.

Upon successful execution, you should see output similar to the following:

```
[2025-05-18 11:23:09.970] [info] llama_api_server in llama-api-server/src/main.rs:202: LOG LEVEL: info
[2025-05-18 11:23:09.973] [info] llama_api_server in llama-api-server/src/main.rs:205: SERVER VERSION: 0.18.5
[2025-05-18 11:23:09.976] [info] llama_api_server in llama-api-server/src/main.rs:544: model_name: Qwen2.5-VL-7B-Instruct

...

[2025-05-18 11:23:10.531] [info] llama_api_server in llama-api-server/src/main.rs:917: plugin_ggml_version: b5361 (commit cf0a43bb)
[2025-05-18 11:23:10.533] [info] llama_api_server in llama-api-server/src/main.rs:952: Listening on 0.0.0.0:8080
```

Then, go to `http://localhost:8080` on your computer to access the chatbot UI on a web page! You can upload an imange and chat with the model based on the image.

### Step 5: Send an API request

You can send an API request to call the model, which is more universal. The following command demonstrates how to send a CURL request to llama-api-server. The request includes a base64-encoded string of an image in the `image_url` field. For demonstration purposes, only a portion of the base64 string is shown here. In practice, you should use the complete base64 string.

> [!TIP]
> [base64.guru](https://base64.guru/converter/encode/image/jpg) provides a tool for encoding JPG to Base64.
> The Qwen 2.5 VL model supoort system prompt. So you can add a system prompt to guide the model behavior.

```bash
curl --location 'http://localhost:8080/v1/chat/completions' \
--header 'Content-Type: application/json' \
--data '{
    "messages": [
        {
            "role": "system",
            "content": "You are a helpful assistant that accurately describes the content of images provided by the user."
        },
        {
            "content": [
                {
                    "type": "text",
                    "text": "Describe the picture"
                },
                {
                    "type": "image_url",
                    "image_url": {
                        "url": "/9j/4AAQSkZJRgABAQAASABIAAD ... knr+Vb+lWR8oTTNwfujOc/hSuhuSsf//Z"
                    }
                }
            ],
            "role": "user"
        }
    ],
    "model": "Qwen2.5-VL-7B-Instruct"
}'

```

If the request is processed successfully, you will receive a response similar to the following:

```bash
{
    "id": "chatcmpl-4367085d-6451-4896-bbd8-a5090604394d",
    "object": "chat.completion",
    "created": 1747369554,
    "model": "Qwen2-VL-2B-Instruct",
    "choices": [
        {
            "index": 0,
            "message": {
                "content": "mixed berries in a paper bowl",
                "role": "assistant"
            },
            "finish_reason": "stop",
            "logprobs": null
        }
    ],
    "usage": {
        "prompt_tokens": 27,
        "completion_tokens": 8,
        "total_tokens": 35
    }
}
```


Congratulations! You have now started an multimodal app on your own device.

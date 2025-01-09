---
sidebar_position: 4
---

# Quick Start with Stable Diffusion models

Stable Diffusion is a state-of-the-art text-to-image generation model that creates high-quality images from text descriptions. With the [LlamaEdge stable-diffusion-api-server](https://github.com/LlamaEdge/sd-api-server), you can build an OpenAI-compatible API server for Stable Diffusion models.

> Want to use the recent Flux models? Check out [Run FLUX.1 [schnell] on your MacBook](https://www.secondstate.io/articles/flux1/).


### Install WasmEdge 

Start by installing WasmEdge version 0.14.1.

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install_v2.sh | bash -s -- -v 0.14.1
```
Choose the appropriate installation command based on your system:

**For Mac Apple Silicon**

```
# Download the stable diffusion plugin for Mac Apple Silicon
curl -LO https://github.com/WasmEdge/WasmEdge/releases/download/0.14.1/WasmEdge-plugin-wasmedge_stablediffusion-0.14.1-darwin_arm64.tar.gz

# Unzip the plugin to $HOME/.wasmedge/plugin
tar -xzf WasmEdge-plugin-wasmedge_stablediffusion-0.14.1-darwin_arm64.tar.gz -C $HOME/.wasmedge/plugin

rm $HOME/.wasmedge/plugin/libwasmedgePluginWasiNN.dylib
```

**For CUDA 12.0 (Ubuntu).** 

```
# Download the stable diffusion plugin for cuda 12.0
curl -LO https://github.com/WasmEdge/WasmEdge/releases/download/0.14.1/WasmEdge-plugin-wasmedge_stablediffusion-cuda-12.0-0.14.1-ubuntu20.04_x86_64.tar.gz

# Unzip the plugin to $HOME/.wasmedge/plugin
tar -xzf WasmEdge-plugin-wasmedge_stablediffusion-cuda-12.0-0.14.1-ubuntu20.04_x86_64.tar.gz -C $HOME/.wasmedge/plugin
```

**For CUDA 11.0 (Ubuntu).** 

```
# Download the stable diffusion plugin for cuda 11.0
curl -LO https://github.com/WasmEdge/WasmEdge/releases/download/0.14.1/WasmEdge-plugin-wasmedge_stablediffusion-cuda-11.3-0.14.1-ubuntu20.04_x86_64.tar.gz

# Unzip the plugin to $HOME/.wasmedge/plugin
tar -xzf WasmEdge-plugin-wasmedge_stablediffusion-cuda-11.3-0.14.1-ubuntu20.04_x86_64.tar.gz -C $HOME/.wasmedge/plugin
```

### Download the portable API server app

Download the server Wasm application. It's lightweight (the size of the server is 2.5 MB) and cross-platform.

```
curl -LO https://github.com/LlamaEdge/sd-api-server/releases/latest/download/sd-api-server.wasm
```

### Download the stable diffusion model


```
curl -LO https://huggingface.co/second-state/stable-diffusion-2-1-GGUF/resolve/main/v2-1_768-nonema-pruned-f16.gguf
```
For more stable diffusion models, go to our [Stable Diffusion Models Collection](https://huggingface.co/collections/second-state/stable-diffusion-models-677f9b3e01ac894463f4b326).

### Start the API server

Start the whisper API server with the following command line.

```
wasmedge --dir .:. sd-api-server.wasm --model-name sd-v2.1 --model v2-1_768-nonema-pruned-f16.gguf
```
The server will start on port 8080 by default.

### Use the API

Then, we can use the API server to generate an image.

```
curl -X POST 'http://localhost:8080/v1/images/generations' \
  --header 'Content-Type: application/json' \
  --data '{
      "model": "sd-v2.1",
      "prompt": "A cute baby cat"
  }'
```

Response example:

```
{"created":1736419627,"data":[{"url":"http://localhost:8080/v1/files/download/file_6420f32d-0b9a-4554-8e0b-a8deac0ab023","prompt":"A cute baby cat"}]}
```

You can view the generated image according to the prompt.

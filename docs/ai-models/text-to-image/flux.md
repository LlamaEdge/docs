---
sidebar_position: 4
---

# Run FLUX models with LlamaEdge


FLUX.1 is an open-source image generation model developed by Black Forest Labs, the creators of Stable Diffusion. With the [LlamaEdge stable-diffusion-api-server](https://github.com/LlamaEdge/sd-api-server), you can build an OpenAI-compatible API server for FLUX models.

> To run the FLUX models, make sure your machine has 1GGB RAM.


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

Download the API server application. It's a Wasm file, which is lightweight (the size of the server is 2.5 MB) and cross-platform.

```
curl -LO https://github.com/LlamaEdge/sd-api-server/releases/latest/download/sd-api-server.wasm
```

### Download the FLUX.1-dev model

We wil use FLUX.1-dev as an example in this tutorial. You can browse [the FLUX model collection](https://huggingface.co/collections/second-state/flux1-gguf-models-66f49625512c24f4b5b36cce) for more versiond of the FLUX model.

```
# Main model
curl -LO https://huggingface.co/second-state/FLUX.1-dev-GGUF/resolve/main/flux1-dev-Q4_0.gguf

# VAE file
curl -LO https://huggingface.co/second-state/FLUX.1-dev-GGUF/resolve/main/ae.safetensors

# clip_l encoder
curl -LO https://huggingface.co/second-state/FLUX.1-dev-GGUF/resolve/main/clip_l-Q8_0.gguf

# t5xxl encoder
curl -LO https://huggingface.co/second-state/FLUX.1-dev-GGUF/resolve/main/t5xxl-Q2_K.gguf
```


### Run the FLUX.1-dev model

Start the FLUX.1-dev API server with the following command line.

```
wasmedge --dir .:. sd-api-server.wasm \
  --model-name flux1-dev \
  --diffusion-model flux1-dev-Q4_0.gguf \
  --vae ae.safetensors \
  --clip-l clip_l-Q8_0.gguf \
  --t5xxl t5xxl-Q2_K.gguf
```

When you see [2024-01-01 16:48:45.462] [info] sd_api_server in src/main.rs:168: Listening on 0.0.0.0:8080 printed on the screen, the API server is ready.

### Use the API

Now, you can send an API request to generate images:

```
curl -X POST 'http://localhost:8080/v1/images/generations' \
  --header 'Content-Type: application/json' \
  --data '{
      "model": "flux1-dev",
      "prompt": "Astronaut, wearing futuristic astonaut outfit with space helmet, beautiful body and face, very breathtaking beautiful image, cinematic, 4k, epic Steven Spielberg movie still, sharp focus, emitting diodes, smoke, artillery, sparks, racks, system unit, motherboard, by pascal blanche rutkowski repin artstation hyperrealism painting concept art of detailed character design matte painting, 4 k resolution blade runner",
      "cfg_scale": 1.0,
      "sample_method": "euler",
      "steps": 20
  }'
```

If everything is set up correctly, a few minutes later the terminal will output the following

```
{"created":1736740551,"data":[{"url":"http://localhost:8080/v1/files/download/file_a17e7276-8202-4017-b433-e21b8c0a29c6","prompt":"Astronaut, wearing futuristic astonaut outfit with space helmet, beautiful body and face, very breathtaking beautiful image, cinematic, 4k, epic Steven Spielberg movie still, sharp focus, emitting diodes, smoke, artillery, sparks, racks, system unit, motherboard, by pascal blanche rutkowski repin artstation hyperrealism painting concept art of detailed character design matte painting, 4 k resolution blade runner"}]
```

Open `http://localhost:8080/v1/files/download/file_a17e7276-8202-4017-b433-e21b8c0a29c6` in your browser, and you wil see the generated image.


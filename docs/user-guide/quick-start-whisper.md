---
sidebar_position: 3
---

# Quick Start with Whisper


[Whisper](https://github.com/openai/whisper)     

With the [LlamaEdge whisper API server](https://github.com/LlamaEdge/whisper-api-server), you can build an OpenAI-compatible API server for the whisper model.

### Install WasmEdge

First off, you'll need WasmEdge along with the necessary plugin for whisper, open your terminal and execute the following command:

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install_v2.sh | bash -s
```

Next, install the `wasi-nn-whisper` plugin manually.  We're working to improve the installation experience. Stay tuned.

**For Mac Apple Silicon**

```
# Download the whisper plugin for Mac Apple Silicon
curl -LO https://github.com/WasmEdge/WasmEdge/releases/download/0.14.1/WasmEdge-plugin-wasi_nn-whisper-0.14.1-darwin_arm64.tar.gz

# Unzip the plugin to $HOME/.wasmedge/plugin
tar -xzf WasmEdge-plugin-wasi_nn-whisper-0.14.1-darwin_arm64.tar.gz -C $HOME/.wasmedge/plugin
```
**For CUDA 12.0 (Ubuntu)**

```
# Download the stable diffusion plugin for cuda 12.0
curl -LO https://github.com/WasmEdge/WasmEdge/releases/download/0.14.1/WasmEdge-plugin-wasi_nn-whisper-cuda-12.0-0.14.1-ubuntu20.04_x86_64.tar.gz

# Unzip the plugin to $HOME/.wasmedge/plugin
tar -xzf WasmEdge-plugin-wasi_nn-whisper-cuda-12.0-0.14.1-ubuntu20.04_x86_64.tar.gz -C $HOME/.wasmedge/plugin
```
**For CUDA 11.0 (Ubuntu) **

```
# Download the stable diffusion plugin for cuda 11.0
curl -LO https://github.com/WasmEdge/WasmEdge/releases/download/0.14.1/WasmEdge-plugin-wasi_nn-whisper-cuda-11.3-0.14.1-ubuntu20.04_x86_64.tar.gz

# Unzip the plugin to $HOME/.wasmedge/plugin
tar -xzf WasmEdge-plugin-wasi_nn-whisper-cuda-11.3-0.14.1-ubuntu20.04_x86_64.tar.gz -C $HOME/.wasmedge/plugin
```
For other release assets, please check out [the plugin release assets page](https://github.com/WasmEdge/WasmEdge/releases/tag/0.14.1).


### Download the portable API server app

Download the server Wasm application. It's lightweight (the size of the server is 3.7 MB) and cross-platform.

```
curl -LO https://github.com/LlamaEdge/whisper-api-server/releases/download/0.3.9/whisper-api-server.wasm
```

### Download the whisper model

You can browse and download the ggml model from https://huggingface.co/ggerganov/whisper.cpp/tree/main.
```
curl -LO https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-medium.bin
```

### Run the Whisper model

Start the whisper API server.

```
wasmedge --dir .:. whisper-api-server.wasm -m ggml-medium.bin
```

The server will start on port 8080 by default.

### Use the API

**Transcribe an audio file (Speech-to-Text)**

- Download a test audio file:

  ```bash
  curl -LO https://github.com/LlamaEdge/whisper-api-server/raw/main/data/test.wav
  ```

- Send a transcription request:

  ```bash
  curl --location 'http://localhost:8080/v1/audio/transcriptions' \
    --header 'Content-Type: multipart/form-data' \
    --form 'file=@"test.wav"'
  ```
 
 For non-English audio, specify the language as below. To check the language code, please refer to [List of ISO 639 language codes](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes).

 ```
curl --location 'http://localhost:8080/v1/audio/transcriptions' \
  --header 'Content-Type: multipart/form-data' \
  --form 'file=@"test.wav"' \
  --form 'language="ja"'
 ```
 
  
 Example response:

  ```json
  {
      "text": "[00:00:00.000 --> 00:00:03.540]  This is a test record for Whisper.cpp"
  }
  ```

#### Translate Audio (Speech-to-Text with Translation)

- Download a test audio file:

  ```bash
  curl -LO https://github.com/LlamaEdge/whisper-api-server/raw/main/data/test_cn.wav
  ```

  This audio contains a Chinese sentence, `这里是中文广播`, the English meaning is `This is a Chinese broadcast`.

- Send a translation request:

  ```bash
  curl --location 'http://localhost:8080/v1/audio/translations' \
    --header 'Content-Type: multipart/form-data' \
    --form 'file=@"test-cn.wav"' \
    --form 'language="cn"'
  ```

  Example response:

  ```json
  {
    "text": "[00:00:00.000 --> 00:00:04.000]  This is a Chinese broadcast."
  }
  ```

That's all! Use whisper to process your audio now!

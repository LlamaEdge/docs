---
sidebar_position: 3
---

# LlamaEdge vs llama.cpp

The llama.cpp project is one of the inference backends for LlamaEdge. LlamaEdge provides high level application
components to interact with AI models, such as encoding and decoding data,
managing prompts and contexts, knowledge supplement, and tool use. It simplifies how business applications could 
make use of the models. LlamaEdge and llama.cpp are complementary technologies.

In fact, LlamaEdge is designed to be agnostic to the underlying native runtimes. 
You can swap out llama.cpp for a different LLM
runtime, such as [Intel neural speed engine](https://github.com/WasmEdge/WasmEdge/issues/3260) and [Apple MLX runtime](https://github.com/WasmEdge/WasmEdge/issues/3266), without changing or even recompiling the application code.

Besides LLMs, LlamaEdge could support runtimes for other types of AI models, such as 
[stable diffusion](https://github.com/WasmEdge/WasmEdge/issues/3405), [Yolo](https://github.com/WasmEdge/WasmEdge/issues/2768), [whisper.cpp](https://github.com/WasmEdge/WasmEdge/issues/3287), and [Google MediaPipe](https://github.com/WasmEdge/mediapipe-rs).


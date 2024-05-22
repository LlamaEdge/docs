---
sidebar_position: 3
---

# LlamaEdge vs llama.cpp

LlamaEdge is built upon llama.cpp. So, why do you need the wrapper? Can you just write applications that directly
links and compiles against the llama.cpp's C++ API? Yes, you sure can. But ...

* LlamaEdge apps are written in Rust and compiled to portable Wasm. There is no need to incorporate and link against llama.cpp native library in your own app. You do not need a complex and fragile build system to manage cross-platform C++.
* LlamaEdge installer auto-detects the user's platform and installs the correct drivers and libraries. There is no need for you to create 100+ platform-dependent compile targets and then a complex installer to figure out where to install them.
* LlamaEdge provides high-level Rust components to handle application data on top of llama.cpp. Examples include prompt template management, context (RAG) management for prompts, API servers etc.

Finally, LlamaEdge is designed to be agnostic to the underlying native runtimes. You can swap out llama.cpp for a different LLM
runtime, such as [Intel neural speed engine](https://github.com/WasmEdge/WasmEdge/issues/3260) and [Apple MLX runtime](https://github.com/WasmEdge/WasmEdge/issues/3266), without changing or even recompiling the application code.

Besides LLMs, LlamaEdge is equipped by support runtimes for other types of AI models, such as 
[stable diffusion](https://github.com/WasmEdge/WasmEdge/issues/3405), [Yolo](https://github.com/WasmEdge/WasmEdge/issues/2768), [whisper.cpp](https://github.com/WasmEdge/WasmEdge/issues/3287), and [Google MediaPipe](https://github.com/WasmEdge/mediapipe-rs).


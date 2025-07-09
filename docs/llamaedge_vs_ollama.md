---
sidebar_position: 4
---

# LlamaEdge vs Ollama

There are several popular tools to run "local LLMs". Ollama is one of the earlist and most popular. Why do people
choose LlamaEdge over them?

* LlamaEdge is very small. The entire runtime and application is only 30MB. That is about 1/3 of the nearest competitor.
* LlamaEdge does not need root or sudo permissions. It does not install or run any daemon on your system. Hence LlamaEdge can be easily embedded into your own app.
* LlamaEdge apps are cross-platform. A single binary file can run on all supported OSes, CPUs, and GPUs. That also makes it simple to embed LlamaEdge in your apps.
* Through Docker integration, an LlamaEdge container combines model files, configurations, and runtime into a single package ensuring compatibility and portability over time. All from the Docker Hub you already use.
* LlamaEdge supports alternative runtimes beyond llama.cpp to achieve the most optimal performance for your model and hardware.
* LlamaEdge already supports multimodal vision models. It will soon support speech-to-text and text-to-image models through as OpenAI-compatible APIs.
* LlamaEdge is more secure. **LlamaEdge server is a sandboxed Wasm** and does not require root privileges. So it is much harder to exploit.

Finally, LlamaEdge is a developer platform. It provides Rust APIs and components for you to build your own applications.
It enables developers to create a single compact and cross-platform binary app that can be easily deployed and orchestrated across clouds.


---
sidebar_position: 4
---

# LlamaEdge vs Ollama

There are several popular tools to run "local LLMs". Ollama is one of the earlist and most popular. Why do people
choose LlamaEdge over them?

* LlamaEdge is very small. The entire runtime and application is only 30MB. That is about 1/3 of the nearest competitor.
* LlamaEdge does not need root or sudo permissions It does not install or run any daemon on your system. Hence LlamaEdge can be easily embedded into your own app.
* LlamaEdge works with model files you download from Huggingface. There is no need for a special download hub.
* LlamaEdge provides a fully featured API server that supports both chat and embedding models in OpenAI-compatible API formats.
* LlamaEdge already supports for multimodal vision models. It will soon support text-to-image, and voice models soon through a unified OpenAI-compatible API.

Finally, LlamaEdge is a developer platform. It provides Rust APIs and components for you to build your own applications.
It enables developers to create a single compact and cross-platform binary app that can be easily deployed and orchestrated across clouds.

* The [server-side RAG](user-guide/server-side-rag/quick-start) API server is built on LlamaEdge components.
* The [moxin](https://github.com/project-robius/moxin) LLM client app uses LlamaEdge as the embedded inference engine.
* The [GaiaNet](https://github.com/GaiaNet-AI/gaianet-node) project embeds LlamaEdge to run a large number of decentralized LLM agents across the web.
* The [Terminus OS](https://www.jointerminus.com/) project is a Kubernetes-based personal OS. It embeds LlamaEdge to power AI services such as local search and document QA.


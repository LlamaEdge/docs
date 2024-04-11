---
sidebar_position: 1
---

# Quickstart with one line of command

Enhance your onboarding experience and quickly get started with LlamaEdge using the following scripts.

## Quick start without any argument

```
bash <(curl -sSfL 'https://raw.githubusercontent.com/LlamaEdge/LlamaEdge/main/run-llm.sh')
```

It will download and start the Gemma-2b model automatically. Open http://127.0.0.1:8080 in your browser and start chatting right away!


## Specify a model

```
bash <(curl -sSfL 'https://raw.githubusercontent.com/LlamaEdge/LlamaEdge/main/run-llm.sh') --model llama-2-7b-chat
```

The script will start an API server with a chatbot UI based on your choice. Open http://127.0.0.1:8080 in your browser and start chatting right away!

To explore all the available models, please use the following command line

```
bash <(curl -sSfL 'https://raw.githubusercontent.com/LlamaEdge/LlamaEdge/main/run-llm.sh') --model help
```
## Interactively choose and confirm all steps

```
bash <(curl -sSfL 'https://raw.githubusercontent.com/LlamaEdge/LlamaEdge/main/run-llm.sh') --interactive
```

Follow the on-screen instructions to install the WasmEdge Runtime and download your favorite open-source LLM. Then, choose whether you want to chat with the model via the CLI or via a web UI.

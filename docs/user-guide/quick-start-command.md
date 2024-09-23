---
sidebar_position: 1
---

# Quick Start

LlamaEdge is a suite of component libraries and command line tools for developers to embed and run LLMs in their own apps. The best way to quickly experience LlamaEdge is to use easy-to-use utilities built on top of it.

## Quick Start with Gaia

Gaia is an integrated tool for running open-source LLMs. It is built on LlamaEdge. Following these simple commands, you will be able to get an Internet-accessible chatbot and an OpenAI-compatible API server running on your devices using any open-source model you choose in a few minutes.

Install the Gaia software with a single command on Mac, Linux, or Windows WSL.

```bash
curl -sSfL 'https://github.com/GaiaNet-AI/gaianet-node/releases/latest/download/install.sh' | bash
```

Then, follow the prompt on your screen to set up the environment path. The command line will begin with `source`.

Use `gaianet init` to download the model files and vector database files specified in the `$HOME/gaianet/config.json` file, and it could take a few minutes since the files are large.

```bash
gaianet init
```

> The default `$HOME/gaianet/config.json` runs a Phi 3.5 LLM and a nomic-embed embedding model. You can easily [switch to a Llama 3.1 8b LLM by giving a different configuration](https://github.com/GaiaNet-AI/node-configs/tree/main/llama-3.1-8b-instruct) to `gaianet init`. Configurations for many more LLMs are [available here](https://github.com/GaiaNet-AI/node-configs).

Start running the Gaia node.

```bash
gaianet start
```

Once it starts on your machine, you can simply go to `http://localhost:8080`. You can open a browser to that URL to see the node information and then chat with the LLM. This node API server also supports `v1/chat/completions` and `v1/embeddings` endpoints, fully compatible with OpenAI APIs.

If you are running it on a server or need to access the LLM sevices from the Internet, the Gaia node has automatically set up connection tunneling for you. The script prints the Internet address for the LLM service on the console as follows.

```
... ... https://0xf63939431ee11267f4855a166e11cc44d24960c0.us.gaianet.network
```

To stop running the LLM services, you can run the following script.

```bash
gaianet stop
```

If you're looking to configure LLMs further, explore the details [here](https://docs.gaianet.ai/category/node-operator-guide).

## Quick start with Moxin

Moxin is a cross-platform LLM client written in Rust, and built on LlamaEdge components. It offers an intuitive UI for running LLMs with just a few clicks.

Download the Moxin app install package for your device from the [Moxin website](https://www.moxin.app/). Here's how to get started on macOS:

* Download and install the `dmg` file from https://www.moxin.app/ on your Macbook.
* Browse model cards and choose one model to download after open the Moxin app. As models are quite large, this may take several minutes.
* Engage with the model via a simple and interactive chat interface.

![](quick-start-command-01.png)


---
sidebar_position: 1
---

# Quick Start

Elevate your onboarding experience and seamlessly begin with LlamaEdge using the following tools. Rather than utilizing LlamaEdge command lines, we’ll leverage the developer tools built atop LlamaEdge for a more intuitive and efficient workflow.

## Quick Start with Gaia

Gaia is a CLI tool designed for running open-source LLMs on your own machine.

Install the Gaia software stack effortlessly with a single command, whether on Mac, Linux, or Windows WSL.

```bash
curl -sSfL 'https://github.com/GaiaNet-AI/gaianet-node/releases/latest/download/install.sh' | bash
```
Then, follow the prompt on your screen to set up the environment path. The command line will begin with `source`.

Use `gaianet init` to download the model files and vector database files specified in the `$HOME/gaianet/config.json` file, and it could take a few minutes since the files are large.

```bash
gaianet init
```

Start running both the chat model like Phi-3-mini and the embedding model like nomic-embed.

```bash
gaianet start
```

The script prints the LLM service  address on the console as follows.
You can open a browser to that URL to see the node information and then chat with the LLM. This address also includes `v1/chat/completions` and `v1/embeddings` endpoints, fully compatible with OpenAI APIs.

```
... ... https://0xf63939431ee11267f4855a166e11cc44d24960c0.us.gaianet.network
```

To stop running the LLM services, you can run the following script.

```bash
gaianet stop
```

If you're looking to configure LLMs further, explore the details [here](https://docs.gaianet.ai/category/node-operator-guide).

## Quick start with Moxin

Moxin is an AI LLM client written in Rust, offering an intuitive UI for running LLMs with just a few clicks.

Simply download the relevant package from the [Moxin website](https://www.moxin.app/) for your platform. Here's how to get started on macOS:

* Download and install the `dmg` file from https://www.moxin.app/ on your Macbook.
* Browse model cards and choose one model to download after open the Moxin app. As models are quite large, this may take several minutes.
* Engage with the model via a simple and interactive chat interface.

![](quick-start-command-01.png)


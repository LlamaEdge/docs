---
sidebar_position: 2
---

# LlamaEdge vs Python

Most AI models are trained and even finetuned in Python / PyTorch, but you should not deploy and run them in Python. 
In fact, running production level AI inference in Python is extremely inefficient -- a natively compiled language
can be [35,000x faster than Python](https://www.modular.com/blog/how-mojo-gets-a-35-000x-speedup-over-python-part-1).
Developers choose LlamaEdge over Python because:

* LlamaEdge is only 1/100 the size of a Python runtime. Do you know that the smallest PyTorch Docker image is [almost 4GB](https://hub.docker.com/r/pytorch/pytorch/tags)?
* LlamaEdge is a single install package with no complex dependencies. It is very easy to install and get started. It does not take the [best minds of our generation](https://twitter.com/santiviquez/status/1676677829751177219) to install it.
* Developers can create LlamaEdge apps in Rust, which is much faster than Python in pre and post processing data that goes into the model. A good example is the [LlamaEdge chatbot and API server](https://github.com/LlamaEdge/LlamaEdge/tree/main/api-server) -- it is orders of magnitudes faster than Python-based web app servers.

Learn more: [Why did Elon Musk say that Rust is the Language of AGI?](https://blog.stackademic.com/why-did-elon-musk-say-that-rust-is-the-language-of-agi-eb36303ce341) 

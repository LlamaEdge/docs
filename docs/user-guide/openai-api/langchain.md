---
sidebar_position: 7
---

# LangChain

In this tutorial, I will introduce you how to build a client-side RAG using Llama2-7b-chat model, based on LlamaEdge and Langchain.

> LlamaEdge has [recently became](https://twitter.com/realwasmedge/status/1742437253107130552) an official inference backend for LangChain, allowing LangChain applications to run open source LLMs on heterogeneous GPU devices. 

### Build the client app using Langchian with vector DB support

First, let's build a chatbot web app using Langchain. This part will be built in Python. The app includes uploading file and attaches the Chroma DB and the gpt4all embedding algorithms.

To quick start, fork or clone [the wasm-llm repo](https://github.com/second-state/wasm-llm) and open the wasm-bot folder.

```
git clone https://github.com/second-state/wasm-llm.git
cd wasm-llm/wasm-rag-service
```

Next, let’s install the required python dependencies for this program. We will use conda to control the version and environment.

Follow the [miniconda installation instruction](https://docs.conda.io/projects/miniconda/en/latest/#quick-command-line-install) to install mini conda your own machine. After that, create a conda environment for the chatbot web app. Let’s use chatbot as the name.


```
conda create -n wasm-rag python=3.11
conda activate wasm-rag
```

Then, you may notice that your terminal has entered the `chatbot` environment. Let’s [install the dependencies for this chatbot app](https://github.com/second-state/wasm-llm/blob/main/wasm-bot/requirements.txt). All the dependencies are included in the `requirements.txt`.


```
pip install -r requirements.txt
```

With all dependencies installed, then we can execute the chatbot app.

```
streamlit run app.py
```


If everything goes well, you will see the following messages on your terminal. In the meanwhile,  a web page will be opened in your browser.

```
You can now view your Streamlit app in your browser.
Local URL: http://localhost:8501
Network URL: http://192.168.0.103:8501
```

![image](https://github.com/LlamaEdge/docs/assets/45785633/af418d8e-9377-4613-b976-4ed3bec1836c)

Now, we have completed the first part — a RAG client app waiting for a LLM backend to answer user’s question.


### Build an OpenAI compatible API server for the open source LLM using LlamaEdge


Let’s build a API server for the open source LLM with WasmEdge.

First, install WasmEdge runtime with one single command line.

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- --plugin wasi_nn-ggml
```

Second, download the model file in GGUF file. Here, I use llama2-7b as an example. We have tried several LLMs and concluded that Llama2-7B is the beat for RAG applications.

```
curl -LO https://huggingface.co/second-state/Llama-2-7B-Chat-GGUF/resolve/main/Llama-2-7b-chat-hf-Q5_K_M.gguf
```

Third, download an API server app. It is a cross-platform portable Wasm app that can run on many CPU and GPU devices. 


```
curl -LO https://github.com/second-state/LlamaEdge/releases/latest/download/llama-api-server.wasm
```


Finally, use the following command lines to start an API server for the model.  If you have did the above steps, just run the follwing command line.


```
wasmedge --dir .:. --nn-preload default:GGML:AUTO:Llama-2-7b-chat-hf-Q5_K_M.gguf llama-api-server.wasm -p llama-2-chat -c 4096 -m NA
```


If everything goes well, the following information will be printed on the terminal.

```
[INFO] Socket address: 0.0.0.0:8080
[INFO] Model name: default
[INFO] Model alias: default
[INFO] Prompt context size: 512
[INFO] Number of tokens to predict: 1024
[INFO] Number of layers to run on the GPU: 100
[INFO] Batch size for prompt processing: 512
[INFO] Temperature for sampling: 0.8
[INFO] Penalize repeat sequence of tokens: 1.1
[INFO] Prompt template: HumanAssistant
[INFO] Log prompts: false
[INFO] Log statistics: false
[INFO] Log all information: false
[INFO] Starting server ...
ggml_init_cublas: GGML_CUDA_FORCE_MMQ:   no
ggml_init_cublas: CUDA_USE_TENSOR_CORES: yes
ggml_init_cublas: found 1 CUDA devices:
  Device 0: Orin, compute capability 8.7, VMM: yes
[INFO] Plugin version: b1953 (commit 6f9939d1)
[INFO] Listening on http://0.0.0.0:8080
```

Now the Llama2-7B-Chat model is hosted at the port of 8080.


### Connect your self-hosted LLMs with the chatbot web app


Go back to the web page opened in the first step. Click Use cusom service on the bottom left of page and click the Connect button.
Then you will see a section to upload your own data locally.  Upload a pdf file here. When the uploading process is done, the bot will send you a message: “Hello 👋, how can I help you?”,  which is a ready sign.

Ask a question, and the bot will reply to you based on the file you uploaded.

![image](https://github.com/LlamaEdge/docs/assets/45785633/0b5273f6-7edd-4fcf-b917-c5931609c5db)

### What’s next?

We will introduce you how to build such a client-side RAG app with OpenWebui



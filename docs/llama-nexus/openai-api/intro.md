---
sidebar_position: 1
---

# Start an LlamaEdge API service

Since LlamaEdge provides an OpenAI-compatible API service, it can be a drop-in replacement for OpenAI in almost all LLM applications and frameworks. 
Checkout the articles in this section for instructions and examples for how to use locally hosted LlamaEdge API services in popular LLM apps.

## Start the API servers for multiple models

First, you will need to start an OpenAI compatible API server.

* Start an OpenAI compatible API server for Large Language Models (LLM)
➔ [Get Started with LLM](/docs/category/llm)


* Start an OpenAI compatible API server for Whisper
➔ [Get Started with Speech to Text](/docs/category/speech-to-text)

* Start an OpenAI compatible API server for GPT-SOVITs and Piper
➔ [Get Started with Text to Speech](/docs/category/text-to-speech)

* Start an OpenAI compatible API server for Stable Diffusion and FLUX
➔ [Get Started with Text-to-Image](/docs/category/text-to-image)

* Start an OpenAI compatible API server for Llava and Qwen-VL
➔ [Get Started with Multimodal](/docs/category/multimodal)


## OpenAI replacement

Now, you can ready to use this API server in OpenAI ecosystem apps as a drop-in replacement for the OpenAI API!
In general, for any OpenAI tool, you could just replace the following.

|Config option | Value | Note |
|-----|--------|-------|
| API endpoint URL | `http://localhost:8080/v1` | If the server is accessible from the web, you could use the public IP and port |
| Model Name (for LLM) | `llama-3-8b-chat` | The first value specified in the `--model-name` option |
| Model Name (for Text embedding) | `nomic-embed` | The second value specified in the `--model-name` option |
| API key | Empty | Or any value if the app does not permit empty string |

## The OpenAI Python library

You can install the [official OpenAI Python library](https://pypi.org/project/openai/) as follows.

```
pip install openai
```

When you create an OpenAI client using the library, you can pass in the API endpoint point as the `base_url`.

```
import openai

client = openai.OpenAI(base_url="http://localhost:8080/v1", api_key="")
```

Alternatively, you could set an environment variable at the OS level.

```
export OPENAI_API_BASE=http://localhost:8080/v1
```

Then, when you make API calls from the `client`, make sure that the `model` is set to the model name
available on your node.

```
response = client.chat.completions.create(
    model="llama-3-8b-chat",
    messages=[
        {"role": "system", "content": "You are a strategic reasoner."},
            {"role": "user", "content": "What is the purpose of life?"}
        ],
        temperature=0.7,
        max_tokens=500
    ]
)
```

That's it! You can now take any application built with the official OpenAI Python library and use your own
LlamaEdge device as its backend!


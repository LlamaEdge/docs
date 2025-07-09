---
sidebar_position: 1
---

# Start the llama-nexus API service

Since LlamaEdge provides an OpenAI-compatible API service, it can be a drop-in replacement for OpenAI in almost all LLM applications and frameworks. 
You can start LlamaEdge API servers for individual AI models, and use llama-nexus to combine multiple AI models
into a single API server.


## OpenAI replacement

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


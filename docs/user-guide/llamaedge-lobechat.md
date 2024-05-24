---
sidebar_position: 5
---

# LobeChat + LlamaEdge

The LobeChat framework supports OpenAI providers, and since LlamaEdge is compatible with the OpenAI API, you can use LlamaEdge as the backend Large Language Model (LLM) API. This integration supports:

* The hosted LobeChat service
* Any product built on the open-source LobeChat framework

## Prerequisites

Before proceeding, follow the [Getting Started with LlamaEdge guide](get-started-with-llamaedge.md) to run an open-source LLM locally.

## Steps to integrate LobeChat and LlamaEdge

Open the [LobeChat Language Model setting page](https://chat-preview.lobehub.com/settings/modal?agent=&session=inbox&tab=llm&topic=CIfo1UYZ) and choose the first one OpenAI.

First, Enter some random characters in the OpenAI API Key field, and input `http://localhost:8080/v1` in the API Proxy Address field.

Then, enable the Client-Side Fetching Mode. 

Next, Click on the Get Model List button to automatically detect the model you're using. Select that model from the list.

Finally, you can click on the Check button to check the connect status.

![](lobechat-llamaedge-01.png)


After that, go back to [the chat page](https://chat-preview.lobehub.com/chat?session=inbox&agent=) and choose the model you just chose in the previous step. Now you can chat with the model via LobeChat.

![](lobechat-llamaedge-02.png)


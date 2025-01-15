---
sidebar_position: 3
---

# Calling external tools

Tool calling is one of the truly "LLM native" interaction modes that has never existed before. 
It gives the "thinking" LLMs the ability to "act" -- both in acquiring new knowledge and in performing real world actions. It is a crucial part of any agentic application.

Open source LLMs are increasingly good at using tools. The Llama 3 models have now made it possible to have reliable tool calling performance on 8b class of LLMs running on your own laptop!

In this tutorial, we will show you a simple Python program that allows a local LLM to run code and manipulate data on the local computer!


## Prerequisites

Follow [this guide](/docs/openai-api/intro.md) to start an LlamaEdge API server. 
For example, we will need an open source model that is capable of tool calling. 
The Groq-tuned Llama 3 8B model is a good choice. Let's download the model file. 

```
curl -LO https://huggingface.co/second-state/Llama-3-Groq-8B-Tool-Use-GGUF/resolve/main/Llama-3-Groq-8B-Tool-Use-Q5_K_M.gguf
```

Then start the LlamaEdge API server for this model as follows. 

```
wasmedge --dir .:. \
    --nn-preload default:GGML:AUTO:Llama-3-Groq-8B-Tool-Use-Q5_K_M.gguf \
    --nn-preload embedding:GGML:AUTO:nomic-embed-text-v1.5.f16.gguf \
    llama-api-server.wasm \
    --model-alias default,embedding \
    --model-name llama-3-groq-8b,nomic-embed \
    --prompt-template groq-llama3-tool,embedding \
    --batch-size 128,8192 \
    --ctx-size 8192,8192
```

Note the `groq-llama3-tool` prompt template. It constructs user queries and LLM responses, including JSON messages for tool calls, into proper formats that the model is finetuned to follow. 

> You can [start a Gaia node](https://github.com/GaiaNet-AI/node-configs/tree/main/llama-3-groq-8b-tool) for the Llama-3-Groq model. You can then use the node's API URL endpoint and model name in your tool call apps.

## Run the demo agent

The [agent app](https://github.com/second-state/llm_todo) is written in Python. It demonstrates how the LLM could use tools to operate a SQL database. In this case, it starts and operates an in-memory SQLite database. The database stores a list of todo items. 

Download the code and install the Python dependencies as follows. 

```
git clone https://github.com/second-state/llm_todo
cd llm_todo
pip install -r requirements.txt
```

Set the environment variables for the API server and model name we just set up. 

```
export OPENAI_MODEL_NAME="llama-3-groq-8b"
export OPENAI_BASE_URL="http://127.0.0.1:8080/v1"
```

Run the `main.py` application and bring up the command line chat interface. 

```
python main.py
```

## Use the agent

Now, you can ask the LLM to perform tasks. For example, you can say 

```
User: 
Help me to write down it I'm going to fix a bug
```

The LLM understands that you need to insert a record into the database and returns a tool call response in JSON. 

```
Assistant:
<tool_call>
{"id": 0, "name": "create_task", "arguments": {"task": "going to fix a bug"}}
</tool_call>
```

The agent app (i.e., `main.py`) executes the tool call `create_task` in the JSON response, and sends back the results as role `Tool`. You do not need to do anything here as it happens automatically in `main.py`. The SQLite database is updated when the agent app executes the tool call. 

```
Tool:
[{'result': 'ok'}]
```

The LLM receives the execution result and then answers you. 

```
Assistant:
I've added "going to fix a bug" to your task list. Is there anything else you'd like to do?
```

You can continue the conversation. 

To learn more about how tool calling works, see [this article](https://github.com/LlamaEdge/LlamaEdge/blob/main/api-server/ToolUse.md).


## Code walkthrough

The `main.py` script serves as a great example to show the anatomy of a tool call application. 

First, there is the `Tools` JSON structure that defines the available tools. Each tool is designed as a function, with a function name and a set of parameters. The `description` field is especially important. It explains when and how the tool should be used. The LLM "understands" this description and uses it to determine whether this tool should be used to respond to a user query. The LLM will include those function names in its tool call responses when needed. 

```
Tools = [
    {
        "type": "function",
        "function": {
            "name": "create_task",
            "description": "Create a task",
            "parameters": {
                "type": "object",
                "properties": {
                    "task": {
                        "type": "string",
                        "description": "Task's content",
                    }
                },
            },
        },
    },
    ... ...
]
```

Then, the `eval_tools()` function maps the tool function names and parameters in the LLM JSON responses to actual Python functions that need to be executed. 

```
def eval_tools(tools):
    result = []
    for tool in tools:
        fun = tool.function
        if fun.name == "create_task":
            arguments = json.loads(fun.arguments)
            result.append(create_task(arguments["task"]))
        ... ...

    if len(result) > 0:
        print("Tool:")
        print(result)

    return result
```

The Python functions perform CURD database operations as you would expect. 

```
def create_task(task):
    try:
        conn.execute("INSERT INTO todo (task, status) VALUES (?, ?)", (task, "todo"))
        conn.commit()
        return {"result": "ok"}
    except Exception as e:
        return {"result": "error", "message": str(e)}
```

With the tool call functions defined both in JSON and Python, we can now look into how the agent manages the conversation. The user query is sent through the `chat_completions` function. 

```
def chat_completions(messages):
    stream = Client.chat.completions.create(
        model=MODEL_NAME,
        messages=messages,
        tools=Tools,
        stream=True,
    )

    tool_result = handler_llm_response(messages, stream)
    if len(tool_result) > 0:
        for result in tool_result:
            messages.append({"role": "tool", "content": json.dumps(result)})
        return False
    else:
        return True
```

When it receives a response, it calls `handler_llm_response()` to determine if the LLM response requires tool call. If tool call is not needed, the LLM response is simply displayed to the user. 

But if a tool call JSON section is present in the LLM response, the `handler_llm_response()` function is responsible of executing it by calling the associated Python function. Each tool call execution result is automatically sent back to the LLM as a message with the `tool` role. The LLM will then use these `tool` result messages to generate a new response. 

```
def handler_llm_response(messages, stream):
    tools = []
    content = ""
    print("Assistant:")
    for chunk in stream:
        if len(chunk.choices) == 0:
            break
        delta = chunk.choices[0].delta
        print(delta.content, end="")
        content += delta.content
        if len(delta.tool_calls) == 0:
            pass
        else:
            if len(tools) == 0:
                tools = delta.tool_calls
            else:
                for i, tool_call in enumerate(delta.tool_calls):
                    if tools[i] == None:
                        tools[i] = tool_call
                    else:
                        argument_delta = tool_call["function"]["arguments"]
                        tools[i]["function"]["arguments"].extend(argument_delta)
    if len(tools) == 0:
        messages.append({"role": "assistant", "content": content})
    else:
        tools_json = [tool.json() for tool in tools]
        messages.append(
            {"role": "assistant", "content": content, "tool_call": tools_json}
        )

    print()

    return eval_tools(tools)
```

## Make it robust 

One of the key challenges for LLM apps is that LLM responses are often unreliable. What if

*The LLM fails to generate a correct tool call response that is required to answer the user query.*

In this case, you could adjust and finetune the description for each tool call function. The LLM selects its tools based on those descriptions. Writing descriptions to match common user queries is essential. 

*The LLM hallucinates and generate tool calls with non-existent function names or wrong parameters.* 

The agent app should capture this error and ask the LLM to re-generate a response. If the LLM cannot generate a valid tool call response, the agent could answer something like 

[I'm sorry Dave, I'm afraid I can't do that](https://www.youtube.com/watch?v=5lsExRvJTAI)

*The LLM generates malformatted JSON structures for tools.*

Same as above. The agent should capture and handle the error. 

Tool calling is a key feature of the nascent field of agentic LLM apps. We cannot wait to see what you come up with!


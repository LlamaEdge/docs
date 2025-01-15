---
sidebar_position: 2
---

# Create a chatbot LLM app

The most common LLM app has to be the chatbot. For that, the base LLM is finetuned with a lot of back and forth conversation examples. The base LLM "learns" how to follow conversations and becomes a chat LLM. Since the conversation examples are fed into the LLM using certain formats, the chat LLM will expect the input prompt to follow the same format. This is called the prompt template. Let's see how that works.

## Build and run

First, let's get the source code.

```
git clone https://github.com/second-state/WasmEdge-WASINN-examples
cd WasmEdge-WASINN-examples
cd wasmedge-ggml/llama
```

Next, build it using the Rust `cargo` tool.

```
cargo build --target wasm32-wasip1 --release
cp target/wasm32-wasip1/release/wasmedge-ggml-llama.wasm .
```

Download a chat LLM.

```
curl -LO https://huggingface.co/second-state/Llama-2-7B-Chat-GGUF/resolve/main/Llama-2-7b-chat-hf-Q5_K_M.gguf
```

Run it! We load the LLM under the name `default` and then ask the `wasmedge-ggml-llama.wasm` app to run the `default` model.

```
wasmedge --dir .:. \
  --nn-preload default:GGML:AUTO:Llama-2-7b-chat-hf-Q5_K_M.gguf \
  wasmedge-ggml-llama.wasm default
```

You can now converse with it on the command line.

## The prompt template

The prompt to the Llama2 LLM must follow the exact same template format it was finetuned on. It is as follows. As you can see, there is a "system prompt" and followed by back-and-forth conversations, ending with the user's new question or prompt. When the LLM answers, we can simply append the answer to the end of the prompt, and then put the next question in `[INST]...[/INST]`.

```
<s>[INST] <<SYS>>
You are a helpful assistant. Be polite!
<</SYS>>

My first question? [/INST] The first answer. </s><s>[INST] My second question? [/INST] The second answer.</s><s>[INST] My third question? [/INST]
```

> Llama2 is just one of the prompt templates for chat. We also have examples for the [chatml template](https://github.com/second-state/WasmEdge-WASINN-examples/tree/master/wasmedge-ggml/chatml) and the [gemma template](https://github.com/second-state/WasmEdge-WASINN-examples/tree/master/wasmedge-ggml/gemma).

## Code walkthrough

The source code of this project is [here](https://github.com/second-state/WasmEdge-WASINN-examples/blob/master/wasmedge-ggml/llama/src/main.rs). It starts the execution context with `options` and sends in the prompt to `compute()`.

```
let graph = GraphBuilder::new(GraphEncoding::Ggml, ExecutionTarget::AUTO)
    .config(serde_json::to_string(&options).expect("Failed to serialize options"))
    .build_from_cache(model_name)
    .expect("Failed to build graph");
let mut context = graph
    .init_execution_context()
    .expect("Failed to init context");

... ...

let tensor_data = prompt.as_bytes().to_vec();
context.set_input(0, TensorType::U8, &[1], &tensor_data).expect("Failed to set input");
context.compute().expect("Failed to compute");
let output = get_output_from_context(&context);
println!("{}", output.trim());
```

The interesting part, however, is how we construct the prompt. It starts with the system prompt.

```
let mut saved_prompt = String::new();
let system_prompt = String::from("You are a helpful, respectful and honest assistant. Always answer as short as possible, while being safe." );
```

Then, in the question and answer loop, we will append the question, run the inference, and then append the answer to the prompt according to the template.

```
loop {
    let input = read_input();
    if saved_prompt.is_empty() {
        saved_prompt = format!(
            "[INST] <<SYS>> {} <</SYS>> {} [/INST]",
            system_prompt, input
        );
    } else {
        saved_prompt = format!("{} [INST] {} [/INST]", saved_prompt, input);
    }

    ... ...

    match context.compute() {
        ... ....
    }
    let mut output = get_output_from_context(&context);
    println!("ASSISTANT:\n{}", output.trim());

    // Update the saved prompt.
    output = output.trim().to_string();
    saved_prompt = format!("{} {}", saved_prompt, output);
}
```

## Streaming response

An important usability feature of chatbot apps is to stream LLM responses back to the user. LlamaEdge provides APIs that allow the application to retrieve the LLM responses one word at a time. We have a [complete example here](https://github.com/second-state/WasmEdge-WASINN-examples/blob/master/wasmedge-ggml/llama-stream/). Instead of calling `compute()` on the execution context, you should call `compute_single()` instead. The following code retrieves the response one token at a time in a loop and prints the token as it arrives.

```
println!("ASSISTANT:");
loop {
    match context.compute_single() {
        ... ...
    }
    // Retrieve the single output token and print it.
    let token = get_single_output_from_context(&context);
    print!("{}", token);
    io::stdout().flush().unwrap();
    }
    println!();
}
```

The `get_single_output_from_context()` helper function calls the new API function `get_output_single()` on the execution context.

```
fn get_single_output_from_context(context: &GraphExecutionContext) -> String {
    get_data_from_context(context, 0, true)
}

fn get_data_from_context(context: &GraphExecutionContext, index: usize, is_single: bool) -> String {
    // Preserve for 4096 tokens with average token length 6
    const MAX_OUTPUT_BUFFER_SIZE: usize = 4096 * 6;
    let mut output_buffer = vec![0u8; MAX_OUTPUT_BUFFER_SIZE];
    let mut output_size = if is_single {
        context
            .get_output_single(index, &mut output_buffer)
            .expect("Failed to get single output")
    } else {
        context
            .get_output(index, &mut output_buffer)
            .expect("Failed to get output")
    };
    output_size = std::cmp::min(MAX_OUTPUT_BUFFER_SIZE, output_size);

    return String::from_utf8_lossy(&output_buffer[..output_size]).to_string();
}
```

That's it!

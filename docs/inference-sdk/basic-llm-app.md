---
sidebar_position: 1
---

# Create a basic LLM app

At the most basic level, the LLM completes text. That is why the input text is called a "prompt". The base model simply comes up with the next words that are likely to follow the prompt. In this example, we will demonstrate this basic use case.

## Build and run

First, let's get the source code.

```
git clone https://github.com/second-state/WasmEdge-WASINN-examples
cd WasmEdge-WASINN-examples
cd wasmedge-ggml/basic
```

Next, build it using the Rust `cargo` tool.

```
cargo build --target wasm32-wasip1 --release
cp target/wasm32-wasip1/release/wasmedge-ggml-basic.wasm .
```

Download a non-chat LLM. This one a code completion model. You give it a request and it will respond with code that meets your request.

```
curl -LO https://huggingface.co/second-state/StarCoder2-7B-GGUF/resolve/main/starcoder2-7b-Q5_K_M.gguf
```

Run it! We load the LLM under the name `default` and then ask the `wasmedge-ggml-basic.wasm` app to run the `default` model.

```
wasmedge --dir .:. \
  --env n_predict=100 \
  --nn-preload default:GGML:AUTO:starcoder2-7b-Q5_K_M.gguf \
  wasmedge-ggml-basic.wasm default
```

Try a few examples. All those examples are to prompt the LLM to write code and complete the requested tasks.

```
USER:
def print_hello_world():

USER:
fn is_prime(n: u64) -> bool {

USER:
Write a Rust function to check if an input number is prime:
```

## Source code walkthrough

The Rust source code for this example is [here](https://github.com/second-state/WasmEdge-WASINN-examples/blob/master/wasmedge-ggml/basic/src/main.rs). The first omportant step in `main()` is to create an execution context. The `config()` function takes an `options` struct that provides inference options for the model, such as context length, temperature etc. You can check the `get_options_from_env()` function in the source code to see how the `options` is constructed.

> The `model_name` is `default`, which correspond to the model name in `--nn-preload`.

```
let graph = GraphBuilder::new(GraphEncoding::Ggml, ExecutionTarget::AUTO)
    .config(serde_json::to_string(&options).expect("Failed to serialize options"))
    .build_from_cache(model_name)
    .expect("Failed to build graph");
let mut context = graph
    .init_execution_context()
    .expect("Failed to init context");
```

Next, we simply pass the request prompt text to the execution context as a byte array, and call `compute()`.

```
let tensor_data = prompt.as_bytes().to_vec();
context.set_input(0, TensorType::U8, &[1], &tensor_data).expect("Failed to set input");
context.compute().expect("Failed to compute");
```

Finally, you simply get the computed output from the execution context, and print it as a string.

```
let output = get_output_from_context(&context);
println!("{}", output.trim());
```

The above helper function `get_output_from_context()` uses a buffer to read data from the context.

```
fn get_data_from_context(context: &GraphExecutionContext, index: usize) -> String {
    // Preserve for 4096 tokens with average token length 6
    const MAX_OUTPUT_BUFFER_SIZE: usize = 4096 * 6;
    let mut output_buffer = vec![0u8; MAX_OUTPUT_BUFFER_SIZE];
    let mut output_size = context
        .get_output(index, &mut output_buffer)
        .expect("Failed to get output");
    output_size = std::cmp::min(MAX_OUTPUT_BUFFER_SIZE, output_size);

    return String::from_utf8_lossy(&output_buffer[..output_size]).to_string();
}
```

That's it!

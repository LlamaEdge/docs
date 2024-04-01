---
sidebar_position: 4
---

# Create an embedding app

An important LLM task is to generate embeddings for natural language sentences. It converts a sentence to a vector of numbers called an "embedding". The embedding vectors can then be stored in a vector database. You can search it later to find similiar sentences.

## Build and run

First, let's get the source code.

```
git clone https://github.com/second-state/WasmEdge-WASINN-examples
cd WasmEdge-WASINN-examples
cd wasmedge-ggml/embedding
```

Next, build it using the Rust `cargo` tool.

```
cargo build --target wasm32-wasi --release
cp target/wasm32-wasi/release/wasmedge-ggml-llama-embedding.wasm .
```

Download an embedding model.

```
curl -LO https://huggingface.co/second-state/All-MiniLM-L6-v2-Embedding-GGUF/resolve/main/all-MiniLM-L6-v2-ggml-model-f16.gguf
```

Run it! We load the embedding model under the name `default` and then ask the `wasmedge-ggml-llama-embedding.wasm` app to run the `default` model.

```
$ wasmedge --dir .:. \
  --nn-preload default:GGML:AUTO:all-MiniLM-L6-v2-ggml-model-f16.gguf \
  wasmedge-ggml-llama-embedding.wasm default
```

Now, you can enter a prompt and the program will use the embedding model to generate the embedding vector for you!

```
Prompt:
What's the capital of the United States?
Raw Embedding Output: {"n_embedding": 384, "embedding": [0.5426152349,-0.03840282559,-0.03644151986,0.3677068651,-0.115977712...(omitted)...,-0.003531290218]}
Interact with Embedding:
N_Embd: 384
Show the first 5 elements:
embd[0] = 0.5426152349
embd[1] = -0.03840282559
embd[2] = -0.03644151986
embd[3] = 0.3677068651
embd[4] = -0.115977712
```

## Code walkthrough

The Rust source code for this project is [here](https://github.com/second-state/WasmEdge-WASINN-examples/blob/master/wasmedge-ggml/embedding/src/main.rs). First, we start the execution context with the `--nn-preload` model by its name.

```
let graph = GraphBuilder::new(GraphEncoding::Ggml, ExecutionTarget::AUTO)
    .config(options.to_string())
    .build_from_cache(model_name)
    .expect("Create GraphBuilder Failed, please check the model name or options");
let mut context = graph
    .init_execution_context()
    .expect("Init Context Failed, please check the model");
```

Then we call the `compute()` function on the execution context, and pass in a sentence to compute an embedding vector.

```
let tensor_data = prompt.as_bytes().to_vec();
context.set_input(0, TensorType::U8, &[1], &tensor_data).unwrap();
context.compute().unwrap();
```

You can then retrieve the generated embedding vector from the execution context. The embedding data is a JSON structure. The `n_embedding` field is the size of embedding vector. This vector size is determined by the embedding model itself. That is, an embedding model will always generate embeddings of the same size. The `embedding` field is the array for the vector data.

```
let embd = get_embd_from_context(&context);
let n_embd = embd["n_embedding"].as_u64().unwrap();

println!("Show the first 5 elements:");
for idx in 0..5 {
    println!("embd[{}] = {}", idx, embd["embedding"][idx as usize]);
}
```

The `get_embd_from_context()` function is straightforward. It simply retrieves data from the execution context's output buffer.

```
fn get_embd_from_context(context: &GraphExecutionContext) -> Value {
    serde_json::from_str(&get_data_from_context(context, 0)).unwrap()
}

fn get_data_from_context(context: &GraphExecutionContext, index: usize) -> String {
    // Preserve for 4096 tokens with average token length 15
    const MAX_OUTPUT_BUFFER_SIZE: usize = 4096 * 15 + 128;
    let mut output_buffer = vec![0u8; MAX_OUTPUT_BUFFER_SIZE];
    let mut output_size = context.get_output(index, &mut output_buffer).unwrap();
    output_size = std::cmp::min(MAX_OUTPUT_BUFFER_SIZE, output_size);

    String::from_utf8_lossy(&output_buffer[..output_size]).to_string()
}
```

You can upsert the `embd["embedding"]` data structure to any vector database you might use.

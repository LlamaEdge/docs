---
sidebar_position: 5
---

# Translation Agent on LlamaEdge

This LLM Translation Agent originally built by [Prof. Andrew Ng](https://www.linkedin.com/posts/andrewyng_github-andrewyngtranslation-agent-activity-7206347897938866176-5tDJ/) is designed to facilitate accurate and efficient translation across multiple languages. It employs open source LLMs (Large Language Models) to provide high-quality translations. You can use your own fine-tuned models or any LLMs on Hugging Face like Meta's Llama 3. This documentation shows how the Transgenic Agent utilizes the Gemma-2-9B model for translation.


>For commands on starting and running this agent, refer to [GitHub - Second State/translation-agent](https://github.com/second-state/translation-agent/blob/use_llamaedge/step-by-step-use-LocalAI.md).


## Run Gemma-2-9B on your own device

See detailed instructions to [Run Gemma-2-9B on your own device.](https://www.secondstate.io/articles/gemma-2-9b/)

Download the [Gemma-2-9B-it model GGUF file](https://huggingface.co/second-state/gemma-2-9b-it-GGUF). Since the size of the model is 6.40G, it could take a while to download.

```
curl -LO https://huggingface.co/second-state/gemma-2-9b-it-GGUF/resolve/main/gemma-2-9b-it-Q5_K_M.gguf
```

Start an API server for the model.

```
wasmedge --dir .:. --nn-preload default:GGML:AUTO:gemma-2-9b-it-Q5_K_M.gguf \
  llama-api-server.wasm \
  --prompt-template gemma-instruct \
  --ctx-size 4096 \
  --model-name gemma-2-9b
```

## Run the Translation Agent to run on top of Gemma-2-9B

To get started, clone the Translation Agent.

```
git clone https://github.com/second-state/translation-agent.git
    
cd translation-agent
git checkout use_llamaedge
```

Here, we run Gemma-2-9B locally and our Translation Agent on top of them respectively to showcase their translation quality. We test a simple translation task to see the results so as to compare their translation capabilities. You will need to install [WasmEdge](https://github.com/WasmEdge/WasmEdge) and the [LlamaEdge API server](https://github.com/LlamaEdge/LlamaEdge) to run those models across major GPU and CPU platforms.

```
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install_v2.sh | bash -s -- -v 0.13.5

curl -LO https://github.com/LlamaEdge/LlamaEdge/releases/latest/download/llama-api-server.wasm
```

You will also need the following configurations and prerequisites to run the agent app.

```
export OPENAI_BASE_URL="http://localhost:8080/v1"
export PYTHONPATH=${PWD}/src
export OPENAI_API_KEY="LLAMAEDGE"

pip install python-dotenv
pip install openai tiktoken icecream langchain_text_splitters
```

Find the `examples/example_script.py` file in your cloned agent repo and review its code. It tells the agent where to find your document and how to translate it. Change the model name to the one you are using, here we’re using `gemma-2-9b` model; also change the source and target languages you want (here we put `Chinese` as the source language and `English` as the target language).

Then, you can find a `examples/sample-texts` folder in your cloned repo. Put[the file you want to translate](https://hackmd.io/tdLiVR3TSc-8eVg_E-j9QA?view#Source-text-Intro-of-Forbidden-City) in this folder and get its path. Here because we named our [source text](https://hackmd.io/tdLiVR3TSc-8eVg_E-j9QA?view#Source-text-Intro-of-Forbidden-City) `forbiddencity.txt`, the relative path to the document would be `sample-texts/forbiddencity.txt`.

```
import os  
import translation_agent as ta  
    
if __name__ == "__main__":
    source_lang, target_lang, country = "Chinese", "English", "Britain"
    
    relative_path = "sample-texts/forbiddencity.txt"
    script_dir = os.path.dirname(os.path.abspath(__file__))
    
    full_path = os.path.join(script_dir, relative_path)
    
    with open(full_path, encoding="utf-8") as file:
        source_text = file.read()
    
    print(f"Source text:\n\n{source_text}\n------------\n")
    
    translation = ta.translate(
            source_lang=source_lang,
            target_lang=target_lang,
            source_text=source_text,
            country=country,
            model="gemma-2-9b",
    )
    
    print(f"Translation:\n\n{translation}")
```


Run the below commands to have your text file translated into English.

```
cd examples    
python example_script.py
```

Wait a few minutes and [the English translation](https://hackmd.io/tdLiVR3TSc-8eVg_E-j9QA?view#English-Translation-by-Gemma-2-9B) will appear on your terminal screen.




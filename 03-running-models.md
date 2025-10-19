
# Running Your First Model

With `llama.cpp` built, you're ready for the exciting part: running a large language model. This guide will walk you through downloading a model and using the primary command-line tool, `llama-cli`, to generate text.

## Step 1: Get a Model

`llama.cpp` uses a special model format called **GGUF**. You cannot use models from PyTorch or TensorFlow directly. You need a file that ends in `.gguf`.

The best place to find GGUF models is the [Hugging Face Hub](https://huggingface.co/models?library=gguf&sort=trending). For this guide, we'll use a small, high-quality model from Google, **Gemma 2B Instruct**.

There are two ways to get the model:

### Option A: Automatic Download (Recommended)

The `llama-cli` tool can download the model for you directly from Hugging Face. This is the easiest method.

All you need is the Hugging Face repository ID, which for this model is `ggml-org/gemma-2-2b-it-GGUF`.

### Option B: Manual Download

If you prefer to download the model yourself, you can use `wget` or your browser. This gives you more control over where the model is stored.

```bash
# Create a directory for your models
mkdir -p models

# Download the model file (approx. 1.67 GB)
wget -O models/gemma-2-2b-it.Q4_K_M.gguf https://huggingface.co/ggml-org/gemma-2-2b-it-GGUF/resolve/main/gemma-2-2b-it.Q4_K_M.gguf
```

## Step 2: Run Inference

Now, let's use `llama-cli` to generate some text. The examples below assume you are in the root `llama.cpp` directory.

### Simple Text Completion

This is the most basic use case. We provide the model with a starting prompt and it generates the rest.

Let's ask it a question. We'll use the `-m` flag to specify the model. You can provide either a local file path or a Hugging Face repository ID. Using the ID will trigger the automatic download.

```bash
# This will automatically download the model on the first run
./build/bin/llama-cli -m ggml-org/gemma-2-2b-it-GGUF/gemma-2-2b-it.Q4_K_M.gguf -p "What is the capital of France?"
```
> **Note:** The automatic download requires `wget` to be installed on your system. If you prefer, you can still use the manual download method and provide the local path: `./build/bin/llama-cli -m models/gemma-2-2b-it.Q4_K_M.gguf ...`

The model will print some diagnostic information and then output the answer. The output you see might be slightly different.

```
... (diagnostic info) ...

What is the capital of France?

The capital of France is Paris. It is the most populous city in the country, with a population of over 2 million people.
```

### Interactive Chat

For instruction-tuned models like Gemma-2B-IT, the best way to interact with them is through a conversation. You can do this by running `llama-cli` in interactive mode with the `-cnv` flag.

```bash
./build/bin/llama-cli -m models/gemma-2-2b-it.Q4_K_M.gguf -cnv
```

The program will load the model and wait for your input. It will feel like a chat application.

```
== Running in interactive mode. ==
 - Press Ctrl+C to interject.
 - Press Return to return control to the user.
 - To return control without starting a new line, end your input with '/'.

> What is the capital of France?

The capital of France is Paris.

> What is it famous for?

Paris is famous for many things, including:

* **Eiffel Tower:** An iconic iron tower that is one of the most recognizable landmarks in the world.
* **Louvre Museum:** The world's largest art museum, home to masterpieces like the Mona Lisa and the Venus de Milo.
* **Notre-Dame Cathedral:** A medieval Catholic cathedral that is a masterpiece of French Gothic architecture.
> (Press Ctrl+C to exit)
```

## Step 3: Enable GPU Acceleration

If you built `llama.cpp` with a hardware backend (as described in the **Build from Source** guide), you must tell it to use the GPU. If you don't, it will default to using the CPU.

The key flag is `-ngl` (or `--n-gpu-layers`). This tells `llama.cpp` how many layers of the model to offload to the GPU.

- A value of `0` means only the CPU is used.
- A positive value (e.g., `32`) offloads that many layers.
- A large value (e.g., `999`) will offload as many layers as possible.

**Recommendation:** Start by setting `-ngl 999`. The program is smart enough to only offload the maximum number of layers that fit in your GPU's VRAM.

Here is the command to run our previous query with GPU acceleration:

```bash
./build/bin/llama-cli -m models/gemma-2-2b-it.Q4_K_M.gguf -p "What is the capital of France?" -ngl 999
```

You will see new lines in the diagnostic output confirming that the GPU is being used. For a CUDA build, it will look something like this:

```
... (some output) ...
ggml_init_cublas: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 4090, compute capability 8.9, VMM: yes
... (more output) ...
llm_load_tensors: ggml ctx size =    0.23 MiB
llm_load_tensors: offloading 33 layers to GPU
llm_load_tensors: offloaded 33/33 layers to GPU
llm_load_tensors:        CPU buffer size =   129.33 MiB
llm_load_tensors:      CUDA0 buffer size =  1541.33 MiB
...
```

Notice the lines `offloading 33 layers to GPU` and the reported `CUDA0 buffer size`. This confirms acceleration is working! You should notice a significant improvement in generation speed.

## Common `llama-cli` Flags

`llama-cli` has many options. Here are a few of the most important ones for getting started:

| Flag | Alias | Description |
| :--- | :--- | :--- |
| `--model` | `-m` | **(Required)** Specifies the path to the GGUF model file. |
| `--prompt` | `-p` | The text prompt to start generation from. |
| `--n-predict` | `-n` | The maximum number of tokens to generate. Default: `-1` (infinity). |
| `--ctx-size` | `-c` | Sets the size of the prompt context window (in tokens). Default: `512`. |
| `--n-gpu-layers` | `-ngl` | The number of model layers to offload to the GPU. |
| `--conversation` | `-cnv` | Forces the tool to run in interactive (chat) mode. |
| `--help` | `-h` | Shows a list of all available commands and options. |

---

Now you can run models locally. But what if you want to build an application or a web UI on top of your model? For that, you'll need the `llama-server`.

---

### 💡 DIY: Explore Sampling Parameters

Now that you can run the model, you can start to control *how* it generates text. The creativity and determinism of the output are controlled by **sampling parameters**.

Try this exercise:

1.  Run the same prompt (e.g., `"Tell me a short story about a brave knight."`) three times with different temperature settings:
    *   `--temp 0.2` (More focused and deterministic)
    *   `--temp 0.8` (A good balance)
    *   `--temp 1.5` (More creative, potentially chaotic)
2.  Use the `--top-k` flag. Try values like `10` and `100`.
3.  Document how the output changes with each flag. In your own words, what do you think `temperature` and `top-k` do?

---

**Next → [Using the API Server](./04-api-server.md)**


# A Tour of the Source Code

Welcome to the final chapter of our introductory tour. You've learned how to build, run, and interact with `llama.cpp`. Now, it's time to look at the blueprint. This guide provides a high-level map of the source code, designed to help new contributors and curious developers navigate the repository.

## Core Directories

The `llama.cpp` repository is organized into several key directories. Here is a breakdown of the most important ones.

### `/` (The Root)

This is the main entry point of the project.

-   `CMakeLists.txt`: The master script for the `CMake` build system. It defines all the build targets, options, and dependencies.
-   `convert_hf_to_gguf.py`: The primary Python script for converting Hugging Face models into the GGUF format. It's a complex and important piece of the ecosystem.
-   `Makefile`: A simpler, alternative build system using `make`. While `CMake` is recommended for its flexibility, the `Makefile` is great for quick, standard builds.

### `ggml/`

This is the heart of the project. It's a submodule containing the `ggml` tensor library.

-   `ggml/src/ggml.c`: The core of `ggml`. This massive file implements the fundamental concepts we discussed: tensor operations, memory contexts, and the computation graph logic.
-   `ggml/include/gguf.h` & `ggml/src/gguf.c`: The header and implementation for the GGUF file format. This code handles reading the metadata and tensor information from `.gguf` files.
-   `ggml/src/ggml-cuda.cu`, `ggml/src/ggml-metal.m`: These files (and others like them) contain the backend-specific kernels for GPU acceleration. When `ggml` builds a computation graph, it can dispatch operations like matrix multiplications to functions defined in these files.

### `common/`

This directory contains code that is shared across many of the command-line tools (like `llama-cli` and `llama-server`).

-   `common/common.cpp`: Contains shared logic for parsing command-line arguments and loading models.
-   `common/sampling.cpp`: A critical file that implements the various token sampling methods: temperature, top-k, top-p, mirostat, and more. After the model produces its raw output (logits), this code is responsible for choosing the next token.

### `include/`

This directory contains the public-facing C API for the `llama.cpp` library.

-   `llama.h`: This is the most important header for developers who want to integrate `llama.cpp` into their own C/C++ applications. It defines the functions for loading models, running inference, and managing the context.

### `src/`

This directory contains the implementation of the `libllama` library itself.

-   `llama.cpp`: The main implementation file for the `llama.h` API. This file is the glue that connects `ggml`, the GGUF loader, and the model-specific logic into a cohesive whole.

### `tools/`

This directory contains the source code for the user-facing executables.

-   `tools/main/main.cpp`: The source for `llama-cli`. It's an excellent, comprehensive example of how to use `libllama`.
-   `tools/server/server.cpp`: The source for `llama-server`. It demonstrates how to wrap `libllama` in a web server.
-   `tools/quantize/quantize.cpp`: The source for the `quantize` tool.

## Execution Flow: A `llama-cli` Example

To tie everything together, let's trace the high-level execution flow of a simple command: `llama-cli -m my-model.gguf -p "Hello"`

1.  **`tools/main/main.cpp`**: The program starts here. It parses the command-line arguments (`-m`, `-p`, etc.) using the utilities in `common/common.cpp`.

2.  **`llama.cpp` (`llama_load_model_from_file`)**: `main.cpp` calls this function to load the model specified by the `-m` flag.

3.  **`ggml/src/gguf.c`**: `llama_load_model_from_file` uses the GGUF functions to open `my-model.gguf`, read its header, and parse all the key-value metadata (architecture, layer count, context length, tokenizer vocabulary, etc.).

4.  **`ggml/src/ggml.c`**: Based on the metadata, `llama.cpp` creates a `ggml_context` and allocates memory for the model's weights and runtime memory. It uses `mmap` to map the tensor data from the GGUF file directly into memory.

5.  **`llama.cpp` (`llama_eval`)**: The main evaluation loop begins. The prompt "Hello" is tokenized using the vocabulary loaded from the GGUF file.

6.  **`ggml/src/ggml.c`**: Inside `llama_eval`, a computation graph representing the transformer's forward pass is built. This graph consists of `ggml_mul_mat`, `ggml_add`, `ggml_silu`, and other operations, all linked together.

7.  **`ggml-cuda.cu` / `ggml-metal.m`**: `ggml_graph_compute` is called. As `ggml` traverses the graph, it sees that some operations (like the large matrix multiplications) can be offloaded to the GPU. It calls the specialized CUDA or Metal kernels defined in these backend files to execute those operations on the GPU.

8.  **`common/sampling.cpp`**: The forward pass produces a list of logits (a probability distribution over the entire vocabulary). This is passed to the sampler, which applies temperature, top-k, etc., and selects the next token.

9.  The token is converted back to text and printed to the screen. The process repeats with the new token added to the context until the generation is complete.

[**Diagram Placeholder:** A sequence diagram illustrating the `llama-cli` execution flow. It should show the sequence of calls between `main.cpp`, `src/llama.cpp`, `ggml/src/gguf.c`, `ggml/src/ggml.c`, and the GPU backends (`ggml-cuda.cu`, etc.), matching the steps described in the text.]

## Your Journey as a Contributor

This concludes our initial tour of the `llama.cpp` project. You now have a map that connects the high-level features to the underlying code.

The best way to learn more is to dive in. Try to follow the execution path of a function that interests you. Read the code, set breakpoints in a debugger, and don't be afraid to experiment. The `llama.cpp` community is active and welcoming to new contributors.

Welcome, and happy coding!

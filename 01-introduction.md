# Introduction to llama.cpp
Welcome to A Practical Guide to llama.cpp — an independent, community-written resource for developers, researchers, and curious users who want to understand and leverage this project.

Based on the open-source project llama.cpp by Georgi Gerganov and contributors.

## What is llama.cpp?

At its core, `llama.cpp` is a plain C/C++ implementation for running Large Language Models (LLMs). This "zero-dependency" approach makes it incredibly portable, efficient, and easy to embed in other applications.

It is the primary development ground for its tensor library, **`ggml`**, and the home of the **GGUF (GPT-Generated Universal Format)** file format, which is central to the ecosystem.

### The GGUF Advantage

You will see the term **GGUF** everywhere. Think of it as a `.zip` file for LLMs. It contains everything needed to run a model:

- **The Model Weights:** The core parameters of the neural network.
- **The Tokenizer:** The vocabulary and logic to convert text to and from numbers the model understands.
- **Metadata:** Information about the model, such as its architecture, context length, and recommended chat templates.

The key feature of GGUF is its support for **quantization**. Quantization is the process of reducing the precision of the model's weights (e.g., from 16-bit floating-point numbers to 4-bit integers). This dramatically reduces memory usage and speeds up inference, making it possible to run powerful models on devices with limited resources, like laptops, phones, and single-board computers.

### Core Components

The `llama.cpp` repository isn't a single program but a toolkit. Here are the primary tools you'll encounter:

| Tool | Purpose | Who is it for? |
| :--- | :--- | :--- |
| `llama-cli` | The main command-line interface. A powerful, versatile tool for text generation, conversation, and testing nearly all of `llama.cpp`'s features. | **Everyone.** Especially power users and developers. |
| `llama-server` | A lightweight, OpenAI-compatible web server. It exposes the model's capabilities via a REST API, allowing you to build applications on top of it. | **Developers** building services or web UIs. |
| `llama-bench` | A benchmarking tool to measure the performance of your model and hardware setup. | **Researchers & Advanced Users** optimizing for performance. |
| `llama-perplexity` | A tool to evaluate a model's quality by measuring its perplexity on a given text. | **Researchers & Model Developers.** |
| `convert_*.py` | A collection of Python scripts to convert models from other formats (like PyTorch `.pth` or Hugging Face `safetensors`) into GGUF. | **Users** who need to prepare a model for `llama.cpp`. |

## Who These Docs Are For

We've designed this documentation to serve three primary audiences. Find the path that best suits you:

#### 🚀 For New Users

If you're new to `llama.cpp` and just want to run a model and start chatting, your journey starts here:

1.  **Installation:** Learn how to get `llama.cpp` running on your system.
2.  **Running Models:** Find a model and run your first inference.

#### 💻 For Developers

If you want to build applications with `llama.cpp` or contribute to the project:

1.  **Building from Source:** Compile `llama.cpp` with custom backends like Metal, CUDA, or BLAS.
2.  **Using `llama-server`:** Learn to use the API for your own applications.
3.  **Internals & Architecture:** Dive deep into the design of `ggml` and `llama.cpp`.

#### 🔬 For Researchers & Power Users

If you're focused on model quality, performance tuning, and advanced features:

1.  **Quantization Guide:** Understand the different quantization methods and their trade-offs.
2.  **Performance Benchmarking:** Use `llama-bench` to optimize your setup.
3.  **Grammars (GBNF):** Constrain model output to specific formats like JSON.

## Architectural Philosophy

Understanding the "why" behind `llama.cpp` helps in using it effectively.

-   **CPU-First, but GPU-Accelerated:** `llama.cpp` is optimized for CPU execution (via ARM NEON, AVX, etc.), ensuring it runs well on almost any machine. However, it also has first-class support for GPU backends (Metal, CUDA, Vulkan, HIP) to accelerate inference when available.
-   **Quantization is Key:** We believe that for widespread, local inference, models must be small and fast. Quantization is not an afterthought; it's a core feature.
-   **Minimal Dependencies, Maximum Portability:** By avoiding heavy frameworks and sticking to C/C++, `llama.cpp` can be compiled and run on an enormous variety of systems, from macOS and Windows to Linux, Android, and even web browsers via WebAssembly.

---

Ready to get started? The first step for any user is to get the tools onto your machine.

**Next → [Build from Source](./02-build-from-source.md)**

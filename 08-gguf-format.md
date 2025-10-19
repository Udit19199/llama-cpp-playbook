
# The GGUF File Format

`GGUF` stands for **Georgi Gerganov Universal Format**. As the name suggests, it is the foundational file format that makes the `ggml` ecosystem, including `llama.cpp`, work so seamlessly.

This document provides a technical overview of the GGUF format for developers and researchers.

## What is GGUF?

At a high level, GGUF is a binary file format designed to store everything needed to run a large language model in a **single, self-contained file**. This includes:

-   The model's architecture and hyperparameters.
-   The complete tokenizer vocabulary and settings.
-   The model's weights (tensors), which can be quantized or in full precision.

This single-file approach eliminates the need for separate configuration files, vocabulary files, and model weight files, making GGUF models incredibly portable and easy to use.

## File Structure

A GGUF file is composed of a few key sections, laid out sequentially:

1.  **Header:** A small section containing:
    -   The magic number `GGUF` to identify the file type.
    -   The version of the GGUF format the file uses.

2.  **Metadata Key-Value Store:** A flexible section that stores all non-tensor data. This is the "brains" of the file.

3.  **Tensor Info:** A section that defines the name, type, shape, and offset of every tensor in the file.

4.  **Padding:** Optional padding to ensure the following tensor data section is aligned to a specific byte boundary (typically 32 or 64 bytes). This alignment is crucial for performance.

5.  **Tensor Data:** The raw binary data for all the model's weights, packed together.

```text
+--------------------------+
| GGUF Header              |
| (Magic, Version, Counts) |
+--------------------------+
| Metadata                 |
| (Key-Value Pairs)        |
+--------------------------+
| Tensor Info              |
| (Name, Shape, Type, ...) |
+--------------------------+
| Padding                  |
+--------------------------+
| Tensor Data              |
| (Aligned Binary Weights) |
+--------------------------+
```

[**Diagram Placeholder:** An enhanced version of the file structure diagram. It should use arrows to show how the `Tensor Info` block (containing names and offsets) acts as a directory pointing to specific locations within the large `Tensor Data` block.]

## The Metadata Section

This is where the power and flexibility of GGUF shine. It is a simple key-value store where the keys are strings and the values can be various data types (integers, floats, booleans, strings, or arrays of these types).

`llama.cpp` uses this section to store everything it needs to know about the model before loading it. While developers can add custom keys, there is a set of standardized keys that `llama.cpp` understands.

Here are some of the most important standardized keys:

| Key | Description |
| :--- | :--- |
| `general.architecture` | **(Required)** The architecture of the model, e.g., `llama`, `falcon`, `gemma`. |
| `general.name` | A user-friendly name for the model. |
| `llama.context_length` | The maximum context length the model supports. |
| `llama.embedding_length` | The size of the model's embedding layer. |
| `llama.block_count` | The number of transformer blocks (layers) in the model. |
| `llama.attention.head_count` | The number of attention heads. |
| `llama.attention.head_count_kv` | The number of key-value heads (for Grouped-Query Attention). |
| `tokenizer.ggml.model` | The tokenizer type, e.g., `llama` for SentencePiece, `gpt2` for BPE. |
| `tokenizer.ggml.tokens` | An array of strings representing the tokenizer's vocabulary. |
| `tokenizer.ggml.scores` | An array of floats representing the score for each vocabulary token (for SentencePiece). |
| `tokenizer.ggml.merges` | An array of strings for BPE merges. |
| `tokenizer.chat_template` | A Jinja2 template string defining how to format conversational prompts. |

Because this is a key-value store, new metadata can be added to the GGUF standard without breaking compatibility with older versions of `llama.cpp`. An older client will simply ignore keys it doesn't recognize.

## The Tensor Data Section

This section contains the raw, binary data for every tensor in the model. The `Tensor Info` section that precedes it acts as a directory, providing the name, shape, data type, and offset in the file for each tensor.

### Memory Mapping (`mmap`)

A key performance feature of `llama.cpp` is its use of memory mapping (`mmap`). Instead of reading the entire tensor data from the file into RAM or VRAM, `llama.cpp` can map the file directly into its virtual address space. This means the operating system handles loading the data from disk as it's needed.

This results in **near-instantaneous model loading**, as no time is spent copying gigabytes of data from disk to memory upfront. The padding in the GGUF file ensures that this memory-mapped data is correctly aligned for efficient access by the CPU or GPU.

## GGUF Tooling

The `gguf-py` package, included in the `llama.cpp` repository, provides helpful Python scripts for inspecting and manipulating GGUF files:

-   `gguf-dump.py`: Prints all the metadata and tensor info from a GGUF file.
-   `gguf-set-metadata.py`: Allows you to change metadata values in an existing file.

These tools are invaluable for developers who need to debug conversion scripts or inspect the properties of a model.

---

With a solid understanding of `ggml` and `GGUF`, you are now equipped to explore the `llama.cpp` source code itself.

**Next → [Core Concepts: The KV Cache](./10-kv-cache.md)**

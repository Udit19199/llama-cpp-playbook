# Core Concepts: The KV Cache

You cannot understand the performance of a transformer during inference without understanding the **KV Cache**. It is the single most important optimization for generating text quickly.

## The Problem: Transformers are Slow

A transformer block works by relating every token in the context to every other token. When you generate a new token, the model must perform calculations over the *entire* sequence of previous tokens.

Imagine you have generated 100 tokens and want to generate the 101st. A naive implementation would re-compute the attention scores for all 100 previous tokens. When you generate the 102nd token, it would re-compute everything for the 101 previous tokens. This is incredibly wasteful and slow, making generation an O(n^2) operation in sequence length.

## The Solution: Cache the Keys and Values

The KV Cache solves this. Inside the attention mechanism, the "Query" (Q) vector of the newest token is compared against the "Key" (K) and "Value" (V) vectors of *all previous tokens*.

The crucial insight is that the Key and Value vectors for the previous tokens do not change between generation steps. They can be calculated once and then stored.

The KV Cache is simply a large block of memory (often in VRAM) that stores the K and V tensors for every token in the current context.

### How it Works

1.  **Prompt Processing (Prefill):** When you first provide a prompt, the model processes all tokens at once. As it does, it calculates the K and V vectors for each token and fills the KV cache. This is a parallel, efficient operation.
2.  **Token Generation (Decoding):** To generate the next token, the model only needs to process a *single* new token. It calculates the Q, K, and V vectors for this one token.
    *   It appends the new K and V vectors to the cache.
    *   It uses the new Q vector to attend to the *entire* cache of K and V vectors (including the one it just added).
3.  This process repeats for every new token.

By using the cache, each new token generation step becomes much faster, as the model avoids re-calculating attention for all the previous tokens. This changes the generation complexity from O(n^2) to O(n).

### Memory Management in `llama.cpp`

The KV cache is the largest consumer of memory after the model weights themselves. Its size is determined by:

`context_length * num_layers * embedding_dim * 2 (for K and V) * bytes_per_element`

`llama.cpp` allocates the KV cache as a large, contiguous block of memory within its context. Managing this memory efficiently is key, especially for long context lengths. When the context window gets full, strategies are needed to evict old tokens, though this is an advanced topic (e.g., context shifting).

Understanding the KV cache is essential for any engineer working on LLM inference, as it is the primary factor influencing token generation speed and memory usage.

---

**Next → [A Tour of the Source Code](./09-source-code-structure.md)**

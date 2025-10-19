
# A Guide to Quantization

Quantization is one of the most important concepts in the `llama.cpp` ecosystem. It's the process of reducing the precision of a model's weights, which has two huge benefits:

1.  **Smaller Model Size:** A quantized model takes up significantly less disk space and, more importantly, less RAM/VRAM.
2.  **Faster Inference:** Computations with lower-precision numbers (integers) are often much faster than with high-precision floating-point numbers, especially on CPUs.

Think of it like compressing a high-resolution PNG image into a JPG. You lose a small amount of imperceptible detail, but the file size becomes a fraction of the original. Quantization allows you to run powerful, billion-parameter models on consumer hardware like laptops and even phones.

## How to Quantize a Model

To quantize a model, you need to start with a high-precision version, typically in 16-bit floating-point (`F16`) format. You then use the `quantize` tool that you compiled during the build process.

The basic command is:

```bash
./build/bin/quantize <path-to-f16-model.gguf> <path-to-output-quantized-model.gguf> <quantization-type>
```

For example, to convert an `F16` model to the popular `Q4_K_M` format, you would run:

```bash
./build/bin/quantize ./models/my-model-F16.gguf ./models/my-model-Q4_K_M.gguf Q4_K_M
```

This will create a new, much smaller model file ready for use.

## Choosing a Quantization Method

`llama.cpp` supports many different quantization methods. They are named using a convention that indicates the number of bits they use and the specific techniques involved (e.g., `Q4_K_M`, `Q8_0`).

The most important trade-off is between **model size/speed** and **quality/accuracy**. Lower-bit quants are smaller and faster but may have a more noticeable impact on the model's performance.

Here is a breakdown of the most common and recommended quantization types, from highest quality to lowest.

| Quantization Type | Bits/Weight | Use Case & Trade-offs |
| :--- | :--- | :--- |
| `F16` | 16.0 | **Unquantized.** The original, high-precision format. Highest quality, but largest size and slowest performance. Useful as a baseline or on very high-end hardware. |
| `Q8_0` | 8.5 | **Highest Quality Quant.** Very little perceptible quality loss compared to F16. A great choice if you have plenty of VRAM and prioritize quality above all else. |
| `Q6_K` | 6.6 | **High Quality.** A good balance, offering quality very close to Q8_0 but with a significant size reduction. |
| `Q5_K_M` | 5.7 | **Recommended High Quality.** Often considered a sweet spot. Better quality than the 4-bit quants with a modest size increase. |
| `Q4_K_M` | 4.9 | **The Gold Standard.** The most popular and recommended method for general use. Provides an excellent balance of model quality, speed, and size. **When in doubt, start here.** |
| `Q4_0` | 4.5 | **Legacy 4-bit.** A slightly smaller but less robust 4-bit quant. `Q4_K_M` is generally preferred. |
| `Q3_K_M` | 4.0 | **Low Quality.** A more aggressive quantization. Size and speed are excellent, but the quality loss can be noticeable. Good for very resource-constrained devices. |
| `Q2_K` | 3.2 | **Very Low Quality.** The smallest and fastest option. Expect a significant drop in model performance (e.g., increased repetition or nonsensical output). Use only when absolutely necessary. |

> **Note on Bits/Weight:** The "Bits/Weight" value is an *average*. It includes the overhead from the quantization scales and other metadata associated with the method, which is why it is often not a whole number.

> **What does "K_M" or "K_S" mean?** The `_K` indicates an improved version of the quantization method that uses a more sophisticated technique to preserve model quality. The `_S`, `_M`, and `_L` suffixes refer to small, medium, and large variants of the method, which offer different trade-offs. For most users, `_K_M` is the recommended choice.

## Practical Recommendations

-   **For High-End GPUs (≥ 16GB VRAM):** Start with `Q6_K` or `Q8_0`. You have the VRAM to spare, so you can enjoy near-lossless quality.

-   **For Mid-Range GPUs or Apple Silicon (8-16GB VRAM):** `Q4_K_M` and `Q5_K_M` are your best friends. `Q4_K_M` is the perfect all-rounder, while `Q5_K_M` is a great option if you can afford the slightly larger size.

-   **For CPU-only or Low-VRAM GPUs (< 8GB):** You may need to use `Q3_K_M` or even `Q2_K` for larger models to ensure they run at an acceptable speed. For smaller models (e.g., 3B or 7B), `Q4_K_M` should still be your first choice.

## Advanced: Importance Matrix (imatrix)

For the best possible quality, `llama.cpp` supports using an **importance matrix** during quantization. This is a file that tells the quantizer which weights are more "important" and should be preserved with higher precision.

Using an imatrix can significantly reduce the quality loss, especially for very low-bit quants. The process involves an extra step of calculating this matrix on a calibration dataset.

While this is an advanced topic, it's good to know that it exists as a way to push the boundaries of quantization quality.

---

### How to Measure Quality Loss?

While the table above gives general guidance, the actual impact of quantization depends on the model and your specific task. You can quantitatively measure the quality loss by using the `llama-perplexity` tool on a test dataset.

By comparing the perplexity score of the `F16` model to a `Q4_K_M` version, you can make an informed, data-driven decision about the trade-off. A lower perplexity score is better.

**Example:**
```bash
# Evaluate the original F16 model
./build/bin/llama-perplexity -m ./models/my-model-F16.gguf -f wiki.test.raw

# Evaluate the quantized model
./build/bin/llama-perplexity -m ./models/my-model-Q4_K_M.gguf -f wiki.test.raw
```

---

Understanding quantization is key to using `llama.cpp` effectively. Now that you can create and run optimized models, let's explore another powerful feature: controlling the model's output format using grammars.

**Next → [Controlling Output with Grammars](./06-grammars.md)**

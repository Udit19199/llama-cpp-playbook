# Using the API Server (`llama-server`)

While `llama-cli` is excellent for direct interaction, the real power for developers comes from `llama-server`. This tool exposes your model through a powerful, OpenAI-compatible REST API, instantly turning `llama.cpp` into a backend that can power any application, from web UIs to research workflows.

## Step 1: Start the Server

Starting the server is as simple as running `llama-cli`. You point it to a model file and, optionally, tell it to use the GPU.

Let's use the same Gemma model from the previous guide.

```bash
# From your llama.cpp root directory
# Use -ngl 999 to offload all possible layers to the GPU
./build/bin/llama-server -m models/gemma-2-2b-it.Q4_K_M.gguf -ngl 999
```

If successful, the server will load the model and start listening for HTTP requests on port `8080`.

```
... (model loading logs) ...
lama_new_context_with_model: max tensor size =   256.00 MiB
lama server listening at http://127.0.0.1:8080
```

Your `llama.cpp` instance is now an API service! Keep this terminal window open. You'll need a separate terminal for the next steps.

## Step 2: Interact with the API

The server provides several endpoints, but the most important one is `/v1/chat/completions`. It behaves just like the OpenAI Chat Completions API, making it compatible with a vast ecosystem of tools and libraries.

Let's make a request using `curl` from a new terminal.

### Standard Chat Request

This is a simple request-response interaction.

```bash
curl --request POST \
  --url http://localhost:8080/v1/chat/completions \
  --header "Content-Type: application/json" \
  --data '{ \
    "model": "gemma-2-2b-it.Q4_K_M.gguf", \
    "messages": [ \
      { \
        "role": "system", \
        "content": "You are a helpful assistant." \
      }, \
      { \
        "role": "user", \
        "content": "What is the capital of France?" \
      } \
    ] \
  }'
```

The server will return a JSON object containing the model's response:

```json
{
  "id": "chatcmpl-12345",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "gemma-2-2b-it.Q4_K_M.gguf",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The capital of France is Paris."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 24,
    "completion_tokens": 7,
    "total_tokens": 31
  }
}
```

### Streaming Responses

For a better user experience in applications like chatbots, you can stream the response as it's being generated. You do this by adding `"stream": true` to your request.

```bash
curl --request POST \
  --url http://localhost:8080/v1/chat/completions \
  --header "Content-Type: application/json" \
  --data '{ \
    "model": "gemma-2-2b-it.Q4_K_M.gguf", \
    "messages": [ \
      { "role": "user", "content": "Tell me a short story about a robot." }
    ],
    "stream": true
  }'
```

Instead of a single JSON response, the server will send a stream of Server-Sent Events (SSE). Each event contains a small chunk (a token) of the generated text. This allows you to display the response to the user word-by-word.

### The Built-in Web UI

For quick tests, `llama-server` comes with a basic web interface. Just open your browser and navigate to the address shown when the server starts:

**http://localhost:8080**

You'll find a simple, clean chat interface—perfect for quick prompts without having to use `curl`.

## Example: Using the API with Python

Here is a practical example of how you would call the server from a Python script using the popular `requests` library.

```python
# Filename: simple_client.py
import requests
import json

# Server URL
url = "http://localhost:8080/v1/chat/completions"

# Headers
headers = {
    "Content-Type": "application/json",
}

# Request data
data = {
    "model": "gemma-2-2b-it.Q4_K_M.gguf",
    "messages": [
        {
            "role": "system",
            "content": "You are a helpful assistant who always responds in Markdown."
        },
        {
            "role": "user",
            "content": "List three benefits of exercise."
        }
    ],
    "stream": False  # Set to True for streaming
}

# Make the request
response = requests.post(url, headers=headers, data=json.dumps(data))

# Print the response
if response.status_code == 200:
    print(response.json()['choices'][0]['message']['content'])
else:
    print(f"Error: {response.status_code} - {response.text}")

```

Run this script (`pip install requests` if you don't have it), and you will get the model's formatted response directly in your terminal.

## Common `llama-server` Flags

Many flags are shared with `llama-cli`, but here are some of the most important ones for the server:

| Flag | Description |
| :--- | :--- |
| `-m, --model` | **(Required)** Path to the GGUF model file. |
| `-ngl, --n-gpu-layers` | Number of model layers to offload to the GPU. |
| `--port <N>` | Sets the port the server listens on. Default: `8080`. |
| `--host <IP>` | Sets the host address to bind to. Default: `127.0.0.1`. Use `0.0.0.0` to make it accessible on your local network. |
| `-c, --ctx-size` | Sets the context size (in tokens) for the model. |

---

You now have the tools to build powerful, AI-driven applications. To get the most out of your models, it's important to understand quantization—the process of making models smaller and faster.

**Next → [A Guide to Quantization](./05-quantization-guide.md)**

# Running GGUF Models with Ollama

[Ollama](https://ollama.com) is the easiest way to serve a GGUF model locally. This guide shows how to create a custom `Modelfile` from a Qwodel-quantized GGUF and run it.

---

## Prerequisites

- [Install Ollama](https://ollama.com/download) for your platform.
- A `.gguf` file produced by Qwodel.

---

## Step 1: Quantize your model

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="gguf",
    model_path="./llama-3",
    output_dir="./output"
)
output = quantizer.quantize(format="Q4_K_M")
# output → ./output/llama-3-q4_k_m.gguf
```

---

## Step 2: Create a Modelfile

Create a file named `Modelfile` (no extension) in the same directory:

```
FROM ./output/llama-3-q4_k_m.gguf

PARAMETER temperature 0.7
PARAMETER top_p 0.9
PARAMETER num_ctx 4096

SYSTEM "You are a helpful assistant."
```

### Common Modelfile parameters

| Parameter | Description |
|---|---|
| `temperature` | Sampling temperature (0.0 = deterministic, 1.0 = creative) |
| `top_p` | Nucleus sampling threshold |
| `num_ctx` | Context window size in tokens |
| `SYSTEM` | System prompt prepended to every conversation |

---

## Step 3: Create and run the model

```bash
# Register the model with Ollama
ollama create my-llama3 --file ./Modelfile

# Run it interactively
ollama run my-llama3
```

---

## Step 4: Use via API

Ollama exposes a local REST API compatible with the OpenAI client:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"  # required but unused
)

response = client.chat.completions.create(
    model="my-llama3",
    messages=[{"role": "user", "content": "Explain quantization in one sentence."}]
)
print(response.choices[0].message.content)
```

---

## Useful Ollama CLI commands

```bash
ollama list               # List all registered models
ollama show my-llama3     # Show model info
ollama rm my-llama3       # Remove a model
ollama ps                 # Show running models
```

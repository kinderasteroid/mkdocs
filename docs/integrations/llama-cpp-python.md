# Running GGUF Models with llama-cpp-python

[llama-cpp-python](https://github.com/abetlen/llama-cpp-python) provides Python bindings for llama.cpp, letting you load and run a GGUF model directly in Python without a separate server.

---

## Install

```bash
pip install llama-cpp-python
```

For GPU acceleration (CUDA):
```bash
CMAKE_ARGS="-DGGML_CUDA=on" pip install llama-cpp-python
```

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

## Step 2: Load and run

```python
from llama_cpp import Llama

llm = Llama(
    model_path="./output/llama-3-q4_k_m.gguf",
    n_ctx=4096,       # Context window
    n_gpu_layers=-1,  # -1 = offload all layers to GPU (omit for CPU-only)
    verbose=False
)

response = llm.create_chat_completion(
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user",   "content": "What is quantization?"}
    ]
)
print(response["choices"][0]["message"]["content"])
```

---

## Text completion (non-chat)

```python
output = llm(
    "The capital of France is",
    max_tokens=64,
    temperature=0.7,
    top_p=0.9,
    stop=["\n"]
)
print(output["choices"][0]["text"])
```

---

## OpenAI-compatible server

llama-cpp-python includes a built-in HTTP server:

```bash
python -m llama_cpp.server \
    --model ./output/llama-3-q4_k_m.gguf \
    --n_ctx 4096 \
    --n_gpu_layers -1
```

Then use it like any OpenAI endpoint:

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="none")
response = client.chat.completions.create(
    model="local-model",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

---

## Key parameters

| Parameter | Description |
|---|---|
| `n_ctx` | Context window in tokens (default 512) |
| `n_gpu_layers` | Number of layers to offload to GPU (-1 = all) |
| `n_threads` | CPU threads to use for inference |
| `verbose` | Print llama.cpp debug output |

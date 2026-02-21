# Running AWQ Models with vLLM

[vLLM](https://github.com/vllm-project/vllm) is a high-throughput GPU inference engine with first-class support for AWQ quantized models. It is the recommended way to serve Qwodel's AWQ output in production.

---

## Prerequisites

- NVIDIA GPU with CUDA 12.1+
- An AWQ-quantized model directory from Qwodel

## Install

```bash
pip install vllm
```

---

## Step 1: Quantize your model

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="awq",
    model_path="./gemma-2-9b",
    output_dir="./output/gemma-2-9b-awq"
)
quantizer.quantize(format="int4")
# Output: ./output/gemma-2-9b-awq/  (safetensors directory)
```

---

## Step 2: Serve with vLLM

```bash
vllm serve ./output/gemma-2-9b-awq \
    --quantization awq \
    --dtype float16 \
    --max-model-len 8192
```

vLLM starts an OpenAI-compatible server at `http://localhost:8000`.

---

## Step 3: Call the API

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="none"
)

response = client.chat.completions.create(
    model="./output/gemma-2-9b-awq",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user",   "content": "Summarize the benefits of AWQ quantization."}
    ],
    temperature=0.7,
    max_tokens=512
)
print(response.choices[0].message.content)
```

---

## Python offline inference (no server)

```python
from vllm import LLM, SamplingParams

llm = LLM(
    model="./output/gemma-2-9b-awq",
    quantization="awq",
    dtype="float16"
)

params = SamplingParams(temperature=0.8, max_tokens=256)
outputs = llm.generate(["Tell me about model quantization."], params)

for output in outputs:
    print(output.outputs[0].text)
```

---

## Useful vLLM serve options

| Flag | Description |
|---|---|
| `--quantization awq` | Tell vLLM to use AWQ mode |
| `--tensor-parallel-size N` | Spread model across N GPUs |
| `--max-model-len` | Maximum sequence length |
| `--gpu-memory-utilization` | Fraction of GPU memory to use (default 0.9) |
| `--port` | HTTP port (default 8000) |

# GGUF Backend

CPU-friendly quantization for [llama.cpp](https://github.com/ggerganov/llama.cpp)-compatible runtimes.

**Install:** `pip install qwodel[gguf]`

---

## Supported Formats

| Format | Description |
|---|---|
| `Q4_K_M` | Best balance of speed and quality. **Recommended for most users.** |
| `Q5_K_M` | Better quality than Q4_K_M, slightly larger. |
| `Q5_K_S` | Small 5-bit K-quant. |
| `Q6_K` | High quality — between Q8_0 and Q4_K_M. |
| `Q8_0` | Near-lossless. Requires more RAM. |
| `Q2_K` | Maximum compression. Reduced quality. |
| `Q3_K_M` | 3-bit medium quality. |
| `Q4_0` | Compact 4-bit, slightly smaller than Q4_K_M. |
| `Q4_K_S` | Small 4-bit K-quant. |
| `IQ4_NL` | 4.5 bpw importance-based quantization. |
| `IQ3_M` | 3.66 bpw compact importance quantization. |

---

## Parameters

### `Quantizer(...)` — Initialization

| Parameter | Type | Default | Description |
|---|---|---|---|
| `model_path` | `str` | — | Path to HuggingFace model directory or existing `.gguf` file. |
| `output_dir` | `str` | `./quantized_models` | Output directory. |

> GGUF has no additional backend-specific initialization parameters.

### `quantize(format)` — Runtime

| Parameter | Type | Required | Description |
|---|---|---|---|
| `format` | `str` | Yes | One of the formats listed above. |

---

## Example

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="gguf",
    model_path="./llama-3",
    output_dir="./output"
)
output = quantizer.quantize(format="Q4_K_M")
print(f"Output: {output}")
```

**CLI:**
```bash
qwodel quantize ./llama-3 --backend gguf --format Q4_K_M --output ./output
```

---

## After Quantization

Your output is a single `.gguf` file. Run it with:

- [Ollama](../integrations/ollama.md)
- [llama-cpp-python](../integrations/llama-cpp-python.md)

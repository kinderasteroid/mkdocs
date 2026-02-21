# AWQ Backend

GPU-based INT4 quantization using `llm-compressor`. Requires an NVIDIA GPU with CUDA.

**Install:**
```bash
pip install torch==2.1.2 torchvision==0.16.2 torchaudio==2.1.2 \
    --index-url https://download.pytorch.org/whl/cu121
pip install qwodel[awq]
```

---

## Supported Formats

| Format | Description |
|---|---|
| `int4` | 4-bit weight quantization (W4A16). Best for GPU inference. |

---

## Parameters

### `Quantizer(...)` — Initialization

| Parameter | Type | Default | Description |
|---|---|---|---|
| `calibration_dataset` | `str` | `"wikitext:wikitext-2-raw-v1"` | Calibration dataset. Supports HF IDs, `repo:subset` syntax, and local `.json`/`.jsonl`/`.txt` files. |
| `calibration_split` | `str` | `"train"` | Dataset split to use. |
| `token` | `str` | `None` | HuggingFace API token for gated/private models. |
| `batch_size` | `int` | Auto | Calibration batch size. Auto-selected based on available VRAM. |
| `seq_length` | `int` | Auto | Max sequence length for calibration. Auto-selected based on VRAM. |
| `num_samples` | `int` | Auto | Number of calibration samples. Auto-selected based on VRAM. |

### `quantize(...)` — Runtime overrides

| Parameter | Type | Description |
|---|---|---|
| `batch_size` | `int` | Override batch size. |
| `seq_len` | `int` | Override sequence length. |
| `num_samples` | `int` | Override number of calibration samples. |
| `ignore` | `List[str]` | Modules to skip. Supports exact names and `re:` regex patterns (e.g. `["lm_head", "re:.*vision_tower.*"]`). |

---

## VRAM Auto-Config

When `batch_size`, `seq_length`, and `num_samples` are not set, they are automatically chosen based on available GPU VRAM headroom:

| VRAM Headroom | `batch_size` | `seq_len` | `num_samples` |
|---|---|---|---|
| < 4 GB | 1 | 2048 | 32 |
| 4–8 GB | 2 | 4096 | 64 |
| 8–16 GB | 4 | 4096 | 128 |
| 16–24 GB | 8 | 8192 | 128 |
| > 24 GB | 16 | 8192 | 256 |

---

## Example

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="awq",
    model_path="./gemma-2",
    output_dir="./output",
    calibration_dataset="wikitext:wikitext-2-raw-v1",
    token="hf_..."           # only needed for gated models
)
output = quantizer.quantize(format="int4")
print(f"Output: {output}")
```

**CLI:**
```bash
qwodel quantize ./gemma-2 --backend awq --format int4 --output ./output
```

---

## After Quantization

Your output is a directory of `safetensors` files. Load it with [vLLM →](../integrations/vllm.md)

# Using Local LLM Models

If you already have a model stored on disk — whether downloaded with `git lfs`, exported from a training run, or compiled from source — Qwodel can quantize it directly. No internet connection required after the model is on disk.

---

## Supported local formats

| Source | Typical layout | Qwodel accepts? |
|---|---|---|
| HuggingFace `snapshot_download` | `model_dir/config.json` + `*.safetensors` | ✓ |
| `git lfs clone` of a HF repo | Same as above | ✓ |
| Manual HF format (`save_pretrained`) | Same as above | ✓ |
| llama.cpp self-build output | `.gguf` file | ✓ (GGUF backend pass-through) |
| Raw PyTorch checkpoint (`.pt`/`.pth`) | Single weight file | See [PyTorch guide](pytorch.md) |

---

## Step 1: Verify your model directory

A valid HuggingFace-format model directory must contain at minimum:

```
my-model/
├── config.json          # required — architecture definition
├── tokenizer.json       # required — tokenizer config
├── tokenizer_config.json
└── model.safetensors    # weights (or sharded: model-00001-of-00003.safetensors …)
```

Quick check:

```bash
ls -lh ./my-model/
# Look for config.json and at least one .safetensors or .bin file
```

---

## Step 2: Quantize

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="gguf",             # or "awq" / "coreml"
    model_path="./my-model",    # absolute or relative local path
    output_dir="./output"
)
output = quantizer.quantize(format="Q4_K_M")
print(f"Output: {output}")
```

CLI equivalent:

```bash
qwodel quantize \
    --backend gguf \
    --format Q4_K_M \
    --model ./my-model \
    --output ./output
```

---

## Working with `git lfs`

Many HF repos use Git LFS for large weight files. Clone correctly:

```bash
# Install git-lfs if needed
sudo apt install git-lfs    # Ubuntu/Debian
brew install git-lfs         # macOS

git lfs install
git clone https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3 ./mistral-7b
```

Then point Qwodel at `./mistral-7b`.

---

## Self-built llama.cpp models

If you compiled a model to GGUF yourself using `llama.cpp`, you can still pass it to the GGUF backend for re-quantization to a different format:

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="gguf",
    model_path="./my-model.gguf",   # existing GGUF file
    output_dir="./output"
)
output = quantizer.quantize(format="Q2_K")   # re-quantize to smaller format
```

---

## Tips for large models

**Use fast local storage** — Place the model directory on an SSD. GGUF conversion reads the entire model sequentially; a slow spinning disk can make it 3–5× slower.

**RAM requirements** — Qwodel loads model weights into CPU RAM during conversion. As a rough guide, allow ~2× the model file size in available RAM.

**Sharded safetensors** — Qwodel handles sharded safetensors (`model-0000X-of-0000Y.safetensors`) transparently; just pass the directory.

---

## Verify your output

```python
from pathlib import Path

output = Path("./output/my-model-q4_k_m.gguf")
print(f"Size: {output.stat().st_size / 1e9:.2f} GB")
print(f"Exists: {output.exists()}")
```

---

**Next:** [PyTorch Post-Training →](pytorch.md)

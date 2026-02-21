# Downloading and Integrating HuggingFace Models

Qwodel accepts any model directory that follows the HuggingFace Transformers format.
The easiest way to get such a directory is with the `huggingface_hub` library, which is already installed as a Qwodel core dependency.

---

## Prerequisites

- A [Hugging Face account](https://huggingface.co/join) (free)
- `huggingface_hub` — already included in `pip install qwodel`
- For gated models (e.g. Llama, Gemma): a **User Access Token** with read permissions

---

## Step 1: Log in (once, per machine)

```bash
huggingface-cli login
# Paste your token when prompted.
# Token is stored in ~/.cache/huggingface/token
```

Or pass it programmatically:

```python
from huggingface_hub import login
login(token="hf_...")
```

---

## Step 2: Download the model

```python
from huggingface_hub import snapshot_download

model_dir = snapshot_download(
    repo_id="meta-llama/Llama-3.2-3B-Instruct",
    # token="hf_..."  # only needed if not logged in via CLI
    ignore_patterns=["*.msgpack", "flax_model*", "tf_model*"],  # skip non-PyTorch weights
)
print(model_dir)
# → ~/.cache/huggingface/hub/models--meta-llama--Llama-3.2-3B-Instruct/snapshots/<hash>
```

`snapshot_download` handles resumable downloads, caching, and multi-file shards automatically.

### Download to a custom directory

```python
model_dir = snapshot_download(
    repo_id="mistralai/Mistral-7B-v0.3",
    local_dir="./models/mistral-7b",
)
```

---

## Step 3: Quantize with Qwodel

Pass the downloaded directory straight to `Quantizer`:

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="gguf",
    model_path=model_dir,       # path returned by snapshot_download
    output_dir="./output"
)
output = quantizer.quantize(format="Q4_K_M")
print(f"Quantized model saved to: {output}")
```

Or using the CLI:

```bash
qwodel quantize \
    --backend gguf \
    --format Q4_K_M \
    --model ~/.cache/huggingface/hub/models--meta-llama--Llama-3.2-3B-Instruct/snapshots/<hash> \
    --output ./output
```

---

## Download + quantize in one script

```python
from huggingface_hub import snapshot_download
from qwodel import Quantizer

REPO = "Qwen/Qwen2.5-7B-Instruct"
OUTPUT = "./output"

print(f"Downloading {REPO}...")
model_dir = snapshot_download(
    repo_id=REPO,
    ignore_patterns=["*.msgpack", "flax_model*", "tf_model*"],
)

print("Quantizing...")
quantizer = Quantizer(
    backend="gguf",
    model_path=model_dir,
    output_dir=OUTPUT,
)
quantized_path = quantizer.quantize(format="Q4_K_M")
print(f"Done → {quantized_path}")
```

---

## Useful `snapshot_download` options

| Option | Description |
|---|---|
| `repo_id` | `"org/model-name"` on Hugging Face Hub |
| `local_dir` | Save to a specific directory instead of the cache |
| `token` | HF access token (for gated/private models) |
| `ignore_patterns` | Skip unneeded file types (saves bandwidth) |
| `revision` | Download a specific branch, tag, or commit hash |

---

## Troubleshooting

**`gated repo`** — You need to accept the model license on the HF website and ensure your token has read access.

**`OSError: Disk quota exceeded`** — Use `local_dir` to point at a volume with more space, or set `HF_HOME` env var to redirect the cache.

**Slow downloads** — Set `HF_HUB_ENABLE_HF_TRANSFER=1` and install `hf_transfer` (`pip install hf_transfer`) for faster parallel downloads.

---

**Next:** [Serving with Ollama →](ollama.md)

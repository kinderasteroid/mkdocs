# PyTorch Post-Training Integration

After training or fine-tuning a model in PyTorch, you can quantize it with Qwodel before deployment.
This guide covers the standard workflow: save your model in HuggingFace format, then pass it to Qwodel.

---

## Prerequisites

```bash
pip install qwodel[awq]      # AWQ backend — GPU quantization post fine-tune
# or
pip install qwodel[gguf]     # GGUF backend — CPU inference
```

---

## Step 1: Save your trained model

Qwodel expects models in HuggingFace Transformers **`save_pretrained`** format.
After training, call:

```python
# After your training loop
model.save_pretrained("./my-finetuned-model")
tokenizer.save_pretrained("./my-finetuned-model")
```

This creates:

```
my-finetuned-model/
├── config.json
├── tokenizer.json
├── tokenizer_config.json
├── special_tokens_map.json
└── model.safetensors   (or pytorch_model.bin)
```

> **Tip:** Use `save_pretrained(..., safe_serialization=True)` to write `.safetensors` instead of `.bin`
> — it is faster and safer to load.

---

## Step 2: (Optional) Merge LoRA / PEFT adapters

If you trained with PEFT (LoRA, QLoRA, etc.), you must **merge the adapter into the base model** before quantization — Qwodel quantizes the full merged weights.

```python
from peft import PeftModel
import torch

base_model_id = "meta-llama/Llama-3.2-3B-Instruct"
adapter_path   = "./my-lora-adapter"
merged_path    = "./my-merged-model"

# Load base
from transformers import AutoModelForCausalLM, AutoTokenizer
base = AutoModelForCausalLM.from_pretrained(
    base_model_id,
    torch_dtype=torch.bfloat16,
    device_map="cpu"   # merge on CPU to avoid OOM
)
tokenizer = AutoTokenizer.from_pretrained(base_model_id)

# Apply and merge adapter
model = PeftModel.from_pretrained(base, adapter_path)
model = model.merge_and_unload()    # fuses LoRA weights into base

# Save merged model
model.save_pretrained(merged_path, safe_serialization=True)
tokenizer.save_pretrained(merged_path)
print(f"Merged model saved to {merged_path}")
```

---

## Step 3: Quantize with Qwodel

Pass the saved (or merged) directory to `Quantizer`:

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="awq",                     # GPU — best for fine-tuned instruction models
    model_path="./my-finetuned-model", # or ./my-merged-model for LoRA
    output_dir="./output"
)
output = quantizer.quantize(format="int4")
print(f"AWQ-quantized model: {output}")
```

For CPU-only deployment:

```python
quantizer = Quantizer(
    backend="gguf",
    model_path="./my-finetuned-model",
    output_dir="./output"
)
output = quantizer.quantize(format="Q4_K_M")
```

---

## Full end-to-end example

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments, Trainer
from qwodel import Quantizer

MODEL_ID   = "Qwen/Qwen2.5-1.5B-Instruct"
SAVE_DIR   = "./my-finetuned"
OUTPUT_DIR = "./output"

# --- 1. Load base model ----------------------------------------------------
tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)
model = AutoModelForCausalLM.from_pretrained(MODEL_ID, torch_dtype=torch.bfloat16)

# --- 2. Fine-tune (placeholder — plug in your dataset / trainer) -----------
# trainer = Trainer(model=model, args=TrainingArguments(...), ...)
# trainer.train()

# --- 3. Save in HF format ---------------------------------------------------
model.save_pretrained(SAVE_DIR, safe_serialization=True)
tokenizer.save_pretrained(SAVE_DIR)

# --- 4. Quantize ------------------------------------------------------------
quantizer = Quantizer(
    backend="gguf",
    model_path=SAVE_DIR,
    output_dir=OUTPUT_DIR,
)
quantized = quantizer.quantize(format="Q4_K_M")
print(f"Quantized model → {quantized}")
```

---

## Validating the quantized model

After quantization it is good practice to do a quick sanity-check inference:

```python
from llama_cpp import Llama   # pip install llama-cpp-python

llm = Llama(model_path=str(quantized), n_ctx=2048, verbose=False)
result = llm("Once upon a time,", max_tokens=64)
print(result["choices"][0]["text"])
```

---

## Choosing the right backend after training

| Scenario | Recommended backend | Format |
|---|---|---|
| GPU serving in production | `awq` | `int4` |
| CPU local inference (chat, RAG) | `gguf` | `Q4_K_M` |
| Apple Silicon device | `coreml` | `float16` |
| Mobile / edge (quantized further) | `gguf` | `Q2_K` |

---

**Next:** [Serving with vLLM →](vllm.md)

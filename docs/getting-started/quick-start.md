# Quick Start

Get your first quantized model in under 5 minutes.

---

## Python API

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="gguf",
    model_path="meta-llama/Llama-2-7b-hf",
    output_dir="./quantized"
)

output_path = quantizer.quantize(format="Q4_K_M")
print(f"Done! Quantized model: {output_path}")
```

That's it. You now have a `.gguf` file ready for deployment.

---

## Command Line

```bash
qwodel quantize meta-llama/Llama-2-7b-hf \
    --backend gguf \
    --format Q4_K_M \
    --output ./quantized
```

---

## Which format should I use?

Not sure what `Q4_K_M` means? Start here:

- **GGUF:** `Q4_K_M` is the recommended default — best balance of size and quality.
- **AWQ:** Use `int4` — it's the only GPU format.
- **CoreML:** Use `float16` for broadest iOS/macOS compatibility.

See [Concepts → Perplexity vs Speed](../concepts/perplexity-vs-speed.md) for a deeper explanation.

---

## Next Steps

| | |
|---|---|
| 📚 | [Understand the concepts](../concepts/quantization-types.md) — Why do different formats exist? |
| ⚙️ | [Choose your backend](../guides/choosing-a-backend.md) — GGUF, AWQ, or CoreML? |
| 🚀 | [Run your model](../integrations/index.md) — Load into Ollama, llama.cpp, vLLM, or iOS |
| 📖 | [API Reference](../api-reference/quantizer.md) — Full `Quantizer` class docs |

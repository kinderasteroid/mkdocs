# Batch Processing

Quantize multiple models in a single script using a simple loop.

---

## Basic loop

```python
from qwodel import Quantizer

models = [
    "meta-llama/Llama-2-7b-hf",
    "meta-llama/Llama-2-13b-hf",
    "mistralai/Mistral-7B-v0.1",
]

for model_path in models:
    print(f"\nQuantizing {model_path}...")
    try:
        quantizer = Quantizer(
            backend="gguf",
            model_path=model_path,
            output_dir="./quantized"
        )
        output = quantizer.quantize(format="Q4_K_M")
        print(f"Yes Done: {output}")
    except Exception as e:
        print(f"No Failed: {e}")
```

---

## Multiple formats per model

```python
from qwodel import Quantizer

model_path = "./llama-3"
formats = ["Q4_K_M", "Q5_K_M", "Q8_0"]

quantizer = Quantizer(backend="gguf", model_path=model_path, output_dir="./output")

for fmt in formats:
    output = quantizer.quantize(format=fmt)
    print(f"{fmt}: {output}")
```

> **Tip:** You only need to create the `Quantizer` instance once when iterating over formats for the same model.

---

## Multiple backends

```python
from qwodel import Quantizer

model_path = "./my-model"

tasks = [
    {"backend": "gguf",   "format": "Q4_K_M",    "output_dir": "./output/gguf"},
    {"backend": "awq",    "format": "int4",       "output_dir": "./output/awq"},
    {"backend": "coreml", "format": "float16",    "output_dir": "./output/coreml"},
]

for task in tasks:
    quantizer = Quantizer(
        backend=task["backend"],
        model_path=model_path,
        output_dir=task["output_dir"]
    )
    output = quantizer.quantize(format=task["format"])
    print(f"[{task['backend']}] {output}")
```

---

## Error handling

Always wrap each quantization in a `try/except` so one failure doesn't stop the whole batch. The key exceptions are:

| Exception | Cause |
|---|---|
| `ValidationError` | Bad model path or unsupported format |
| `DependencyError` | Missing backend library or binary |
| `QuantizationError` | Runtime failure during quantization |

```python
from qwodel import Quantizer
from qwodel.core.exceptions import ValidationError, DependencyError, QuantizationError

for model_path in models:
    try:
        q = Quantizer(backend="gguf", model_path=model_path, output_dir="./output")
        q.quantize(format="Q4_K_M")
    except ValidationError as e:
        print(f"Bad input: {e}")
    except DependencyError as e:
        print(f"Missing dependency: {e}")
    except QuantizationError as e:
        print(f"Quantization failed: {e}")
```

See [Exceptions →](../api-reference/exceptions.md) for the full list.

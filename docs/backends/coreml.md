# CoreML Backend

Quantizes models for Apple devices (iOS, macOS, iPadOS) using `coremltools`.

**Install:** `pip install qwodel[coreml]`

> Requires macOS. Xcode Command Line Tools must be installed.

---

## Supported Formats

| Format | Compression | Notes |
|---|---|---|
| `float16` | ~2x | Half-precision. Minimal accuracy loss. **Recommended start.** |
| `int8_linear` | ~4x | 8-bit linear quantization. Good accuracy. |
| `int8_symmetric` | ~4x | 8-bit symmetric quantization. Faster ANE ops. |
| `int6` | ~5x | 6-bit palettization. Balance between int4 and int8. |
| `int4` | ~8x | 4-bit palettization. Maximum compression. **iOS 18+ only.** |

---

## Parameters

### `Quantizer(...)` — Initialization

| Parameter | Type | Default | Description |
|---|---|---|---|
| `input_shape` | `tuple` | `(1, 512)` | `(batch_size, seq_length)` for model tracing. |
| `compute_units` | `str` | `"ALL"` | CoreML compute units: `"ALL"`, `"CPU_ONLY"`, `"CPU_AND_GPU"`. |
| `seq_length` | `int` | `512` | Maximum sequence length for dynamic shape range. |

### `quantize(format)` — Runtime

| Parameter | Type | Required | Description |
|---|---|---|---|
| `format` | `str` | Yes | One of the formats listed above. |

---

## Example

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="coreml",
    model_path="./my-model",
    output_dir="./output",
    compute_units="ALL",
    seq_length=512
)
output = quantizer.quantize(format="float16")
print(f"Output: {output}")
```

**CLI:**
```bash
qwodel quantize ./my-model --backend coreml --format float16 --output ./output
```

---

## After Quantization

Your output is a `.mlpackage` directory. Load it into an iOS or macOS app:
[iOS App Integration →](../integrations/ios-app.md)

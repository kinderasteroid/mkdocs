# Quantizer

The main entry point for all quantization operations.

```python
from qwodel import Quantizer
```

---

## Constructor

```python
Quantizer(
    backend,
    model_path,
    output_dir="./quantized_models",
    progress_callback=None,
    **backend_kwargs
)
```

| Parameter | Type | Required | Description |
|---|---|---|---|
| `backend` | `str` | Yes | Backend to use: `"awq"`, `"gguf"`, or `"coreml"`. |
| `model_path` | `str` | Yes | Path to the source model. Can be a local directory, `.gguf` file, or HuggingFace model ID. |
| `output_dir` | `str` | No | Directory to save the output. Defaults to `./quantized_models`. |
| `progress_callback` | `Callable` | No | Optional callback `(percent: int, stage: str, message: str)` for progress updates. |
| `**backend_kwargs` | `Any` | No | Backend-specific arguments — see each [backend page](../backends/index.md). |

---

## Methods

### `quantize(format, **kwargs) → Path`

Runs the quantization process.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `format` | `str` | Yes | Quantization format string (e.g., `"int4"`, `"Q4_K_M"`, `"float16"`). |
| `**kwargs` | `Any` | No | Runtime overrides for backend-specific parameters. |

**Returns:** `Path` to the quantized model file or directory.

---

### `get_model_info() → Dict`

Returns metadata about the quantized model.

| Key | Type | Description |
|---|---|---|
| `source_model` | `str` | Path to the input model. |
| `quantized_model` | `str` | Path to the output model. |
| `backend` | `str` | Backend used. |
| `file_size` | `int` | Output file size in bytes. |
| `input_format` | `str` | Detected format of the source model. |

---

### `list_formats(backend=None) → Dict` *(static)*

Lists available quantization formats.

| Parameter | Type | Description |
|---|---|---|
| `backend` | `str \| None` | Specific backend name, or `None` to list all. |

---

### `list_backends() → List[str]` *(static)*

Returns a list of all registered backend names.

---

## Convenience function

```python
from qwodel import quantize

quantize(
    model_path="./my-model",
    output_dir="./output",
    backend="gguf",
    format="Q4_K_M"
)
```

| Parameter | Type | Required | Description |
|---|---|---|---|
| `model_path` | `str` | Yes | Path to source model. |
| `output_dir` | `str` | Yes | Output directory. |
| `backend` | `str` | Yes | Backend name. |
| `format` | `str` | Yes | Quantization format. |
| `progress_callback` | `Callable` | No | Optional progress callback. |
| `**kwargs` | `Any` | No | Additional backend/format arguments. |

---

## Logging

`qwodel` uses Python's standard `logging` module:

```python
import logging
logging.basicConfig(level=logging.INFO)
```

Logger names: `AWQQuantizer`, `GGUFQuantizer`, `CoreMLQuantizer`.

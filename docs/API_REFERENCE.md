# API Reference

Complete reference for the `qwodel` Python API.

---
# Install Full Qwodel (Note This causes full dependency installation)

### **Install:** `pip install qwodel[all]`
## `Quantizer` Class

The main entry point for all quantization operations.

```python
from qwodel import Quantizer
```

### Constructor

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
| `backend` | `str` | ✅ | Backend to use: `"awq"`, `"gguf"`, or `"coreml"`. |
| `model_path` | `str` | ✅ | Path to the source model. Can be a local directory, `.gguf` file, or HuggingFace model ID. |
| `output_dir` | `str` | ❌ | Directory to save the output. Defaults to `./quantized_models`. |
| `progress_callback` | `Callable` | ❌ | Optional callback `(percent: int, stage: str, message: str)` for progress updates. |
| `**backend_kwargs` | `Any` | ❌ | Backend-specific arguments — see each backend section below. |

---

### Methods

#### `quantize(format, **kwargs) → Path`

Runs the quantization process.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `format` | `str` | ✅ | Quantization format string (e.g., `"int4"`, `"Q4_K_M"`, `"float16"`). |
| `**kwargs` | `Any` | ❌ | Runtime overrides for backend-specific parameters. |

**Returns:** `Path` to the quantized model file or directory.

---

#### `get_model_info() → Dict`

Returns metadata about the quantized model.

| Key | Type | Description |
|---|---|---|
| `source_model` | `str` | Path to the input model. |
| `quantized_model` | `str` | Path to the output model. |
| `backend` | `str` | Backend used. |
| `file_size` | `int` | Output file size in bytes. |
| `input_format` | `str` | Detected format of the source model. |

---

#### `list_formats(backend=None) → Dict` *(static)*

Lists available quantization formats.

| Parameter | Type | Description |
|---|---|---|
| `backend` | `str \| None` | Specific backend name, or `None` to list all. |

---

#### `list_backends() → List[str]` *(static)*

Returns a list of all registered backend names.

---

## Convenience Function

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
| `model_path` | `str` | ✅ | Path to source model. |
| `output_dir` | `str` | ✅ | Output directory. |
| `backend` | `str` | ✅ | Backend name. |
| `format` | `str` | ✅ | Quantization format. |
| `progress_callback` | `Callable` | ❌ | Optional progress callback. |
| `**kwargs` | `Any` | ❌ | Additional backend/format arguments. |

---

## Backends

### AWQ Backend (`backend="awq"`)

GPU-based INT4 quantization using `llm-compressor`. Requires an NVIDIA GPU with CUDA.

**Install:** `pip install qwodel[awq]`

#### Supported Formats

| Format | Description |
|---|---|
| `int4` | 4-bit weight quantization (W4A16). Best for GPU inference. |

#### Initialization Parameters (`Quantizer(...)`)

| Parameter | Type | Default | Description |
|---|---|---|---|
| `calibration_dataset` | `str` | `"wikitext:wikitext-2-raw-v1"` | Dataset for calibration. Supports HF IDs, `repo:subset` syntax, and local `.json`/`.jsonl`/`.txt` files. |
| `calibration_split` | `str` | `"train"` | Dataset split to use. |
| `token` | `str` | `None` | HuggingFace API token for gated/private models. |
| `batch_size` | `int` | Auto | Calibration batch size. Auto-selected based on available VRAM. |
| `seq_length` | `int` | Auto | Max sequence length for calibration. Auto-selected based on VRAM. |
| `num_samples` | `int` | Auto | Number of calibration samples. Auto-selected based on VRAM. |

#### Runtime Overrides (`quantize(...)`)

These can be passed to `quantize()` to override the values set at init time.

| Parameter | Type | Description |
|---|---|---|
| `batch_size` | `int` | Override batch size. |
| `seq_len` | `int` | Override sequence length. |
| `num_samples` | `int` | Override number of calibration samples. |
| `ignore` | `List[str]` | Modules to skip quantization. Supports exact names and `re:` regex patterns (e.g., `["lm_head", "re:.*vision_tower.*"]`). |

#### VRAM Auto-Config

When `batch_size`, `seq_length`, and `num_samples` are not set, they are automatically chosen based on available GPU VRAM headroom:

| VRAM Headroom | `batch_size` | `seq_len` | `num_samples` |
|---|---|---|---|
| < 4 GB | 1 | 2048 | 32 |
| 4–8 GB | 2 | 4096 | 64 |
| 8–16 GB | 4 | 4096 | 128 |
| 16–24 GB | 8 | 8192 | 128 |
| > 24 GB | 16 | 8192 | 256 |


### GGUF Backend (`backend="gguf"`)

CPU-friendly quantization for `llama.cpp`-compatible runtimes.

**Install:** `pip install qwodel[gguf]`

#### Supported Formats

| Format | Description |
|---|---|
| `Q4_K_M` | Best balance of speed and quality. Recommended for most users. |
| `Q8_0` | Near-lossless quality. Requires more RAM. |
| `Q2_K` | Maximum compression. Reduced quality. |
| `Q3_K_M` | 3-bit medium quality. |
| `Q4_0` | Compact 4-bit, slightly smaller than Q4_K_M. |
| `Q4_K_S` | Small 4-bit K-quant. |
| `Q5_K_M` | Better quality than Q4_K_M, slightly larger. |
| `Q5_K_S` | Small 5-bit K-quant. |
| `Q6_K` | High quality between Q8_0 and Q4_K_M. |
| `IQ4_NL` | 4.5 bpw importance-based quantization. |
| `IQ3_M` | 3.66 bpw compact importance quantization. |

#### Initialization Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `model_path` | `str` | — | Path to HuggingFace model directory or existing `.gguf` file. |
| `output_dir` | `str` | `./quantized_models` | Output directory. |

> **Note:** GGUF has no additional backend-specific init parameters.

#### Example

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="gguf",
    model_path="./llama-3",
    output_dir="./output"
)
quantizer.quantize(format="Q4_K_M")
```

---

### CoreML Backend (`backend="coreml"`)

Quantizes models for Apple devices (iOS, macOS, iPadOS) using `coremltools`.

**Install:** `pip install qwodel[coreml]`

#### Supported Formats

| Format | Compression | Notes |
|---|---|---|
| `float16` | ~2x | Half-precision. Minimal accuracy loss. Universal compatibility. |
| `int8_linear` | ~4x | 8-bit linear quantization. Good accuracy. |
| `int8_symmetric` | ~4x | 8-bit symmetric quantization. Faster ops. |
| `int4` | ~8x | 4-bit palettization. iOS 18+ only. |
| `int6` | ~5x | 6-bit palettization. Balance between int4 and int8. |

#### Initialization Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `input_shape` | `tuple` | `(1, 512)` | `(batch_size, seq_length)` for model tracing. |
| `compute_units` | `str` | `"ALL"` | CoreML compute units: `"ALL"`, `"CPU_ONLY"`, `"CPU_AND_GPU"`. |
| `seq_length` | `int` | `512` | Maximum sequence length for dynamic shape range. |

#### Example

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="coreml",
    model_path="./my-model",
    output_dir="./output",
    compute_units="ALL",
    seq_length=512
)
quantizer.quantize(format="int8_linear")
```

---

## Logging

`qwodel` uses Python's standard `logging` module. Configure it in your application:

```python
import logging
logging.basicConfig(level=logging.INFO)
```

The logger name for each backend is its class name (e.g., `AWQQuantizer`, `GGUFQuantizer`, `CoreMLQuantizer`).

---

## Exceptions

| Exception | Description |
|---|---|
| `QuantizationError` | General quantization failure. |
| `ValidationError` | Invalid input path, format, or architecture. |
| `DependencyError` | Missing required library or binary. |
| `FormatNotSupportedError` | Requested format is not supported by the backend. |

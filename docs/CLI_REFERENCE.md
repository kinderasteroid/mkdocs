# CLI Reference

Command-line tools provided by `qwodel`.

---

## Installation

Install with the backend extras you need:

```bash
pip install qwodel[all]     #Everything

pip install qwodel[awq]     # AWQ (GPU)
pip install qwodel[gguf]    # GGUF (CPU)
pip install qwodel[coreml]  # CoreML (Apple)
```

---

## `qwodel` — Main CLI

```bash
qwodel [COMMAND] [OPTIONS]
```

---

### `quantize`

Quantize a model from the command line.

```bash
qwodel quantize MODEL_PATH [OPTIONS]
```

#### Arguments

| Argument | Description | Required |
|---|---|---|
| `MODEL_PATH` | Path to the input model (local directory, `.gguf` file, or HuggingFace model ID). | ✅ |

#### Options

| Option | Short | Type | Default | Description |
|---|---|---|---|---|
| `--backend` | `-b` | `str` | Required | Backend to use: `awq`, `gguf`, `coreml`. |
| `--format` | `-f` | `str` | Required | Quantization format (e.g., `int4`, `Q4_K_M`, `float16`). |
| `--output-dir` | `-o` | `str` | `./quantized_models` | Directory to save the output. |
| `--verbose` | `-v` | flag | `False` | Enable verbose/debug logging. |

#### Examples

```bash
# AWQ INT4 quantization
qwodel quantize ./gemma -b awq -f int4 -o ./output

# GGUF Q4_K_M quantization
qwodel quantize ./llama-3 -b gguf -f Q4_K_M -o ./output

# CoreML INT8 quantization
qwodel quantize ./my-model -b coreml -f int8_linear -o ./output
```

---

### `list-formats`

List all available quantization formats for a backend.

```bash
qwodel list-formats [BACKEND]
```

| Argument | Description | Required |
|---|---|---|
| `BACKEND` | Backend name: `awq`, `gguf`, `coreml`. Omit to list all. | ❌ |

#### Examples

```bash
# List GGUF formats
qwodel list-formats gguf

# List all formats across all backends
qwodel list-formats
```

---

### `check`

Verify your installation and check which backends and dependencies are available.

```bash
qwodel check
```

No arguments or options required.

---

## Format Quick Reference

### AWQ Formats

| Format | Description |
|---|---|
| `int4` | 4-bit weight quantization (W4A16). GPU inference. |

### GGUF Formats

| Format | Description |
|---|---|
| `Q4_K_M` | Best balance of speed and quality. Recommended. |
| `Q8_0` | Near-lossless quality. |
| `Q2_K` | Maximum compression. |
| `Q3_K_M` | 3-bit medium quality. |
| `Q4_0` | Compact 4-bit. |
| `Q4_K_S` | Small 4-bit K-quant. |
| `Q5_K_M` | Better quality than Q4_K_M. |
| `Q5_K_S` | Small 5-bit K-quant. |
| `Q6_K` | High quality. |
| `IQ4_NL` | 4.5 bpw importance-based. |
| `IQ3_M` | 3.66 bpw compact. |

### CoreML Formats

| Format | Compression | Notes |
|---|---|---|
| `float16` | ~2x | Half-precision. Universal. |
| `int8_linear` | ~4x | 8-bit linear. |
| `int8_symmetric` | ~4x | 8-bit symmetric. |
| `int4` | ~8x | iOS 18+ only. |
| `int6` | ~5x | Balance between int4 and int8. |

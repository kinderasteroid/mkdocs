# `qwodel quantize`

Quantize a model from the command line.

```bash
qwodel quantize MODEL_PATH [OPTIONS]
```

---

## Arguments

| Argument | Description | Required |
|---|---|---|
| `MODEL_PATH` | Path to the input model (local directory, `.gguf` file, or HuggingFace model ID). | Yes |

## Options

| Option | Short | Type | Default | Description |
|---|---|---|---|---|
| `--backend` | `-b` | `str` | Required | Backend to use: `awq`, `gguf`, `coreml`. |
| `--format` | `-f` | `str` | Required | Quantization format (e.g., `int4`, `Q4_K_M`, `float16`). |
| `--output-dir` | `-o` | `str` | `./quantized_models` | Directory to save the output. |
| `--verbose` | `-v` | flag | `False` | Enable verbose/debug logging. |

---

## Examples

```bash
# GGUF Q4_K_M (recommended default)
qwodel quantize ./llama-3 -b gguf -f Q4_K_M -o ./output

# AWQ INT4 (GPU)
qwodel quantize ./gemma -b awq -f int4 -o ./output

# CoreML INT8 (Apple)
qwodel quantize ./my-model -b coreml -f int8_linear -o ./output

# From HuggingFace Hub
qwodel quantize meta-llama/Llama-2-7b-hf -b gguf -f Q4_K_M -o ./output
```

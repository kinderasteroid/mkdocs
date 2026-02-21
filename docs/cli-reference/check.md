# `qwodel check`

Inspect your Qwodel installation and verify which backends have their dependencies available.

---

## Usage

```bash
qwodel check
```

No options required. The command exits with code `0` even if some backends are missing — it is informational only.

---

## Example output

```
Qwodel v0.0.12 — Dependency Check
──────────────────────────────────────────────────
 Backend   Status      Notes
──────────────────────────────────────────────────
 gguf      ✓ ready     torch 2.6.0, llama-cpp-python 0.3.4
 awq       ✓ ready     torch 2.6.0, llmcompressor 0.2.1
 coreml    ✗ missing   coremltools not installed
──────────────────────────────────────────────────
2 of 3 backends available.
Run `pip install qwodel[coreml]` to enable the missing backend.
```

---

## What gets checked

| Backend | Dependencies verified |
|---|---|
| `gguf` | `torch`, `llama_cpp` |
| `awq` | `torch`, `llmcompressor`, `accelerate` |
| `coreml` | `coremltools` |

---

## Fix missing backends

```bash
pip install qwodel[gguf]     # enable GGUF
pip install qwodel[awq]      # enable AWQ  (needs NVIDIA GPU + CUDA)
pip install qwodel[coreml]   # enable CoreML  (macOS only)
pip install qwodel[all]      # enable everything
```

---

**See also:** [Installation](../getting-started/installation.md), [list-backends](list-backends.md)

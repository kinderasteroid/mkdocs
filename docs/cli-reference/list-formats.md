# `qwodel list-formats`

List all available quantization formats, optionally filtered by backend.

```bash
qwodel list-formats [BACKEND]
```

---

## Arguments

| Argument | Description | Required |
|---|---|---|
| `BACKEND` | Backend name: `awq`, `gguf`, `coreml`. Omit to list all. | No |

---

## Examples

```bash
# List all formats across all backends
qwodel list-formats

# List GGUF formats only
qwodel list-formats gguf

# List AWQ formats only
qwodel list-formats awq

# List CoreML formats only
qwodel list-formats coreml
```

---

## Quick reference

### AWQ
| Format | Description |
|---|---|
| `int4` | 4-bit weight quantization (W4A16). |

### GGUF
| Format | Description |
|---|---|
| `Q4_K_M` | Best balance — recommended. |
| `Q5_K_M` | Better quality, slightly larger. |
| `Q6_K` | High quality. |
| `Q8_0` | Near-lossless. |
| `Q2_K` | Maximum compression. |
| `Q3_K_M` | 3-bit medium quality. |
| `Q4_0` | Compact 4-bit. |
| `Q4_K_S` | Small 4-bit K-quant. |
| `Q5_K_S` | Small 5-bit K-quant. |
| `IQ4_NL` | 4.5 bpw importance-based. |
| `IQ3_M` | 3.66 bpw compact. |

### CoreML
| Format | Notes |
|---|---|
| `float16` | Universal, minimal quality loss. |
| `int8_linear` | 8-bit linear. |
| `int8_symmetric` | 8-bit symmetric, faster ANE. |
| `int6` | 6-bit palettization. |
| `int4` | iOS 18+ only. |

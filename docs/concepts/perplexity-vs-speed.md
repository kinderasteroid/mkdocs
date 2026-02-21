# Perplexity vs Speed

Quantization is always a trade-off. This page helps you understand the three dimensions of that trade-off — **quality**, **size**, and **speed** — so you can pick the right format for your use case.

---

## What is Perplexity?

**Perplexity** (PPL) is the standard metric for measuring language model quality. Lower is better. A model with perplexity 5.2 is more accurate than one with perplexity 5.9.

When you quantize a model, you slightly reduce its quality (raise its perplexity). The question is: how much quality are you willing to trade for a smaller file and faster inference?

---

## The GGUF Format Ladder

For GGUF, the format name encodes the bit depth. Here's a practical ladder from smallest/fastest to largest/best:

| Format | Bits per weight | Size (7B model) | Perplexity loss | Recommended for |
|---|---|---|---|---|
| `Q2_K` | ~2.6 | ~2.7 GB | High | Experimentation only |
| `Q3_K_M` | ~3.35 | ~3.3 GB | Medium-high | Tight RAM constraints |
| `Q4_0` | 4.0 | ~3.8 GB | Medium | Legacy compatibility |
| `Q4_K_S` | 4.4 | ~4.0 GB | Medium | Slightly better than Q4_0 |
| **`Q4_K_M`** | **4.8** | **~4.1 GB** | **Low** | **Yes Default recommendation** |
| `Q5_K_S` | 5.5 | ~4.6 GB | Very low | More RAM, better quality |
| `Q5_K_M` | 5.7 | ~4.8 GB | Very low | Quality-focused deployments |
| `Q6_K` | 6.6 | ~5.5 GB | Minimal | Near-lossless at 6-bit |
| `Q8_0` | 8.0 | ~7.2 GB | Near-zero | Testing / highest quality |

> **Rule of thumb:** `Q4_K_M` is the right choice for 90% of users. Only go lower if you're constrained by RAM; only go higher if quality is more important than file size.

---

## The "K" Suffix

Formats with `_K` (e.g., `Q4_K_M`, `Q5_K_S`) use **k-quants** — a smarter quantization scheme that uses mixed precision (some layers at higher bits) for better quality at the same average bit width. Always prefer `_K` variants over plain ones (e.g., `Q4_K_M` over `Q4_0`).

---

## AWQ: Always 4-bit

AWQ only supports one format: `int4` (W4A16 — 4-bit weights, 16-bit activations). The quality trade-off is managed by the calibration step, not by choosing a format. AWQ at `int4` typically outperforms GGUF `Q4_K_M` on perplexity because of its activation-aware approach.

---

## CoreML: Float16 vs Int

| Format | Bits | Notes |
|---|---|---|
| `float16` | 16 | Minimal quality loss. Best starting point. |
| `int8_linear` | 8 | ~2x smaller than float16. Good quality. |
| `int8_symmetric` | 8 | Faster ops on ANE. Similar quality. |
| `int6` | 6 | Balance between int4 and int8. |
| `int4` | 4 | Maximum compression. iOS 18+ only. |

---

## Summary: Choosing a Format

```
Prioritize compatibility?      → Q4_K_M (GGUF)
Prioritize quality?            → Q6_K or Q8_0 (GGUF), or AWQ int4 on GPU
Prioritize smallest size?      → Q2_K or Q3_K_M (GGUF), int4 (CoreML)
Targeting iPhone?              → float16 or int8_linear (CoreML)
```

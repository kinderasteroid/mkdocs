# Quantization Types

Quantization compresses a model's weights from high-precision numbers (float32) to lower-precision ones (int4, int8). Less precision = smaller file, faster inference, slight accuracy loss.

There are three major methods you'll encounter: **GGUF**, **AWQ**, and **GPTQ**. Here's how they differ.

---

## At a Glance

| | GGUF | AWQ | GPTQ |
|---|---|---|---|
| **Runs on** | CPU (+ GPU offload) | NVIDIA GPU | NVIDIA GPU |
| **Quantization type** | Post-training (PTQ) | Post-training (PTQ) | Post-training (PTQ) |
| **Calibration data needed** | No No | Yes Yes | Yes Yes |
| **Precision** | 2–8 bit | 4-bit (W4A16) | 4-bit (W4A32) |
| **Primary runtime** | llama.cpp, Ollama | vLLM, LMDeploy | llm-compressor |
| **Supported by Qwodel** | Yes | Yes | No |

---

## GGUF

**GGUF** (GPT-Generated Unified Format) is the format used by [llama.cpp](https://github.com/ggerganov/llama.cpp). It is the most widely compatible format and can run on a CPU alone.

**How it works:** Weights are quantized into fixed-bit buckets (e.g., 4-bit, 8-bit) using simple round-to-nearest math. No calibration data needed.

**Tradeoff:** Because it doesn't use calibration data, more quality is lost at very low bit widths (Q2, Q3) compared to calibration-based methods.

**Use it when:**
- You want the widest runtime support (Ollama, llama.cpp, LM Studio)
- You're deploying on CPU or consumer hardware
- You don't have a GPU for quantization

---

## AWQ (Activation-Aware Weight Quantization)

**AWQ** uses a small calibration dataset to find which weights matter most (those with large activation scales) and protects them from aggressive quantization. This results in better quality than GGUF at equivalent bit widths.

**How it works:** Measures activation magnitudes across calibration samples, scales important weight channels before quantizing to 4-bit, so critical information is preserved.

**Tradeoff:** Requires a CUDA GPU to quantize and to run inference.

**Use it when:**
- You're deploying on NVIDIA GPUs
- You need the best quality/size ratio at 4-bit
- You want to serve via vLLM or similar GPU servers

---

## GPTQ

**GPTQ** (Generative Pre-trained Transformer Quantization) is another calibration-based method. It uses second-order gradient information (inverse Hessian) to minimize quantization error per layer.

> ℹ️ Qwodel does not currently support GPTQ. Use [AutoGPTQ](https://github.com/AutoGPTQ/AutoGPTQ) directly for GPTQ quantization.

---

## CoreML (Apple-specific)

CoreML is not a quantization algorithm — it's Apple's ML inference framework. Qwodel converts and quantizes models into CoreML's `.mlpackage` format using `coremltools`. The quantization methods used are palettization (int4/int6) and linear quantization (int8/float16).

**Use it when:**
- You're targeting iOS, iPadOS, or macOS
- You want on-device inference without network calls

---

## Which should you choose?

```
Running on CPU (any machine)?       → GGUF
Running on NVIDIA GPU (server)?     → AWQ
Deploying to iPhone/Mac?            → CoreML
```

See [Choosing a Backend →](../guides/choosing-a-backend.md) for a more detailed decision guide.

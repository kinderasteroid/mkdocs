# Choosing a Backend

Qwodel supports three quantization backends. This guide helps you pick the right one.

---

## Decision flowchart

```
Are you deploying to an iPhone, iPad, or Mac app?
    └─ Yes → CoreML

Do you have an NVIDIA GPU with CUDA?
    ├─ Yes, and you're serving via vLLM / GPU inference → AWQ
    └─ No (CPU machine, consumer hardware, broad compat.) → GGUF
```

---

## GGUF — Best default choice

**When to use:**
- You want to run on CPU (laptops, servers without GPU)
- You're loading into Ollama, LM Studio, llama.cpp, or a similar runtime
- You need broad file-format compatibility

**Quick example:**
```python
quantizer = Quantizer(backend="gguf", model_path="./model", output_dir="./output")
quantizer.quantize(format="Q4_K_M")
```

**Recommended formats:** `Q4_K_M` (default), `Q5_K_M` (higher quality), `Q8_0` (near-lossless)

---

## AWQ — Best quality on GPU

**When to use:**
- You have an NVIDIA GPU with CUDA 12.1+
- You're hosting with vLLM, LMDeploy, or TGI
- You want the best accuracy at 4-bit

**Quick example:**
```python
quantizer = Quantizer(
    backend="awq",
    model_path="./model",
    output_dir="./output",
    calibration_dataset="wikitext:wikitext-2-raw-v1"
)
quantizer.quantize(format="int4")
```

**Note:** AWQ quantization itself also requires a GPU. You cannot quantize AWQ on CPU.

---

## CoreML — Apple devices only

**When to use:**
- You're building an iOS, iPadOS, or macOS app
- You need on-device inference (no server required)
- Your model will be shipped inside an app bundle

**Quick example:**
```python
quantizer = Quantizer(
    backend="coreml",
    model_path="./model",
    output_dir="./output",
    compute_units="ALL",
    seq_length=512
)
quantizer.quantize(format="float16")
```

**Recommended formats:** `float16` (universal), `int8_linear` (smaller, iOS 16+)

---

## Side-by-side comparison

| | GGUF | AWQ | CoreML |
|---|---|---|---|
| Hardware to quantize | CPU | NVIDIA GPU | Mac (any) |
| Hardware to infer | CPU (+ optional GPU offload) | NVIDIA GPU | Apple chip |
| Min RAM to quantize 7B model | ~16 GB RAM | ~10 GB VRAM | ~16 GB RAM |
| Output file | `.gguf` | `safetensors` dir | `.mlpackage` |
| Runtimes | Ollama, llama.cpp, LM Studio | vLLM, LMDeploy | Core ML framework |

---

**Next:** [Run your quantized model →](../integrations/index.md)

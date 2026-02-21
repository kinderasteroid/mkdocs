# Integrations

This section covers two things:

1. **Sourcing models** — how to get a model into Qwodel (HuggingFace Hub, local disk, post-training).
2. **Deploying output** — how to load a Qwodel-quantized file into popular runtimes.

## Model sources

| Page | Use case |
|---|---|
| [HuggingFace](huggingface.md) | Download a model from HF Hub and quantize it |
| [Local LLM](local-llm.md) | Use a model already on disk (git-lfs clone, compiled build, etc.) |
| [PyTorch Post-Training](pytorch.md) | Quantize your own fine-tuned / LoRA-merged model |

## Deployment runtimes

| Page | Output type | Runtime |
|---|---|---|
| [Ollama](ollama.md) | `.gguf` | Ollama — run models locally with one command |
| [llama-cpp-python](llama-cpp-python.md) | `.gguf` | Python bindings for llama.cpp |
| [vLLM](vllm.md) | AWQ `safetensors` | High-throughput GPU inference server |
| [iOS App (CoreML)](ios-app.md) | `.mlpackage` | On-device inference in Swift/Xcode |

## Which integration do I need?

```
Where is my model?
    ├─ On Hugging Face Hub             → HuggingFace guide
    ├─ Already on disk                 → Local LLM guide
    └─ Just finished training/fine-tuning → PyTorch guide

What backend did I use?
    ├─ backend="gguf"
    │   ├─ Want one-command serving?   → Ollama
    │   └─ Want Python integration?    → llama-cpp-python
    ├─ backend="awq"
    │   └─ GPU serving / batch inference → vLLM
    └─ backend="coreml"
        └─ iOS / macOS app             → iOS App (CoreML)
```

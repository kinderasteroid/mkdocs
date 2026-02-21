# Troubleshooting

Common installation and runtime errors, and how to fix them.

---

## Installation issues

### `build backend is missing the 'build_editable' hook`

**Why:** Older `pip` or `setuptools` versions don't support the editable install hook.

**Fix:**
```bash
pip install --upgrade pip setuptools wheel
pip install -e .
```

---

### `llama-quantize not found`

**Why:** GGUF quantization requires the `llama-quantize` C++ binary, which is separate from the Python package.

**Fix:** Ensure `llama-quantize` is compiled from [llama.cpp](https://github.com/ggerganov/llama.cpp) and available in your `PATH`, or install via the pip package:
```bash
pip install llama-cpp-python
```

---

### `CUDA not available` (AWQ)

**Why:** PyTorch was installed without CUDA support.

**Fix:** Reinstall PyTorch with the correct CUDA wheel:
```bash
pip install torch==2.1.2 torchvision==0.16.2 torchaudio==2.1.2 \
    --index-url https://download.pytorch.org/whl/cu121
```

---

## Runtime issues

### `ModuleNotFoundError: No module named 'qwodel'`

**Fix:** Install in editable mode from the repo root:
```bash
pip install -e .
```

---

### Out of memory (OOM) during AWQ quantization

**Why:** Your GPU doesn't have enough VRAM for the default calibration config.

**Fix:** Reduce batch size and sequence length manually:
```python
quantizer.quantize(format="int4", batch_size=1, seq_len=512, num_samples=32)
```

Or let Qwodel auto-select by not passing any of these — it reads available VRAM headroom automatically.

---

### Failed to load calibration dataset

**Why:** The default dataset (`mit-han-lab/pile-val-backup`) may be unavailable or rate-limited.

**Fix:** Use a different dataset:
```python
Quantizer(
    backend="awq",
    model_path="./model",
    output_dir="./output",
    calibration_dataset="wikitext:wikitext-2-raw-v1"
)
```

---

## Still stuck?

Open an issue on [GitHub](https://github.com/yourusername/qwodel/issues) with:
- Your OS and Python version
- The full error traceback
- The command or code you ran

# Installation

Qwodel uses an **extras** system — install only the backend(s) you need.

---

## Requirements

- Python **3.9+**
- pip **23+** (upgrade with `pip install --upgrade pip setuptools wheel`)

---

## Install Options

### Full Install (everything)

```bash
pip install qwodel[all]
```

> **Note:** This installs all three backends and all dependencies. Only do this if you need all three.

---

### GGUF — CPU Quantization

```bash
pip install qwodel[gguf]
```

No GPU required. Works on any machine.

---

### AWQ — GPU Quantization


```bash
pip install qwodel[awq]
```

**Requirements:** NVIDIA GPU, CUDA 12.1+

---

### CoreML — Apple Devices

```bash
pip install qwodel[coreml]
```

**Requirements:** macOS with Apple Silicon or Intel, Xcode CLI tools

---

## Verify Installation

```bash
qwodel check
```

This prints which backends are available and their dependency status.

---

## Troubleshooting

Run into issues? See the [Troubleshooting guide](../help/troubleshooting.md).

---

**Next:** [Quick Start →](quick-start.md)

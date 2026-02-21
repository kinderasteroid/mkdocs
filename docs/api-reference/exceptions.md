# Exceptions

All Qwodel exceptions are importable from `qwodel.core.exceptions`.

```python
from qwodel.core.exceptions import QuantizationError, ValidationError, DependencyError, FormatNotSupportedError
```

---

## Exception Hierarchy

```
Exception
└── QwodelError
    ├── QuantizationError         # General quantization failure
    ├── ValidationError           # Invalid input: path, format, or architecture
    ├── DependencyError           # Missing required library or binary
    └── FormatNotSupportedError   # Requested format not supported by backend
```

---

## Reference

| Exception | When it's raised |
|---|---|
| `QuantizationError` | The quantization process itself failed (runtime error during conversion). |
| `ValidationError` | Invalid `model_path`, unsupported architecture, or bad format string. |
| `DependencyError` | A required library (`autoawq`, `coremltools`, etc.) or binary (`llama-quantize`) is not installed. |
| `FormatNotSupportedError` | The requested format string is not valid for the chosen backend. |

---

## Usage example

```python
from qwodel import Quantizer
from qwodel.core.exceptions import (
    ValidationError,
    DependencyError,
    FormatNotSupportedError,
    QuantizationError,
)

try:
    q = Quantizer(backend="gguf", model_path="./model", output_dir="./output")
    q.quantize(format="Q4_K_M")
except ValidationError as e:
    print(f"Bad model path or format: {e}")
except DependencyError as e:
    print(f"Missing dependency: {e}")
except FormatNotSupportedError as e:
    print(f"Format not supported: {e}")
except QuantizationError as e:
    print(f"Quantization failed: {e}")
```

# Progress Callbacks

Track quantization progress in real time by passing a callback function to the `Quantizer`.

---

## Callback signature

```python
def my_callback(percent: int, stage: str, message: str = "") -> None:
    ...
```

| Argument | Type | Description |
|---|---|---|
| `percent` | `int` | Progress from `0` to `100` |
| `stage` | `str` | Current stage name (e.g., `"Loading"`, `"Quantizing"`, `"Saving"`) |
| `message` | `str` | Optional detail message |

---

## Simple print callback

```python
from qwodel import Quantizer

def on_progress(percent: int, stage: str, message: str = ""):
    print(f"[{percent:3d}%] {stage}: {message}")

quantizer = Quantizer(
    backend="gguf",
    model_path="./model",
    output_dir="./output",
    progress_callback=on_progress
)
quantizer.quantize(format="Q4_K_M")
```

---

## Inline progress bar

```python
def progress_bar(percent: int, stage: str, message: str = ""):
    bar_length = 30
    filled = int(bar_length * percent / 100)
    bar = "█" * filled + "░" * (bar_length - filled)
    print(f"\r[{bar}] {percent:3d}% | {stage}", end="", flush=True)
    if percent == 100:
        print()  # newline at completion

quantizer = Quantizer(
    backend="gguf",
    model_path="./model",
    output_dir="./output",
    progress_callback=progress_bar
)
quantizer.quantize(format="Q4_K_M")
```

Output:
```
[██████████████████░░░░░░░░░░░░]  60% | Quantizing
```

---

## Integration with tqdm

```python
from tqdm import tqdm
from qwodel import Quantizer

pbar = tqdm(total=100, desc="Quantizing", unit="%")
last_pct = [0]

def tqdm_callback(percent: int, stage: str, message: str = ""):
    pbar.update(percent - last_pct[0])
    pbar.set_description(stage)
    last_pct[0] = percent
    if percent == 100:
        pbar.close()

quantizer = Quantizer(
    backend="gguf",
    model_path="./model",
    output_dir="./output",
    progress_callback=tqdm_callback
)
quantizer.quantize(format="Q4_K_M")
```

---

## Logging to a file

```python
import logging

logging.basicConfig(
    filename="quantization.log",
    level=logging.INFO,
    format="%(asctime)s %(message)s"
)

def log_callback(percent: int, stage: str, message: str = ""):
    logging.info(f"[{percent}%] {stage}: {message}")

quantizer = Quantizer(
    backend="gguf",
    model_path="./model",
    output_dir="./output",
    progress_callback=log_callback
)
quantizer.quantize(format="Q4_K_M")
```

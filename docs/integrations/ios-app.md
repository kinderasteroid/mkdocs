# Using CoreML Models in an iOS App

After quantizing a model with Qwodel's CoreML backend, you get a `.mlpackage` directory that can be loaded directly in an iOS or macOS app using Swift's `CoreML` framework.

---

## Prerequisites

- Xcode 15+
- iOS 16+ target (iOS 18+ for `int4` palettization)
- A `.mlpackage` output from Qwodel

---

## Step 1: Quantize your model

```python
from qwodel import Quantizer

quantizer = Quantizer(
    backend="coreml",
    model_path="./my-model",
    output_dir="./output",
    compute_units="ALL",
    seq_length=256
)
quantizer.quantize(format="float16")
# Output: ./output/my-model-float16.mlpackage
```

---

## Step 2: Add to Xcode project

1. Open your Xcode project.
2. Drag the `.mlpackage` folder into the Xcode file navigator.
3. In the dialog, check **"Copy items if needed"** and select your app target.
4. Xcode will automatically generate a Swift class from the model (e.g., `MyModelFloat16`).

---

## Step 3: Load and run inference in Swift

```swift
import CoreML

// Load the model
guard let modelURL = Bundle.main.url(forResource: "my-model-float16", withExtension: "mlpackage") else {
    fatalError("Model not found in bundle")
}

let config = MLModelConfiguration()
config.computeUnits = .all   // Use Neural Engine + GPU + CPU

let model = try! MLModel(contentsOf: modelURL, configuration: config)

// Prepare input (shape depends on your model's input_shape parameter)
let inputArray = try! MLMultiArray(shape: [1, 256], dataType: .float16)
// ... fill inputArray with your tokenized input ...

let input = try! MLDictionaryFeatureProvider(dictionary: ["input_ids": inputArray])

// Run inference
let output = try! model.prediction(from: input)
```

---

## Compute units reference

Set `compute_units` when creating the `Quantizer` (maps to `MLModelConfiguration.computeUnits`):

| Qwodel value | Swift value | What it uses |
|---|---|---|
| `"ALL"` | `.all` | Neural Engine + GPU + CPU (fastest) |
| `"CPU_AND_GPU"` | `.cpuAndGPU` | GPU + CPU (no Neural Engine) |
| `"CPU_ONLY"` | `.cpuOnly` | CPU only |

---

## Format compatibility

| Format | Min iOS | Notes |
|---|---|---|
| `float16` | iOS 16 | Universal — use this first |
| `int8_linear` | iOS 16 | Good quality, smaller |
| `int8_symmetric` | iOS 16 | Faster on ANE |
| `int6` | iOS 17 | Palettization |
| `int4` | iOS 18 | Maximum compression |

---

## Tips

- Start with `float16` to validate correctness, then switch to `int8` or `int4` to reduce app size.
- Use [Core ML Tools](https://coremltools.readme.io/) to inspect and validate your model before adding it to Xcode.
- If the model is large, consider [On-Demand Resources](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/) to avoid bloating your initial app download.

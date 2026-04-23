---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-source/
feedback: LLMNET
version: 26.5.0
title: Model source parameters
description: Configure where Aspose.LLM for .NET loads a model from — local file, Aspose model repository, or Hugging Face — including the resolution order.
keywords:
- model source
- ModelSourceParameters
- Hugging Face
- GGUF
- ModelFilePath
- AsposeModelId
- HuggingFaceRepoId
---

`ModelSourceParameters` tells the engine where to find the model file. The same class type is used in two places on every preset:

- **`BaseModelSourceParameters`** — the main language model (text or vision).
- **`MmprojSourceParameters`** — the vision projector (`mmproj`) for vision presets. Ignored for text-only presets.

## Class reference

```csharp
namespace Aspose.LLM.Abstractions.Parameters;

public class ModelSourceParameters
{
    public string? ModelFilePath { get; set; }          // priority 1 — local path
    public string? AsposeModelId { get; set; }          // priority 2 — Aspose catalog
    public string? HuggingFaceRepoId { get; set; }      // priority 3 — HF repo ID
    public string? HuggingFaceFileName { get; set; }    //             HF file within the repo
}
```

All four properties are nullable. Leave a property `null` to defer to the next priority source.

## Resolution order

At `AsposeLLMApi.Create`, the engine walks the sources in priority order and uses the first one that resolves.

| Priority | Field(s) | Behavior |
|---|---|---|
| 1 | `ModelFilePath` | If set to a non-null non-empty path, load directly from disk. No download. |
| 2 | `AsposeModelId` | If set, download the named model from Aspose's internal catalog into the cache. |
| 3 | `HuggingFaceRepoId` + `HuggingFaceFileName` | If both set, download the named file from the Hugging Face repo into the cache. |

The engine throws an exception if none of the three resolve.

## Where models are cached

Downloaded models live in `EngineParameters.ModelCachePath`. The default on Windows is `%LOCALAPPDATA%\Aspose.LLM\models`; on Linux and macOS it is the equivalent `LocalApplicationData` folder. Change the cache location per preset on [`EngineParameters.ModelCachePath`](/net/developer-reference/parameters/engine/).

When a model has already been downloaded, subsequent runs load from the cache. Delete the cache to force a re-download.

## Typical recipes

### Use a built-in preset's model source

Built-in presets set `HuggingFaceRepoId` and `HuggingFaceFileName` on `BaseModelSourceParameters` in their constructor — no changes needed:

```csharp
var preset = new Qwen25Preset();
// BaseModelSourceParameters.HuggingFaceRepoId = "bartowski/Qwen2.5-7B-Instruct-GGUF"
// BaseModelSourceParameters.HuggingFaceFileName = "Qwen2.5-7B-Instruct-Q4_K_M.gguf"

using var api = AsposeLLMApi.Create(preset);
```

See [Supported presets](/net/product-overview/supported-presets/) for the exact values per preset.

### Load a model from a local file

Override `ModelFilePath` to skip the download entirely. Useful for air-gapped environments or when you already have the GGUF on disk.

```csharp
var preset = new Qwen25Preset();
preset.BaseModelSourceParameters.ModelFilePath = @"C:\models\Qwen2.5-7B-Instruct-Q4_K_M.gguf";
// ModelFilePath wins over the preset's Hugging Face values.

using var api = AsposeLLMApi.Create(preset);
```

### Switch to a different Hugging Face quantization

Keep the preset but change the file name to a different quantization available in the same repo:

```csharp
var preset = new Qwen25Preset();
preset.BaseModelSourceParameters.HuggingFaceFileName = "Qwen2.5-7B-Instruct-Q8_0.gguf";

using var api = AsposeLLMApi.Create(preset);
```

Q8_0 is larger than Q4_K_M (about 2× the file size) but retains more of the original model's quality.

### Bring a custom model from Hugging Face

For a model that does not have a built-in preset, create a custom preset and set `BaseModelSourceParameters` from scratch:

```csharp
public class MyCustomPreset : PresetCoreBase
{
    public MyCustomPreset()
    {
        BaseModelSourceParameters.HuggingFaceRepoId = "your-org/your-model-GGUF";
        BaseModelSourceParameters.HuggingFaceFileName = "your-model.Q4_K_M.gguf";
    }
}
```

See [Custom preset](/net/use-cases/custom-preset/) for the full pattern.

### Configure a vision projector

Vision presets also set `MmprojSourceParameters` — the projector follows the same resolution rules as the base model:

```csharp
var preset = new Qwen3VL2BPreset();
// BaseModelSourceParameters.HuggingFaceRepoId     = "Qwen/Qwen3-VL-2B-Instruct-GGUF"
// BaseModelSourceParameters.HuggingFaceFileName   = "Qwen3VL-2B-Instruct-Q4_K_M.gguf"
// MmprojSourceParameters.HuggingFaceRepoId        = "Qwen/Qwen3-VL-2B-Instruct-GGUF"
// MmprojSourceParameters.HuggingFaceFileName      = "mmproj-Qwen3VL-2B-Instruct-Q8_0.gguf"

// Override the projector to a local file:
preset.MmprojSourceParameters.ModelFilePath = @"C:\models\custom-mmproj.gguf";
preset.MmprojSourceParameters.HuggingFaceRepoId = null;
preset.MmprojSourceParameters.HuggingFaceFileName = null;

using var api = AsposeLLMApi.Create(preset);
```

Leaving the Hugging Face fields on an override is harmless — `ModelFilePath` wins — but clearing them keeps the intent obvious.

## What's next

- [Engine parameters](/net/developer-reference/parameters/engine/) — change the cache location.
- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — pre-populate native binaries for offline scenarios.
- [Custom preset](/net/use-cases/custom-preset/) — full preset-customization patterns.

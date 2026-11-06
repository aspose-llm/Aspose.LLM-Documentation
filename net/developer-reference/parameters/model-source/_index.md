---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-source/
feedback: LLMNET
version: 26.5.0
title: Model source parameters
description: Configure a local model file or Hugging Face source in Aspose.LLM for .NET.
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

- **`BaseModelSourceParameters`**: the main language model (text or vision).
- **`MmprojSourceParameters`**: the vision projector (`mmproj`) for vision presets. Ignored for text-only presets.

## Class reference

```csharp
namespace Aspose.LLM.Abstractions.Parameters;

public class ModelSourceParameters
{
    public string? ModelFilePath { get; set; }          // priority 1: local path
    public string? AsposeModelId { get; set; }          // reserved; ignored
    public string? HuggingFaceRepoId { get; set; }      // remote source when no local path is set
    public string? HuggingFaceFileName { get; set; }    //             HF file within the repo
}
```

All four properties are nullable. `AsposeModelId` is reserved and does not participate in source selection.

## Detailed field reference

Each field has a dedicated page with full defaults, scenario tables, code examples, and interactions.

- [ModelFilePath](/llm/net/developer-reference/parameters/model-source/model-file-path/)
- [AsposeModelId](/llm/net/developer-reference/parameters/model-source/aspose-model-id/)
- [HuggingFaceRepoId](/llm/net/developer-reference/parameters/model-source/hugging-face-repo-id/)
- [HuggingFaceFileName](/llm/net/developer-reference/parameters/model-source/hugging-face-file-name/)

## Resolution order

At `AsposeLLMApi.Create`, the engine walks the sources in priority order and uses the first one that resolves.

| Priority | Field(s) | Behavior |
|---|---|---|
| 1 | `ModelFilePath` | If set to a non-null non-empty path, load directly from disk. No download. |
| 2 | `HuggingFaceRepoId` + optional `HuggingFaceFileName` | Resolve a file from the Hugging Face repository when no local path is set. |

`AsposeModelId` is ignored. Version 26.11.0 may fetch the configured remote model without a separate opt-in when no local path is supplied; the opt-in control is not yet implemented. Obtain the model after reviewing its publisher's license and set `ModelFilePath` to the local file before creating the API.

## Where models are cached

Downloaded models live in `EngineParameters.ModelCachePath`. The default on Windows is `%LOCALAPPDATA%\Aspose.LLM\models`; on Linux and macOS it is the equivalent `LocalApplicationData` folder. Change the cache location per preset on [`EngineParameters.ModelCachePath`](/llm/net/developer-reference/parameters/engine/).

Previously obtained files can be reused from the cache.

## Typical recipes

### Use a built-in preset's model source

Built-in presets set `HuggingFaceRepoId` and `HuggingFaceFileName` on `BaseModelSourceParameters` in their constructor. Inspect the source and license before obtaining the model:

```csharp
var preset = new Qwen25Preset();
// BaseModelSourceParameters.HuggingFaceRepoId = "bartowski/Qwen2.5-7B-Instruct-GGUF"
// BaseModelSourceParameters.HuggingFaceFileName = "Qwen2.5-7B-Instruct-Q4_K_M.gguf"

```

See [Supported presets](/llm/net/product-overview/supported-presets/) for the exact values per preset.

### Load a model from a local file

Override `ModelFilePath` to skip the download entirely. Useful for air-gapped environments or when you already have the GGUF on disk.

```csharp
var preset = new Qwen25Preset();
var modelPath = Environment.GetEnvironmentVariable("ASPOSE_LLM_MODEL_FILE");
if (string.IsNullOrWhiteSpace(modelPath) || !File.Exists(modelPath))
    throw new InvalidOperationException("Provide a local Qwen 2.5 7B Instruct GGUF via ASPOSE_LLM_MODEL_FILE. Preset source: https://huggingface.co/bartowski/Qwen2.5-7B-Instruct-GGUF; model license: https://www.apache.org/licenses/LICENSE-2.0");
preset.BaseModelSourceParameters.ModelFilePath = modelPath;
// ModelFilePath wins over the preset's Hugging Face values.

using var api = AsposeLLMApi.Create(preset);
```

### Switch to a different Hugging Face quantization

The following configures a different file name. Obtain it separately after reviewing the source and license, and set `ModelFilePath` before creating the API:

```csharp
var preset = new Qwen25Preset();
preset.BaseModelSourceParameters.HuggingFaceFileName = "Qwen2.5-7B-Instruct-Q8_0.gguf";

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

See [Custom preset](/llm/net/use-cases/custom-preset/) for the full pattern.

### Configure a vision projector

Vision presets also set `MmprojSourceParameters`: the projector follows the same resolution rules as the base model:

```csharp
var preset = new Qwen3VL2BPreset();
// BaseModelSourceParameters.HuggingFaceRepoId     = "Qwen/Qwen3-VL-2B-Instruct-GGUF"
// BaseModelSourceParameters.HuggingFaceFileName   = "Qwen3VL-2B-Instruct-Q4_K_M.gguf"
// MmprojSourceParameters.HuggingFaceRepoId        = "Qwen/Qwen3-VL-2B-Instruct-GGUF"
// MmprojSourceParameters.HuggingFaceFileName      = "mmproj-Qwen3VL-2B-Instruct-Q8_0.gguf"

// Set both base-model and projector paths before creating the API.
preset.BaseModelSourceParameters.ModelFilePath = Environment.GetEnvironmentVariable("ASPOSE_LLM_MODEL_FILE");
preset.MmprojSourceParameters.ModelFilePath = Environment.GetEnvironmentVariable("ASPOSE_LLM_MMPROJ_FILE");
preset.MmprojSourceParameters.HuggingFaceRepoId = null;
preset.MmprojSourceParameters.HuggingFaceFileName = null;

```

Leaving the Hugging Face fields on an override is harmless: `ModelFilePath` wins, but clearing them keeps the intent obvious.

## What's next

- [Engine parameters](/llm/net/developer-reference/parameters/engine/): change the cache location.
- [Binary manager parameters](/llm/net/developer-reference/parameters/binary-manager/): pre-populate native binaries for offline scenarios.
- [Custom preset](/llm/net/use-cases/custom-preset/): full preset-customization patterns.

---
weight: 40
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/use-cases/custom-preset/
feedback: LLMNET
title: Custom preset
description: Use or create a preset with custom parameters.
keywords:
- custom
- preset
- parameters
---

You can use a built-in preset as-is, or customize parameters by creating a class that extends `PresetCoreBase` or by modifying a preset instance before creating the API.

## Extend a built-in preset

Derive from an existing preset and override or set properties in the constructor:

```csharp
using Aspose.LLM.Abstractions.Parameters.Presets;
using Aspose.LLM.Abstractions.Models;

public class MyQwenPreset : Qwen25Preset
{
    public MyQwenPreset()
    {
        ContextParameters.ContextSize = 8192;
        BaseModelInferenceParameters.GpuLayers = 0; // CPU only
    }
}

// Use it
using var api = AsposeLLMApi.Create(new MyQwenPreset());
```

## Create a preset from base

For full control, extend `PresetCoreBase` and set all required properties (model source, context size, chat template, etc.):

```csharp
public class MyPreset : PresetCoreBase
{
    public MyPreset()
    {
        BaseModelSourceParameters.HuggingFaceRepoId = "your/repo";
        BaseModelSourceParameters.HuggingFaceFileName = "model.gguf";
        ContextParameters.ContextSize = 4096;
        // ... set other parameters
    }
}
```

Refer to existing presets in the codebase (e.g. `Qwen25Preset`, `Gemma3Preset`) for typical values. See [Presets](/net/developer-reference/presets/) and [Supported presets](/net/product-overview/supported-presets/) for more information.

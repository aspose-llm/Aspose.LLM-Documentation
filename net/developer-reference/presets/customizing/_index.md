---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/presets/customizing/
feedback: LLMNET
version: 26.5.0
title: Customizing presets
description: Override fields on built-in Aspose.LLM for .NET presets — either instance-level tweaks or a subclass for reusable configuration.
keywords:
- preset customization
- override
- subclass
- SystemPrompt
- Temperature
- ContextSize
---

Every built-in preset is a plain class with public settable fields on nine parameter bags. Customize a preset by mutating fields before calling `AsposeLLMApi.Create`, or by creating a subclass that applies your defaults in its constructor.

For customizations that substitute entire services (custom model loaders, custom prompt formatters), see [Extensibility](/net/developer-reference/extensibility/).

## Two patterns

### Instance override

Good for one-off changes in a single method or application startup.

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen25Preset();

preset.ContextParameters.ContextSize = 16384;
preset.SamplerParameters.Temperature = 0.2f;
preset.BaseModelInferenceParameters.GpuLayers = 0;
preset.ChatParameters.SystemPrompt = "You are a terse expert.";

using var api = AsposeLLMApi.Create(preset);
```

Mutations after `Create` have no effect — the engine has read the preset. Set everything first.

### Subclass

Good when you use the same tuning in multiple places or want a reusable configuration.

```csharp
using Aspose.LLM.Abstractions.Parameters.Presets;

public class TerseQwenPreset : Qwen25Preset
{
    public TerseQwenPreset()
    {
        ContextParameters.ContextSize = 16384;

        SamplerParameters.Temperature = 0.2f;
        SamplerParameters.TopP = 0.9f;

        BaseModelInferenceParameters.GpuLayers = 999;

        ChatParameters.SystemPrompt = "You are a terse expert. Answer in one sentence.";
        ChatParameters.MaxTokens = 256;
    }
}

// Use everywhere:
using var api = AsposeLLMApi.Create(new TerseQwenPreset());
```

Subclasses inherit the base preset's model source and chat template — you change only the fields you care about.

## Most-overridden fields

| Field | Bag | Typical reason |
|---|---|---|
| `ContextSize` | `ContextParameters` | Reduce for memory, raise for long conversations. |
| `Temperature` | `SamplerParameters` | Lower for determinism, raise for variety. |
| `GpuLayers` | `BaseModelInferenceParameters` | `0` for CPU-only, `999` for full offload. |
| `SystemPrompt` | `ChatParameters` | Define the assistant's role and tone. |
| `MaxTokens` | `ChatParameters` | Cap response length per turn. |
| `CacheCleanupStrategy` | `ChatParameters` | Long conversations — pick the eviction policy. |
| `PreferredAcceleration` | `BinaryManagerParameters` | Force a specific GPU backend. |
| `ModelCachePath` | `EngineParameters` | Shared model cache across processes. |

See the individual [parameter reference pages](/net/developer-reference/parameters/) for each bag's full field list and semantics.

## Common recipes

### Deterministic output

```csharp
preset.SamplerParameters.Temperature = 0.0f;
preset.SamplerParameters.Seed = 42;
```

Output is reproducible across runs.

### Concise enterprise assistant

```csharp
preset.ChatParameters.SystemPrompt =
    "You are a concise enterprise assistant. Each answer fits in two sentences.";
preset.ChatParameters.MaxTokens = 200;
preset.SamplerParameters.Temperature = 0.3f;
```

### CPU-only deployment

```csharp
preset.BaseModelInferenceParameters.GpuLayers = 0;
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.AVX2;
preset.ContextParameters.ContextSize = 4096; // save memory
```

### Reasoning-model-friendly budget

```csharp
var preset = new DeepseekR1Qwen3Preset();
preset.ChatParameters.MaxTokens = 2048; // room for <think> block plus answer
preset.ChatParameters.SystemPrompt = "You are a careful analyst.";
```

### Memory-tight GPU

```csharp
preset.BaseModelInferenceParameters.GpuLayers = 28;   // partial offload
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
preset.ContextParameters.TypeV = GgmlType.Q8_0;        // halve V-cache memory
```

## What's next

- [Creating from scratch](/net/developer-reference/presets/creating-from-scratch/) — new preset for a custom GGUF.
- [Parameters reference](/net/developer-reference/parameters/) — every knob on every bag.
- [Custom preset use case](/net/use-cases/custom-preset/) — runnable end-to-end example.

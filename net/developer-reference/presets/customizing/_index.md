---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/presets/customizing/
feedback: LLMNET
version: 26.5.0
title: Customizing presets
description: Override fields on built-in Aspose.LLM for .NET presets, either instance-level tweaks or a subclass for reusable configuration.
keywords:
- preset customization
- override
- subclass
- SystemPrompt
- Temperature
- ContextSize
---

Every built-in preset is a plain class with public settable fields on nine parameter bags. Customize a preset by mutating fields before calling `AsposeLLMApi.Create`, or by creating a subclass that applies your defaults in its constructor.

{{% alert color="warning" %}}
**No built-in LLM is included in Aspose.LLM.** The library is a local inference runtime: you choose which open source model to use, and the model file is obtained separately and stored on your own machine. Model files are covered by the license of the model publisher, not by your license agreement with Aspose Pty Ltd.

Check the license of the model a preset resolves to before using it commercially. Licenses are listed in [Supported presets](/llm/net/product-overview/supported-presets/).
{{% /alert %}}

For customizations that substitute entire services (custom model loaders, custom prompt formatters), see [Extensibility](/llm/net/developer-reference/extensibility/).

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

Mutations after `Create` have no effect: the engine has read the preset. Set everything first.

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

Subclasses inherit the base preset's model source and chat template: you change only the fields you care about.

### Built-in subclass: the `*PresetCpu` twin

27 of the text presets ship with a ready-made subclass for CPU-only inference: `Llm31_8BPresetCpu`, `Mistral7PresetCpu`, `Hermes3_8BPresetCpu`, and so on. The twin inherits the GPU parent and applies CPU-friendly defaults: zero GPU offload, context capped at 4 K, batch and ubatch shrunk, `FlashAttention` and KV-cache offload disabled. Reach for the twin instead of toggling `GpuLayers = 0` by hand when you want the canonical CPU configuration.

Not every preset has one: check [CPU-tuned variants](/llm/net/product-overview/supported-presets/#cpu-tuned-variants-presetcpu) for the full list and for the presets that have no twin.

```csharp
using var api = AsposeLLMApi.Create(new Llm31_8BPresetCpu());
```

The twin is a vanilla `public sealed class`: it follows exactly the subclass pattern shown above, just authored once in the SDK so every consumer gets the same CPU settings. See [CPU-only deployment](/llm/net/use-cases/cpu-only-deployment/) for the full pattern.

## Most-overridden fields

| Field | Bag | Typical reason |
|---|---|---|
| `ContextSize` | `ContextParameters` | Reduce for memory, raise for long conversations. |
| `Temperature` | `SamplerParameters` | Lower for determinism, raise for variety. |
| `GpuLayers` | `BaseModelInferenceParameters` | `0` for CPU-only, `999` for full offload. |
| `SystemPrompt` | `ChatParameters` | Define the assistant's role and tone. |
| `MaxTokens` | `ChatParameters` | Cap response length per turn. |
| `CacheCleanupStrategy` | `ChatParameters` | Long conversations: pick the eviction policy. |
| `PreferredAcceleration` | `BinaryManagerParameters` | Force a specific GPU backend. |
| `ModelCachePath` | `EngineParameters` | Shared model cache across processes. |

See the individual [parameter reference pages](/llm/net/developer-reference/parameters/) for each bag's full field list and semantics.

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

## Before you ship

{{% alert color="warning" %}}
**Check the license of the model you selected.** Aspose.LLM supplies the runtime, not the model. Whichever model you load, its terms come from the party that published it and they apply to your product. They are not part of, and are not covered by, your license agreement with Aspose Pty Ltd. Some open source models allow commercial use with no strings attached, others attach conditions such as attribution or an acceptable use policy, and a few exclude commercial use or withdraw it above a revenue threshold. [Supported LLMs](/llm/net/product-overview/supported-llms/) lists the license of every family the SDK ships a preset for.
{{% /alert %}}

## What's next

- [Creating from scratch](/llm/net/developer-reference/presets/creating-from-scratch/): new preset for a custom GGUF.
- [Parameters reference](/llm/net/developer-reference/parameters/): every knob on every bag.
- [Custom preset use case](/llm/net/use-cases/custom-preset/): runnable end-to-end example.

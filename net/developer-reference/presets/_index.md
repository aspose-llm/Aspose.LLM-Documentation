---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/presets/
feedback: LLMNET
version: 26.5.0
title: Presets
description: Preset base class, parameter bags, and patterns for using and customizing presets in Aspose.LLM for .NET.
keywords:
- preset
- PresetCoreBase
- model
- parameters
- override
- custom
---

A **preset** bundles every parameter needed to configure a language model for inference: where to find the model file, how to load it, how to generate text, and — for vision presets — how to process images. Pass a preset to `AsposeLLMApi.Create(preset)` and the engine applies the whole configuration at once.

## Base type

All presets derive from `PresetCoreBase` in namespace `Aspose.LLM.Abstractions.Parameters.Presets`. The base type exposes nine parameter bags, each an independently overridable POCO:

| Property | Type | Purpose |
|---|---|---|
| `BaseModelSourceParameters` | [`ModelSourceParameters`](/net/developer-reference/parameters/model-source/) | Where to load the main model (local path, Aspose model ID, or Hugging Face repo and file name). |
| `MmprojSourceParameters` | [`ModelSourceParameters`](/net/developer-reference/parameters/model-source/) | Optional vision projector (mmproj). Set by vision presets; empty on text presets. |
| `BinaryManagerParameters` | [`BinaryManagerParameters`](/net/developer-reference/parameters/binary-manager/) | Native `llama.cpp` release tag, binary cache path, and preferred acceleration backend. |
| `EngineParameters` | [`EngineParameters`](/net/developer-reference/parameters/engine/) | Engine-wide settings: model cache path, debug logging, log directory, default threads. |
| `ChatParameters` | [`ChatParameters`](/net/developer-reference/parameters/chat/) | System prompt, optional history, max tokens, cache cleanup strategy. |
| `ContextParameters` | [`ContextParameters`](/net/developer-reference/parameters/context/) | `llama.cpp` context knobs: context size, batch sizes, rope scaling, flash attention, KV cache dtype. |
| `SamplerParameters` | [`SamplerParameters`](/net/developer-reference/parameters/sampler/) | Sampler knobs: temperature, top-p, top-k, min-p, penalties, DRY, XTC, mirostat, seed, logit bias. |
| `BaseModelInferenceParameters` | [`ModelInferenceParameters`](/net/developer-reference/parameters/model-inference/) | Model-load knobs: GPU layers, main GPU, split mode, tensor split, memory mapping, KV overrides. |
| `MtmdContextParameters` | [`MultimodalContextParameters`](/net/developer-reference/parameters/multimodal-context/) | `mtmd` (vision) context: GPU use, timings, thread count, verbosity, media marker. Set by vision presets. |

Every bag is lazy-initialized — accessing any property is safe even if you did not set it explicitly.

## Create the API with a preset

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(preset);
```

The engine reads every bag on the preset during construction. Mutations to the preset **after** `Create` returns do not affect the alive engine — set everything you need first.

## Override preset parameters before `Create`

You can tweak any field on any bag before passing the preset to `Create`.

```csharp
var preset = new Qwen25Preset();

preset.ContextParameters.ContextSize = 8192;            // smaller context = less memory
preset.SamplerParameters.Temperature = 0.2f;            // more deterministic output
preset.BaseModelInferenceParameters.GpuLayers = 0;      // force CPU-only inference

using var api = AsposeLLMApi.Create(preset);
```

Common overrides:

- **`ContextParameters.ContextSize`** — reduce to save memory or raise to fit long conversations.
- **`SamplerParameters.Temperature`** — lower for deterministic tasks, higher for creative output.
- **`BaseModelInferenceParameters.GpuLayers`** — control GPU offload (0 = CPU only; 999 = full offload).
- **`BinaryManagerParameters.PreferredAcceleration`** — force CUDA, Metal, Vulkan, HIP, or CPU.
- **`ChatParameters.SystemPrompt`** — set a default system prompt for new sessions.
- **`ChatParameters.MaxTokens`** — cap the length of a single response.

See the [custom preset](/net/use-cases/custom-preset/) use case for longer examples.

## Extend a built-in preset

Create a subclass that sets your customizations in the constructor. This is the cleanest approach when you use the same tuning in several places.

```csharp
using Aspose.LLM.Abstractions.Parameters.Presets;

public class LowTemperatureQwen : Qwen25Preset
{
    public LowTemperatureQwen()
    {
        ContextParameters.ContextSize = 16384;
        SamplerParameters.Temperature = 0.1f;
        ChatParameters.SystemPrompt = "You are a precise assistant. Answer briefly.";
    }
}

// Use it
using var api = AsposeLLMApi.Create(new LowTemperatureQwen());
```

## Create a preset from scratch

For a fully custom model, extend `PresetCoreBase` directly and populate the bags you need. At minimum, you must set `BaseModelSourceParameters` so the engine can find the model.

```csharp
using Aspose.LLM.Abstractions.Parameters.Presets;

public class MyCustomPreset : PresetCoreBase
{
    public MyCustomPreset()
    {
        BaseModelSourceParameters.HuggingFaceRepoId = "your-org/your-model-GGUF";
        BaseModelSourceParameters.HuggingFaceFileName = "your-model.Q4_K_M.gguf";

        ContextParameters.ContextSize = 4096;
        ContextParameters.NBatch = 1024;

        SamplerParameters.Temperature = 0.7f;
        SamplerParameters.TopP = 0.9f;

        BaseModelInferenceParameters.GpuLayers = 999;
    }
}
```

See the [custom preset](/net/use-cases/custom-preset/) use case for a full walkthrough including chat template selection for non-standard models.

## Default preset

`AsposeLLMApi.GetDefaultPreset()` returns a fresh `Qwen25Preset` instance. It is a convenience helper for code that needs a sensible preset when none has been chosen yet — not a snapshot of the engine's internal defaults.

```csharp
PresetCoreBase defaultPreset = api.GetDefaultPreset();
// Equivalent to: new Qwen25Preset()
```

For raw parameter values without a preset, call:

```csharp
(ModelInferenceParameters inference,
 ContextParameters context,
 ChatParameters chat,
 SamplerParameters sampler) = await api.GetDefaultParametersAsync();
```

## Supported presets

See [Supported presets](/net/product-overview/supported-presets/) for the full list of built-in text and vision presets with their Hugging Face model sources, context sizes, and quantization levels.

## What's next

- [Custom preset](/net/use-cases/custom-preset/) — extend or replace a built-in preset.
- [Chat sessions](/net/developer-reference/chat-sessions/) — how the engine uses preset parameters per session.
- [Supported presets](/net/product-overview/supported-presets/) — full catalog of built-in presets.

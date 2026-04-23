---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/presets/creating-from-scratch/
feedback: LLMNET
version: 26.5.0
title: Creating a preset from scratch
description: Extend PresetCoreBase to bring your own GGUF model into Aspose.LLM for .NET — text or vision, with full control over every parameter bag.
keywords:
- PresetCoreBase
- custom preset
- bring your own GGUF
- GGUF
- custom model
---

For models that do not have a built-in preset — your own fine-tune, a new open-weight release, a model from a less-common repository — extend `PresetCoreBase` and populate the parameter bags yourself.

## Minimum requirements

At minimum, set `BaseModelSourceParameters` so the engine can find the model. Every other bag has sensible fallbacks.

```csharp
using Aspose.LLM.Abstractions.Parameters.Presets;

public class MyCustomPreset : PresetCoreBase
{
    public MyCustomPreset()
    {
        BaseModelSourceParameters.HuggingFaceRepoId = "your-org/your-model-GGUF";
        BaseModelSourceParameters.HuggingFaceFileName = "your-model.Q4_K_M.gguf";
    }
}
```

`AsposeLLMApi.Create(new MyCustomPreset())` then downloads, loads, and runs the model with default context, sampler, and inference settings.

## Typical full preset

Include every knob you have tested so the behavior is predictable across environments.

```csharp
using Aspose.LLM.Abstractions.Parameters.Presets;

public class MyCustomPreset : PresetCoreBase
{
    public MyCustomPreset()
    {
        // --- Model source ---
        BaseModelSourceParameters.HuggingFaceRepoId = "your-org/your-model-GGUF";
        BaseModelSourceParameters.HuggingFaceFileName = "your-model.Q4_K_M.gguf";

        // --- Context ---
        ContextParameters.ContextSize = 32768;
        ContextParameters.NBatch = 2048;
        ContextParameters.NUbatch = 2048;
        ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;

        // --- Sampler ---
        SamplerParameters.Temperature = 0.7f;
        SamplerParameters.TopP = 0.9f;
        SamplerParameters.TopK = 40;
        SamplerParameters.MinP = 0.05f;
        SamplerParameters.RepetitionPenalty = 1.1f;

        // --- Inference / load ---
        BaseModelInferenceParameters.GpuLayers = 999;
        BaseModelInferenceParameters.UseMemoryMapping = true;

        // --- Chat ---
        ChatParameters.SystemPrompt = "You are a helpful assistant.";
        ChatParameters.MaxTokens = 2048;

        // --- Engine (optional) ---
        // EngineParameters.ModelCachePath = @"D:\models";
    }
}
```

## Vision preset from scratch

Vision models need both the base model and a multimodal projector. Populate `MmprojSourceParameters` too, and tune `MtmdContextParameters` as needed.

```csharp
public class MyVisionPreset : PresetCoreBase
{
    public MyVisionPreset()
    {
        // Base model
        BaseModelSourceParameters.HuggingFaceRepoId = "your-org/your-vl-model-GGUF";
        BaseModelSourceParameters.HuggingFaceFileName = "your-vl-model.Q4_K_M.gguf";

        // Vision projector (mmproj)
        MmprojSourceParameters.HuggingFaceRepoId = "your-org/your-vl-model-GGUF";
        MmprojSourceParameters.HuggingFaceFileName = "mmproj-your-vl-model-Q8_0.gguf";

        // Context
        ContextParameters.ContextSize = 32768;
        ContextParameters.NBatch = 4096;

        // Multimodal context
        MtmdContextParameters.UseGpu = true;

        // Sampler and chat defaults
        SamplerParameters.Temperature = 0.7f;
        ChatParameters.MaxTokens = 1024;

        BaseModelInferenceParameters.GpuLayers = 999;
    }
}
```

## Chat template considerations

The SDK picks the chat template from the model's GGUF metadata at load time. Template families it recognizes: LLaVA, Qwen2VL, Qwen3VL, Pixtral, InternVL, Gemma3, Llama4, MiniCPMV (vision); various text templates for the built-in text presets.

For a custom GGUF:

- **Model from one of the supported families** — template auto-detection usually works.
- **Non-standard template** — the current release does not expose a public override. You may see garbled output. Workarounds:
  - Pick a different GGUF export with richer metadata.
  - File a [support request](https://forum.aspose.com/) with the model's Hugging Face URL.

See [Chat templates](/net/developer-reference/multimodal/chat-templates/) for the vision side.

## Source priority

`ModelSourceParameters` has three fields priority-ordered:

1. `ModelFilePath` — local path, highest priority.
2. `AsposeModelId` — Aspose catalog, second.
3. `HuggingFaceRepoId` + `HuggingFaceFileName` — third.

For a local GGUF:

```csharp
BaseModelSourceParameters.ModelFilePath = @"C:\models\your-model.Q4_K_M.gguf";
// Leave HuggingFace fields unset.
```

## Verify the preset

Before deploying a new custom preset, verify:

- Model downloads successfully from Hugging Face (or the file exists locally).
- The first `Create` completes without errors.
- A simple prompt produces coherent output.
- Multimodal input (for vision presets) is processed correctly — see [Debugging vision](/net/developer-reference/multimodal/debugging-vision/).

Enable debug logging (`EngineParameters.EnableDebugLogging = true`) during initial testing to see template selection, projector load, and batch sizes.

## What's next

- [Using built-in presets](/net/developer-reference/presets/using-built-in/) — for models that already have a preset.
- [Customizing](/net/developer-reference/presets/customizing/) — tweak a built-in preset.
- [Parameters reference](/net/developer-reference/parameters/) — every knob available to a custom preset.
- [Custom preset use case](/net/use-cases/custom-preset/) — runnable end-to-end example.

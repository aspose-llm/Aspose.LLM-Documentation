---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/custom-preset/
feedback: LLMNET
version: 26.5.0
title: Custom preset
description: Customize a built-in Aspose.LLM for .NET preset or build one from scratch — three patterns for bringing your own parameters or your own GGUF model.
keywords:
- custom
- preset
- PresetCoreBase
- override
- bring your own GGUF
- subclass
- parameters
---

Presets bundle the settings needed to run a model. When a built-in preset does not quite fit — different sampler, different context size, a specific system prompt, or a completely different model — you customize it. This page shows three patterns, from simplest to most involved.

## When to customize

- Change a parameter for a single run (temperature, context size, GPU layers).
- Apply the same tuning across your codebase (subclass).
- Run a model that does not have a built-in preset (GGUF file from Hugging Face or a local path).

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).
- Familiarity with [Presets](/net/developer-reference/presets/) and [Supported presets](/net/product-overview/supported-presets/).

## Pattern 1. Override before `Create`

Instantiate a built-in preset and tweak fields on its parameter bags before passing it to `AsposeLLMApi.Create`. Quick and local.

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();

// Smaller context to save memory
preset.ContextParameters.ContextSize = 8192;

// More deterministic sampling
preset.SamplerParameters.Temperature = 0.2f;
preset.SamplerParameters.TopP = 0.9f;

// Force CPU-only inference
preset.BaseModelInferenceParameters.GpuLayers = 0;

// Default system prompt for every session created through this instance
preset.ChatParameters.SystemPrompt = "You are a precise assistant. Answer briefly.";

using var api = AsposeLLMApi.Create(preset);

string reply = await api.SendMessageAsync("Summarize in one sentence: the theory of relativity.");
Console.WriteLine(reply);
```

The engine reads the preset during `Create`. Mutations after `Create` do not affect the alive engine — override everything you need first.

## Pattern 2. Subclass a built-in preset

When you use the same tuning in several places, create a subclass. This is idiomatic for team-wide configuration.

```csharp
using Aspose.LLM.Abstractions.Parameters.Presets;

public class LowTemperatureQwen : Qwen25Preset
{
    public LowTemperatureQwen()
    {
        ContextParameters.ContextSize = 16384;

        SamplerParameters.Temperature = 0.1f;
        SamplerParameters.TopP = 0.95f;
        SamplerParameters.RepetitionPenalty = 1.1f;

        ChatParameters.SystemPrompt =
            "You are a helpful technical assistant. Cite sources when you can, and say when you do not know.";
        ChatParameters.MaxTokens = 1024;

        BaseModelInferenceParameters.GpuLayers = 999; // full GPU offload if available
    }
}

// Use it anywhere:
using var api = AsposeLLMApi.Create(new LowTemperatureQwen());
```

Subclasses inherit the base preset's model source and chat template, so you keep the curated defaults of `Qwen25Preset` and change only what you need.

## Pattern 3. Build a preset from scratch

Extend `PresetCoreBase` directly when you want a model that does not have a built-in preset. This is the "bring your own GGUF" path.

At minimum, set `BaseModelSourceParameters` so the engine knows where the model lives.

```csharp
using Aspose.LLM.Abstractions.Parameters.Presets;

public class MyCustomPreset : PresetCoreBase
{
    public MyCustomPreset()
    {
        // Source: Hugging Face repository and file name
        BaseModelSourceParameters.HuggingFaceRepoId = "your-org/your-model-GGUF";
        BaseModelSourceParameters.HuggingFaceFileName = "your-model.Q4_K_M.gguf";

        // Or a local file path:
        // BaseModelSourceParameters.ModelFilePath = @"C:\models\your-model.Q4_K_M.gguf";

        // Context and batch
        ContextParameters.ContextSize = 16384;
        ContextParameters.NBatch = 2048;

        // Sampling
        SamplerParameters.Temperature = 0.7f;
        SamplerParameters.TopP = 0.9f;
        SamplerParameters.TopK = 40;
        SamplerParameters.RepetitionPenalty = 1.05f;

        // Inference
        BaseModelInferenceParameters.GpuLayers = 999;
        BaseModelInferenceParameters.UseMemoryMapping = true;

        // Chat defaults
        ChatParameters.MaxTokens = 2048;
        ChatParameters.SystemPrompt = "You are a helpful assistant.";
    }
}
```

Use the model-file priority order: if `ModelFilePath` is set, it wins over `AsposeModelId`, which wins over `HuggingFaceRepoId` + `HuggingFaceFileName`.

### Vision preset from scratch

For a model with vision input, also set `MmprojSourceParameters` and tune `MtmdContextParameters`:

```csharp
public class MyVisionPreset : PresetCoreBase
{
    public MyVisionPreset()
    {
        BaseModelSourceParameters.HuggingFaceRepoId = "your-org/your-vl-model-GGUF";
        BaseModelSourceParameters.HuggingFaceFileName = "your-vl-model.Q4_K_M.gguf";

        MmprojSourceParameters.HuggingFaceRepoId = "your-org/your-vl-model-GGUF";
        MmprojSourceParameters.HuggingFaceFileName = "mmproj-your-vl-model-Q8_0.gguf";

        ContextParameters.ContextSize = 32768;
        ContextParameters.NBatch = 4096;

        MtmdContextParameters.UseGpu = true;
        MtmdContextParameters.Verbosity = 1;

        ChatParameters.MaxTokens = 1024;
    }
}
```

Pass image bytes to `SendMessageAsync` or `SendMessageToSessionAsync` via the `media` parameter the same way you would with a built-in vision preset.

## Chat template considerations

Aspose.LLM selects the chat template from the model's metadata when it loads the GGUF. For a model built against a widely-used template (ChatML, Llama 3, Qwen, Gemma, Phi, DeepSeek, Mistral), template detection works automatically.

For a model with a non-standard template, generations may be garbled — the wrong special tokens end up in the prompt. The current release does not expose a public chat-template override, so your options are:

- Pick a GGUF export that uses a standard template.
- Add a system prompt that steers the model toward a known format.
- File a support request via the [Aspose Support Forum](https://forum.aspose.com/) for advice on your specific model.

## Full runnable example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

public class ConciseAssistantPreset : Qwen25Preset
{
    public ConciseAssistantPreset()
    {
        ContextParameters.ContextSize = 8192;

        SamplerParameters.Temperature = 0.3f;
        SamplerParameters.TopP = 0.9f;

        ChatParameters.SystemPrompt =
            "You are a concise assistant. Every answer is at most two short sentences.";
        ChatParameters.MaxTokens = 200;

        BaseModelInferenceParameters.GpuLayers = 999;
    }
}

internal class CustomPresetDemo
{
    public static async Task Main()
    {
        var license = new Aspose.LLM.License();
        license.SetLicense("Aspose.LLM.lic");

        using var api = AsposeLLMApi.Create(new ConciseAssistantPreset());

        string[] questions =
        {
            "What is the capital of Portugal?",
            "What language is spoken there?",
            "Name one famous dish.",
        };

        foreach (string q in questions)
        {
            Console.WriteLine($"> {q}");
            string reply = await api.SendMessageAsync(q);
            Console.WriteLine(reply);
            Console.WriteLine();
        }
    }
}
```

## What's next

- [Presets reference](/net/developer-reference/presets/) — every bag on `PresetCoreBase` and its role.
- [Supported presets](/net/product-overview/supported-presets/) — built-in presets you can start from.
- [Multi-turn chat](/net/use-cases/multi-turn-chat/) — use your custom preset across many sessions.

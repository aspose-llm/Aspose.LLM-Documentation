---
weight: 120
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/bring-your-own-gguf/
feedback: LLMNET
version: 26.5.0
title: Bring your own GGUF
description: Run a custom GGUF model in Aspose.LLM for .NET — pick the file, write a preset, verify compatibility, tune parameters.
keywords:
- custom GGUF
- bring your own model
- PresetCoreBase
- Hugging Face
- GGUF
- fine-tune
---

When a built-in preset does not cover your model — your organization's fine-tune, a newly-released open-weight model, a community variant — extend `PresetCoreBase` and point it at the GGUF file. This use case walks through the full workflow from download to first response.

## When to use this pattern

- You have fine-tuned a base model and exported it as GGUF.
- You want to run a newly released open-weight model that does not yet have a built-in preset.
- You use a quantization variant (`Q6_K`, `IQ4_XS`) not covered by the default preset.
- You want to standardize on a specific model version across your team.

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).
- A GGUF file — either a local path or a Hugging Face repo identifier.

## Step 1. Identify the model

Pick a source:

- **Hugging Face**: find the repo and file name. Example: `bartowski/Qwen2.5-7B-Instruct-GGUF` / `Qwen2.5-7B-Instruct-Q6_K.gguf`.
- **Local file**: absolute path to a `.gguf` on disk.

Confirm the model's base family — that determines which chat template the engine auto-selects. Check the Hugging Face README for "derived from" or the GGUF metadata. Supported families: LLaVA, Qwen2/2.5/3, Pixtral (Mistral), InternVL, Gemma 3, Llama 4, MiniCPM-V, plus the text families of built-in presets.

## Step 2. Write a preset

The minimum viable preset sets `BaseModelSourceParameters`:

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

For a local file:

```csharp
public class MyLocalPreset : PresetCoreBase
{
    public MyLocalPreset()
    {
        BaseModelSourceParameters.ModelFilePath =
            @"C:\models\custom-finetune.Q4_K_M.gguf";
    }
}
```

## Step 3. Tune parameters

The default `PresetCoreBase` starts with conservative values. Override what matters:

```csharp
public class MyFullPreset : PresetCoreBase
{
    public MyFullPreset()
    {
        BaseModelSourceParameters.HuggingFaceRepoId = "your-org/your-model-GGUF";
        BaseModelSourceParameters.HuggingFaceFileName = "your-model.Q4_K_M.gguf";

        // Context — match or exceed your typical prompt length.
        ContextParameters.ContextSize = 16384;
        ContextParameters.NBatch = 2048;

        // Sampling — tune for your use case.
        SamplerParameters.Temperature = 0.7f;
        SamplerParameters.TopP = 0.9f;
        SamplerParameters.TopK = 40;
        SamplerParameters.MinP = 0.05f;
        SamplerParameters.RepetitionPenalty = 1.1f;

        // Inference / load.
        BaseModelInferenceParameters.GpuLayers = 999;
        BaseModelInferenceParameters.UseMemoryMapping = true;

        // Chat defaults.
        ChatParameters.SystemPrompt = "You are a helpful assistant.";
        ChatParameters.MaxTokens = 2048;
    }
}
```

Start with one of the built-in presets for your family as a reference — copy its values to your custom preset and adjust.

## Step 4. Verify with debug logging

First run: turn on debug logging and watch the `[MM]`, `[CTX]`, `[KV]` tags for anything unexpected.

```csharp
using Microsoft.Extensions.Logging;

using var loggerFactory = LoggerFactory.Create(b => b.AddConsole().SetMinimumLevel(LogLevel.Debug));
var logger = loggerFactory.CreateLogger<Program>();

var preset = new MyFullPreset();
preset.EngineParameters.EnableDebugLogging = true;

using var api = AsposeLLMApi.Create(preset, logger);
string reply = await api.SendMessageAsync("Say hello in one short sentence.");
Console.WriteLine(reply);
```

Expected log highlights:

- Model downloaded / loaded from the specified source.
- Chat template selected matches your model family.
- First token produced within a reasonable time.

If the reply contains literal `<image>` or other template markers — the engine fell back to a generic template. Check the model's GGUF metadata; see [Chat templates](/net/developer-reference/multimodal/chat-templates/).

## Step 5. Validate quality

Run representative prompts:

```csharp
string[] probes =
{
    "Summarize this text: [sample].",
    "List three steps to achieve [goal].",
    "Translate this sentence to French: 'The library is open on weekends.'",
};

foreach (string probe in probes)
{
    string reply = await api.SendMessageAsync(probe);
    Console.WriteLine($"Q: {probe}\nA: {reply}\n");
}
```

Compare with the same prompts on the nearest built-in preset, or with manual queries against the model's official reference (e.g., its Hugging Face demo). If output is noticeably worse, re-check:

- Chat template selection (check debug logs).
- Sampler settings (try lower temperature first).
- System prompt (some models need a specific system-turn format).

## Vision variant

Vision models need both the base model and a projector. Set `MmprojSourceParameters` too:

```csharp
public class MyCustomVisionPreset : PresetCoreBase
{
    public MyCustomVisionPreset()
    {
        BaseModelSourceParameters.HuggingFaceRepoId = "your-org/your-vl-model-GGUF";
        BaseModelSourceParameters.HuggingFaceFileName = "your-vl-model.Q4_K_M.gguf";

        MmprojSourceParameters.HuggingFaceRepoId = "your-org/your-vl-model-GGUF";
        MmprojSourceParameters.HuggingFaceFileName = "mmproj-your-vl-model-Q8_0.gguf";

        ContextParameters.ContextSize = 32768;
        BaseModelInferenceParameters.GpuLayers = 999;
    }
}
```

See [Vision presets](/net/developer-reference/multimodal/vision-presets/) for the chat-template caveats.

## Step 6. Share the preset

Commit the preset class to your source control. Team members get the same tuning by instantiating the class — no duplication of HF IDs, context sizes, or sampler knobs across code paths.

## Common errors

| Symptom | Likely cause | Fix |
|---|---|---|
| `InvalidOperationException` at Create | Model file not found in cache and download failed. | Verify Hugging Face repo and file names; check network. |
| Garbled output | Chat template mismatch. | Check debug logs; try a different GGUF export of the same model. |
| Slow responses | Wrong acceleration or `GpuLayers = 0`. | Set `PreferredAcceleration` and `GpuLayers` per your GPU. |
| Repetitive output | Default sampler too aggressive on repetition. | Lower `RepetitionPenalty` to ~1.05. |
| Truncated responses (reasoning model) | `MaxTokens` too low. | Raise to 1024-2048 for chain-of-thought models. |

## What's next

- [Creating a preset from scratch](/net/developer-reference/presets/creating-from-scratch/) — presets reference.
- [Custom preset use case](/net/use-cases/custom-preset/) — simpler override patterns.
- [Chat templates](/net/developer-reference/multimodal/chat-templates/) — how detection works.

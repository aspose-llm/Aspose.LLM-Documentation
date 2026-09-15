---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/presets/using-built-in/
feedback: LLMNET
version: 26.5.0
title: Using built-in presets
description: Pick and use a built-in Aspose.LLM for .NET preset for your scenario, text vs vision, size vs speed, and task-based selection.
keywords:
- preset
- built-in
- Qwen
- Gemma
- Llama
- Phi
- DeepSeek
- picker
---

The built-in presets ship tuned defaults for every popular open-weight model family. Pick one, pass it to `AsposeLLMApi.Create`, and the engine handles model download, binary deployment, sampler tuning, and chat template selection.

{{% alert color="warning" %}}
**No built-in LLM is included in Aspose.LLM.** The library is a local inference runtime: you choose which open source model to use, and the model file is obtained separately and stored on your own machine. Model files are covered by the license of the model publisher, not by your license agreement with Aspose Pty Ltd.

Check the license of the model a preset resolves to before using it commercially. Licenses are listed in [Supported presets](/llm/net/product-overview/supported-presets/).
{{% /alert %}}

This page helps you pick the right preset for your scenario and highlights the minimal code to use it. For full catalog details, see [Supported presets](/llm/net/product-overview/supported-presets/).

## Minimal usage

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(preset);

string reply = await api.SendMessageAsync("Hello!");
```

Every preset follows the same pattern: swap the class name to change the model.

## Picker by task

### Text chat

| Goal | Preset | Notes |
|---|---|---|
| Balanced general assistant | `Qwen25Preset`, `Qwen3Preset`, `Llm31_8BPreset`, or `Mistral7Preset` | 7-8B, good at most tasks. |
| Smallest footprint | `Llm32Preset` (3B) or `Phi4Preset` (mini) | Run on modest hardware. |
| Smallest possible (CPU-only) | `SmallModelPreset` (0.5B), `TinyLlmPreset` (1.1B), or `Llm32_1BPreset` (1B) | Tutorials, smoke tests, edge boxes. |
| Very long context | `Llm32Preset` (131K) or `DeepSeekCoder2Preset` (163K) | For long documents. |
| Coding | `DeepSeekCoder2Preset`, `Qwen25Coder7BPreset`, `Qwen3Coder30BPreset`, or `DevstralSmall2_24BPreset` | Specialized training on code. |
| Multilingual coverage | `Qwen3Preset` or `MistralSmall3Preset` | Trained on broad language mixes. |
| Enterprise-tuned | `Granite3_8BPreset` | IBM Granite 3.1, safety-aligned. |
| Fully-open research | `Olmo2_7BPreset` | AllenAI OLMo 2, fully open training and data. |
| Frontier on a workstation | `Llm3_3_70BPreset`, `SeedOss36BPreset`, or `MistralSmall3Preset` | 24-70B, needs 32 GB+ RAM and partial GPU offload. |
| Step-by-step reasoning | `DeepseekR1Qwen3Preset`, `Ernie4_5_21BPreset`, or `SeedOss36BPreset` | Chain-of-thought style output. Budget 1024-2048 MaxTokens. |
| No GPU available | The `*PresetCpu` twin of any preset: `Mistral7PresetCpu`, `Llm31_8BPresetCpu`, `Phi35MiniPresetCpu`, … | 27 twins ship. `GpuLayers = 0`, context capped at 4 K. See [CPU-tuned variants](/llm/net/product-overview/supported-presets/#cpu-tuned-variants-presetcpu). |
| No built-in preset: use your own GGUF | Extend `PresetCoreBase` | See [Creating from scratch](/llm/net/developer-reference/presets/creating-from-scratch/). |

### Vision

| Goal | Preset | Notes |
|---|---|---|
| Smallest vision model with long context | `Qwen3VL2BPreset` | 2B, 262K context. |
| Strongest reasoning on images | `Ministral3VisionPreset` | 8B, 262K. |
| Vision + chain-of-thought reasoning | `NemotronOmniPreset` | 30B MoE, multimodal + reasoning; needs 32 GB+ RAM. |
| Lightweight GLM-family vision | `Glm4_6VFlashPreset` | Zhipu GLM-4.6V Flash. |

See [Vision presets](/llm/net/developer-reference/multimodal/vision-presets/) for details and memory requirements.

## Trade-offs

| Dimension | Smaller preset | Larger preset |
|---|---|---|
| Speed | Faster (more tokens/sec) | Slower |
| Memory | Less RAM/VRAM | More |
| Quality | Lower on complex tasks | Higher |
| Cost (machine time) | Lower | Higher |

Start with the smallest preset that meets your quality bar. Move up only if the output is not good enough.

## Common overrides on built-in presets

Tweak defaults before `Create` without changing the preset class:

```csharp
var preset = new Qwen25Preset();

// Make output more deterministic.
preset.SamplerParameters.Temperature = 0.2f;

// Use a smaller context to save memory.
preset.ContextParameters.ContextSize = 8192;

// Set a default system prompt.
preset.ChatParameters.SystemPrompt = "You are a concise assistant.";

using var api = AsposeLLMApi.Create(preset);
```

See [Customizing](/llm/net/developer-reference/presets/customizing/) for the full pattern and common knobs.

## Before you ship

{{% alert color="warning" %}}
**Check the license of the model you selected.** Aspose.LLM supplies the runtime, not the model. Whichever model you load, its terms come from the party that published it and they apply to your product. They are not part of, and are not covered by, your license agreement with Aspose Pty Ltd. Some open source models allow commercial use with no strings attached, others attach conditions such as attribution or an acceptable use policy, and a few exclude commercial use or withdraw it above a revenue threshold. [Supported LLMs](/llm/net/product-overview/supported-llms/) lists the license of every family the SDK ships a preset for.
{{% /alert %}}

## What's next

- [Customizing](/llm/net/developer-reference/presets/customizing/): override fields on built-in presets.
- [Creating from scratch](/llm/net/developer-reference/presets/creating-from-scratch/): extend `PresetCoreBase` for a custom model.
- [Supported presets](/llm/net/product-overview/supported-presets/): full catalog with Hugging Face sources.

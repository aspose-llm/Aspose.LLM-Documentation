---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/presets/using-built-in/
feedback: LLMNET
version: 26.5.0
title: Using built-in presets
description: Pick and use a built-in Aspose.LLM for .NET preset for your scenario — text vs vision, size vs speed, and task-based selection.
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

This page helps you pick the right preset for your scenario and highlights the minimal code to use it. For full catalog details, see [Supported presets](/net/product-overview/supported-presets/).

## Minimal usage

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(preset);

string reply = await api.SendMessageAsync("Hello!");
```

Every preset follows the same pattern — swap the class name to change the model.

## Picker by task

### Text chat

| Goal | Preset | Notes |
|---|---|---|
| Balanced general assistant | `Qwen25Preset` or `Qwen3Preset` | 7-8B, good at most tasks. |
| Smallest footprint | `Llama32Preset` (3B) or `Phi4Preset` (mini) | Run on modest hardware. |
| Very long context | `Llama32Preset` (131K), `Oss20Preset` (131K), `DeepSeekCoder2Preset` (163K) | For long documents. |
| Coding | `DeepSeekCoder2Preset` | Specialized training on code. |
| Step-by-step reasoning | `DeepseekR1Qwen3Preset` or `Oss20Preset` (multilingual-reasoner) | Chain-of-thought style output. Budget 1024-2048 MaxTokens. |
| No built-in preset — use your own GGUF | Extend `PresetCoreBase` | See [Creating from scratch](/net/developer-reference/presets/creating-from-scratch/). |

### Vision

| Goal | Preset | Notes |
|---|---|---|
| Smallest vision model with long context | `Qwen3VL2BPreset` | 2B, 262K context. |
| General-purpose vision Q&A | `Qwen25VL3BPreset` | 3B, 128K. |
| Document / text-heavy images | `Gemma3VisionPreset` | Fine-tuned for Latex and structured text. |
| Strongest reasoning on images | `Ministral3VisionPreset` | 8B, 262K. |

See [Vision presets](/net/developer-reference/multimodal/vision-presets/) for details and memory requirements.

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

See [Customizing](/net/developer-reference/presets/customizing/) for the full pattern and common knobs.

## What's next

- [Customizing](/net/developer-reference/presets/customizing/) — override fields on built-in presets.
- [Creating from scratch](/net/developer-reference/presets/creating-from-scratch/) — extend `PresetCoreBase` for a custom model.
- [Supported presets](/net/product-overview/supported-presets/) — full catalog with Hugging Face sources.

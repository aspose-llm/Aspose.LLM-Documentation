---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/multimodal/vision-presets/
feedback: LLMNET
version: 26.5.0
title: Vision presets
description: Built-in vision presets in Aspose.LLM for .NET — Qwen VL, Gemma 3 Vision, Ministral 3 Vision, with model sources, projector files, and picker guidance.
keywords:
- vision preset
- Qwen VL
- Gemma 3 Vision
- Ministral 3
- mmproj
- vision projector
- VLM
---

The SDK ships four built-in vision presets. Each preset configures both the base language model and its multimodal projector (`mmproj`) — the two files load together on first `Create`.

## Available presets

| Preset | Model | Base context | Quantization | mmproj file |
|---|---|---:|---|---|
| `Qwen25VL3BPreset` | Qwen 2.5 VL 3B Instruct | 128 000 | UD-IQ2_XXS | `mmproj-F16.gguf` |
| `Qwen3VL2BPreset` | Qwen 3 VL 2B Instruct | 262 144 | Q4_K_M | `mmproj-Qwen3VL-2B-Instruct-Q8_0.gguf` |
| `Gemma3VisionPreset` | Gemma 3 Vision (Latex fine-tune) | 8 096 | Q4_K_M | `Gemma-3-Vision-Latex.mmproj-f16.gguf` |
| `Ministral3VisionPreset` | Ministral 3 8B Instruct (Mistral AI, 2512 release) | 262 144 | Q4_K_M | `Ministral-3-8B-Instruct-2512-BF16-mmproj.gguf` |

See [Supported presets](/net/product-overview/supported-presets/#vision-presets) for the Hugging Face source repositories.

## Picker

| Need | Try |
|---|---|
| Smallest footprint, long context | `Qwen3VL2BPreset` (2B parameters, 262K context) |
| General-purpose vision Q&A | `Qwen25VL3BPreset` (3B, 128K) |
| Text-heavy images (documents, LaTeX) | `Gemma3VisionPreset` |
| Strongest reasoning on complex images | `Ministral3VisionPreset` (8B, 262K) |

All four produce reasonable image descriptions and simple spatial reasoning. For OCR-style tasks on dense text, lean toward Gemma 3 Vision or Ministral 3 — the larger projectors handle small text better.

## Memory

Vision presets load two files: the base model and the projector. Add the projector memory footprint on top of the base model — typically 200 MB to 2 GB depending on precision.

| Preset | Base (VRAM/RAM) | Projector | Total |
|---|---|---|---|
| `Qwen25VL3BPreset` (UD-IQ2_XXS) | ~2 GB | ~0.8 GB (F16) | ~3 GB |
| `Qwen3VL2BPreset` | ~2 GB | ~0.5 GB (Q8_0) | ~2.5 GB |
| `Gemma3VisionPreset` | ~3 GB | ~1 GB (F16) | ~4 GB |
| `Ministral3VisionPreset` | ~6 GB | ~2 GB (BF16) | ~8 GB |

Add KV cache on top (scales with `ContextParameters.ContextSize`). For long contexts, reduce `TypeV` to `Q8_0` to claw back memory.

## Using a vision preset

Same pattern as any other preset. Pass images via the `media` parameter of `SendMessageAsync` or `SendMessageToSessionAsync`:

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen3VL2BPreset();
using var api = AsposeLLMApi.Create(preset);

byte[] imageBytes = File.ReadAllBytes("document.png");

string reply = await api.SendMessageAsync(
    "Transcribe the text in this image verbatim.",
    media: new[] { imageBytes });

Console.WriteLine(reply);
```

See [Attaching images](/net/developer-reference/multimodal/attaching-images/) for format and size rules.

## Customizing a vision preset

The same override patterns as text presets apply. See [Presets](/net/developer-reference/presets/) for the three approaches:

- **Override before `Create`** — tweak fields on the preset instance.
- **Subclass** — inherit from a built-in vision preset and set defaults in the constructor.
- **From scratch** — extend `PresetCoreBase` and populate both `BaseModelSourceParameters` and `MmprojSourceParameters`.

Additional vision-only knobs live on [`MtmdContextParameters`](/net/developer-reference/parameters/multimodal-context/) — control projector GPU offload, threading, and verbosity.

## What's next

- [Attaching images](/net/developer-reference/multimodal/attaching-images/) — how to pass image bytes.
- [Chat templates](/net/developer-reference/multimodal/chat-templates/) — how the SDK selects the right template per model.
- [Multimodal context parameters](/net/developer-reference/parameters/multimodal-context/) — vision-side tuning knobs.
- [Model source parameters](/net/developer-reference/parameters/model-source/) — configure the `mmproj` download source.

---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/multimodal/vision-presets/
feedback: LLMNET
version: 26.5.0
title: Vision presets
description: Built-in vision presets in Aspose.LLM for .NET, Qwen VL, Gemma 3 Vision, Ministral 3 Vision, with model sources, projector files, and picker guidance.
keywords:
- vision preset
- Qwen VL
- Gemma 3 Vision
- Ministral 3
- mmproj
- vision projector
- VLM
---

The SDK ships four built-in vision presets. Each preset configures both the base language model and its multimodal projector (`mmproj`): the two files load together on first `Create`.

{{% alert color="warning" %}}
**No built-in LLM is included in Aspose.LLM.** The library is a local inference runtime: you choose which open source model to use, and the model file is obtained separately and stored on your own machine. Model files are covered by the license of the model publisher, not by your license agreement with Aspose Pty Ltd.

Check the license of the model a preset resolves to before using it commercially. Licenses are listed in [Supported presets](/llm/net/product-overview/supported-presets/).
{{% /alert %}}

## Available presets

| Preset | Model | Base context | Quantization | mmproj file |
|---|---|---:|---|---|
| `Qwen25VL3BPreset` | Qwen 2.5 VL 3B Instruct | 128 000 | UD-IQ2_XXS | `mmproj-F16.gguf` |
| `Qwen3VL2BPreset` | Qwen 3 VL 2B Instruct | 262 144 | Q4_K_M | `mmproj-Qwen3VL-2B-Instruct-Q8_0.gguf` |
| `Gemma3VisionPreset` | Gemma 3 Vision (Latex fine-tune) | 8 096 | Q4_K_M | `Gemma-3-Vision-Latex.mmproj-f16.gguf` |
| `Ministral3VisionPreset` | Ministral 3 8B Instruct (Mistral AI, 2512 release) | 262 144 | Q4_K_M | `Ministral-3-8B-Instruct-2512-BF16-mmproj.gguf` |

See [Supported presets](/llm/net/product-overview/supported-presets/#vision-presets) for the Hugging Face source repositories.

## Picker

| Need | Try |
|---|---|
| Smallest footprint, long context | `Qwen3VL2BPreset` (2B parameters, 262K context) |
| General-purpose vision Q&A | `Glm4_6VFlashPreset` |
| Text-heavy images (documents, LaTeX) | `Ministral3VisionPreset` (8B, 262K) |
| Strongest reasoning on complex images | `Ministral3VisionPreset` (8B, 262K) |

All four produce reasonable image descriptions and simple spatial reasoning. For OCR-style tasks on dense text, lean toward Gemma 3 Vision or Ministral 3: the larger projectors handle small text better.

## Memory

Vision presets load two files: the base model and the projector. Add the projector memory footprint on top of the base model: typically 200 MB to 2 GB depending on precision.

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

See [Attaching images](/llm/net/developer-reference/multimodal/attaching-images/) for format and size rules.

## Customizing a vision preset

The same override patterns as text presets apply. See [Presets](/llm/net/developer-reference/presets/) for the three approaches:

- **Override before `Create`**: tweak fields on the preset instance.
- **Subclass**: inherit from a built-in vision preset and set defaults in the constructor.
- **From scratch**: extend `PresetCoreBase` and populate both `BaseModelSourceParameters` and `MmprojSourceParameters`.

Additional vision-only knobs live on [`MtmdContextParameters`](/llm/net/developer-reference/parameters/multimodal-context/): control projector GPU offload, threading, and verbosity.

## Before you ship

{{% alert color="warning" %}}
**Check the license of the model you selected.** Aspose.LLM supplies the runtime, not the model. Whichever model you load, its terms come from the party that published it and they apply to your product. They are not part of, and are not covered by, your license agreement with Aspose Pty Ltd. Some open source models allow commercial use with no strings attached, others attach conditions such as attribution or an acceptable use policy, and a few exclude commercial use or withdraw it above a revenue threshold. [Supported LLMs](/llm/net/product-overview/supported-llms/) lists the license of every family the SDK ships a preset for.
{{% /alert %}}

## What's next

- [Attaching images](/llm/net/developer-reference/multimodal/attaching-images/): how to pass image bytes.
- [Chat templates](/llm/net/developer-reference/multimodal/chat-templates/): how the SDK selects the right template per model.
- [Multimodal context parameters](/llm/net/developer-reference/parameters/multimodal-context/): vision-side tuning knobs.
- [Model source parameters](/llm/net/developer-reference/parameters/model-source/): configure the `mmproj` download source.

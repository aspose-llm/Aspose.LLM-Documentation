---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/multimodal/chat-templates/
feedback: LLMNET
version: 26.5.0
title: Chat templates
description: Eight vision chat templates Aspose.LLM for .NET recognizes and selects automatically — LLaVA, Qwen2VL, Qwen3VL, Pixtral, InternVL, Gemma3, Llama4, MiniCPMV.
keywords:
- chat template
- vision template
- LLaVA
- Qwen2VL
- Qwen3VL
- Pixtral
- InternVL
- Gemma3
- Llama4
- MiniCPMV
---

Every vision model family has its own prompt format — a specific way to mark where images are inserted and how text turns are wrapped. Aspose.LLM for .NET ships eight templates and selects the right one automatically from the model's GGUF metadata at load time.

You do **not** configure templates yourself in the current release. The selection is automatic and internal. This page exists so you can identify which templates are supported and recognize the model families behind them.

## Supported templates

| Template | Model family | Typical models |
|---|---|---|
| **LLaVA** | LLaVA-style vision fine-tunes | LLaVA-1.5, LLaVA-1.6, various community LLaVAs |
| **Qwen2VL** | Qwen 2 / 2.5 VL | `Qwen2-VL-*`, `Qwen2.5-VL-*` |
| **Qwen3VL** | Qwen 3 VL | `Qwen3-VL-*` |
| **Pixtral** | Mistral Pixtral | `Pixtral-12B` and derivatives |
| **InternVL** | InternVL series | `InternVL2-*`, `InternVL3-*` |
| **Gemma3** | Gemma 3 Vision | `gemma-3-*-vision` variants |
| **Llama4** | Llama 4 Vision | `Llama-4-*-Vision` |
| **MiniCPMV** | MiniCPM-V | `MiniCPM-V-*` |

## How selection works

When the engine loads a vision model, it:

1. Reads the model's metadata (architecture name, base model name, and vision-specific keys) from the GGUF file.
2. Matches those values against the template dispatch table in `Aspose.LLM.Interop.Multimodal.VisualModelChatTemplates`.
3. Picks the template whose marker tokens and turn format match the detected model family.

If no template matches, the engine falls back to the default text template. Vision turns then emit a generic marker that the model may or may not recognize — output quality degrades.

## Built-in preset → template mapping

Each built-in vision preset is already tested against its matching template:

| Preset | Template auto-selected |
|---|---|
| `Qwen25VL3BPreset` | Qwen2VL |
| `Qwen3VL2BPreset` | Qwen3VL |
| `Gemma3VisionPreset` | Gemma3 |
| `Ministral3VisionPreset` | Pixtral (Mistral vision models share the Pixtral template) |

If you extend one of these presets or use a custom GGUF from the same family, template detection still works.

## Custom vision models

If you build a custom preset pointing at a non-Aspose GGUF from one of the eight supported families, template auto-selection should work out of the box — the engine relies on metadata keys that most upstream GGUF conversions preserve.

If selection fails — the response is garbled or includes literal marker tokens like `<image>` in the output — the model's metadata is missing the expected keys. Options:

- Pick a different GGUF export of the same model with richer metadata.
- File a support request via the [Aspose Support Forum](https://forum.aspose.com/) with the model's Hugging Face URL so the team can add detection for that specific export.

The current SDK version does not expose a public override for forcing a specific template. A future release may add that surface.

## Checking which template was picked

Enable debug logging and look for the `[MM]` tags:

```csharp
preset.EngineParameters.EnableDebugLogging = true;
```

Lines like `[MM] selected template: Qwen3VL` appear shortly after model load. See [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) for the full log taxonomy.

## What's next

- [Vision presets](/net/developer-reference/multimodal/vision-presets/) — built-in presets with their matching templates.
- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — inspect the selected template in logs.
- [Attaching images](/net/developer-reference/multimodal/attaching-images/) — the sending side.

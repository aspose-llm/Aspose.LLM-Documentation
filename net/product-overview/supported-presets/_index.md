---
weight: 20
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/product-overview/supported-presets/
feedback: LLMNET
title: Supported presets
description: Built-in presets for Aspose.LLM for .NET.
keywords:
- preset
- model
- Qwen
- Gemma
- Llama
- Phi
---

Aspose.LLM for .NET uses **presets** to configure the model, context, chat, and sampler parameters. You pass a preset to `AsposeLLMApi.Create(preset)`. The base type is `PresetCoreBase` (from the `Aspose.LLM.Abstractions` package).

## Text presets

Built-in presets for text-only models include:

- **Qwen25Preset** — Qwen 2.5 7B Instruct (GGUF)
- **Qwen3Preset** — Qwen 3
- **Gemma3Preset** — Gemma 3
- **Llama32Preset** — Llama 3.2
- **Phi4Preset** — Phi 4
- **DeepseekR1Qwen3Preset** — DeepSeek R1 / Qwen 3
- **DeepSeekCoder2Preset** — DeepSeek Coder 2
- **Oss20Preset** — OSS 2.0

## Vision presets

For models that support images (multimodal input), use a vision preset:

- **Qwen25VL3BPreset** — Qwen 2.5 VL (vision-language) 3B
- **Qwen3VL2BPreset** — Qwen 3 VL 2B
- **Gemma3VisionPreset** — Gemma 3 Vision
- **Ministral3VisionPreset** — Ministral 3 Vision

Use the optional `media` parameter of `SendMessageAsync` or `SendMessageToSessionAsync` to pass image (or other) bytes when using a vision preset.

## Default preset

`GetDefaultPreset()` returns a preset with engine default parameters (e.g. `Qwen25Preset`). You can also obtain raw default parameters with `GetDefaultParametersAsync()`.

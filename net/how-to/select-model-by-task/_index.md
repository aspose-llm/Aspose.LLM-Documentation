---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/how-to/select-model-by-task/
feedback: LLMNET
version: 26.5.0
title: Select a model by task
description: Pick an Aspose.LLM for .NET built-in preset that fits your task — general chat, reasoning, code, vision, long context.
keywords:
- model selection
- preset selection
- picker
- task
---

Match a built-in preset to the task. Start small; move up only if output quality does not meet your bar.

## Quick picker

| Your task | Preset |
|---|---|
| General chat, mid-complexity tasks | `Qwen25Preset` (7B) |
| Latest general-purpose model | `Qwen3Preset` (8B) |
| Small footprint, fast, long context | `Llama32Preset` (3B, 131K) |
| Smallest possible model | `Phi4Preset` (mini) |
| Coding tasks | `DeepSeekCoder2Preset` |
| Step-by-step reasoning | `DeepseekR1Qwen3Preset` or `Oss20Preset` |
| Largest model, strongest reasoning | `Oss20Preset` (20B) |
| Image understanding, small | `Qwen3VL2BPreset` (2B) |
| Image understanding, mid | `Qwen25VL3BPreset` (3B) |
| Text-heavy images (OCR-style) | `Gemma3VisionPreset` |
| Strongest vision reasoning | `Ministral3VisionPreset` (8B) |

## Decision tree

1. **Do you need vision (image input)?**
   - Yes → pick a vision preset based on size and image type.
   - No → continue.

2. **Is the task coding?**
   - Yes → `DeepSeekCoder2Preset`.
   - No → continue.

3. **Does the task require explicit step-by-step reasoning?**
   - Yes → `DeepseekR1Qwen3Preset` or `Oss20Preset` (budget 1024-2048 `MaxTokens`).
   - No → continue.

4. **How much memory do you have?**
   - 4-8 GB → `Llama32Preset` or `Phi4Preset`.
   - 12-16 GB → `Qwen25Preset` or `Qwen3Preset`.
   - 24+ GB → any preset; `Oss20Preset` for best quality.

## After you pick

Override the default values where they do not fit your scenario. See [Customizing presets](/net/developer-reference/presets/customizing/).

If none of the built-ins fit, [bring your own GGUF](/net/use-cases/bring-your-own-gguf/).

## What's next

- [Supported presets](/net/product-overview/supported-presets/) — catalog with Hugging Face sources.
- [Using built-in presets](/net/developer-reference/presets/using-built-in/) — full picker guidance.
- [Custom preset](/net/use-cases/custom-preset/) — patterns for tuning.

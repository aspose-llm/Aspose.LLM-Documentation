---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/product-overview/supported-presets/
feedback: LLMNET
version: 26.5.0
title: Supported presets
description: Built-in presets for Aspose.LLM for .NET with Hugging Face model sources, default context sizes, and quantization.
keywords:
- preset
- model
- Qwen
- Gemma
- Llama
- Phi
- DeepSeek
- gpt-oss
- Ministral
- vision
- GGUF
---

Aspose.LLM for .NET bundles ready-to-use presets for several popular open-weight model families. Each preset specifies the model source (Hugging Face repository and file name), context size, chat template, and sampler defaults. Pass a preset to `AsposeLLMApi.Create(preset)` and the engine downloads the model and any vision projector on first use.

All presets derive from [`PresetCoreBase`](/net/developer-reference/presets/) (namespace `Aspose.LLM.Abstractions.Parameters.Presets`). You can use a preset as-is, override any parameter before calling `Create`, or extend `PresetCoreBase` for a fully custom model.

## Text presets

| Preset | Model | Hugging Face source | Default context | Quantization |
|---|---|---|---|---|
| `Qwen25Preset` | Qwen 2.5 7B Instruct | `bartowski/Qwen2.5-7B-Instruct-GGUF` | 32 768 | Q4_K_M |
| `Qwen3Preset` | Qwen 3 8B | `bartowski/Qwen_Qwen3-8B-GGUF` | 32 768 | Q4_K_M |
| `Gemma3Preset` | Gemma 3 | `mradermacher/gemma-3-GGUF` | 8 192 | Q4_K_M |
| `Llama32Preset` | Llama 3.2 3B Instruct | `bartowski/Llama-3.2-3B-Instruct-GGUF` | 131 072 | Q4_K_M |
| `Phi4Preset` | Phi 4 mini Instruct | `bartowski/Phi-4-mini-Instruct-GGUF` | 16 384 | Q4_K_M |
| `Oss20Preset` | gpt-oss 20B (multilingual-reasoner fine-tune) | `mradermacher/gpt-oss-20b-multilingual-reasoner-i1-GGUF` | 131 072 | Q4_K_M |
| `DeepSeekCoder2Preset` | DeepSeek-Coder-V2-Lite Instruct | `lmstudio-community/DeepSeek-Coder-V2-Lite-Instruct-GGUF` | 163 840 | IQ3_M |
| `DeepseekR1Qwen3Preset` | DeepSeek-R1 distilled from Qwen 3 8B | `lmstudio-community/DeepSeek-R1-0528-Qwen3-8B-GGUF` | 131 072 | Q4_K_M |
| `UnifiedDefaultLlmParameters` | Baseline template (no model source) | — | 4 096 | — |

Notes:

- `UnifiedDefaultLlmParameters` is a conservative CPU-safe template. It sets only context, threads, and sampler defaults; you must set a model source yourself before calling `Create`.
- Memory requirements scale with context size and quantization. A 7B Q4_K_M model at 32K context needs roughly 6-8 GB of RAM (or GPU memory) for the weights plus KV cache. Longer contexts and larger models need more — see [Features](/net/product-overview/features/) for memory guidance.
- All presets ship with preset-specific sampler tuning (temperature, repetition penalty, penalty context size, etc.). Override any field on `preset.SamplerParameters` before calling `Create` to change behavior.

## Vision presets

Vision presets configure both the base language model and its multimodal projector (`mmproj`). Pass image bytes via the `media` parameter of `SendMessageAsync` or `SendMessageToSessionAsync`.

| Preset | Model | Hugging Face source | `mmproj` file | Default context | Quantization |
|---|---|---|---|---|---|
| `Qwen25VL3BPreset` | Qwen 2.5 VL 3B Instruct | `unsloth/Qwen2.5-VL-3B-Instruct-GGUF` | `mmproj-F16.gguf` | 128 000 | UD-IQ2_XXS |
| `Qwen3VL2BPreset` | Qwen 3 VL 2B Instruct | `Qwen/Qwen3-VL-2B-Instruct-GGUF` | `mmproj-Qwen3VL-2B-Instruct-Q8_0.gguf` | 262 144 | Q4_K_M |
| `Gemma3VisionPreset` | Gemma 3 Vision (Latex fine-tune) | `mradermacher/Gemma-3-Vision-Latex-GGUF` | `Gemma-3-Vision-Latex.mmproj-f16.gguf` | 8 096 | Q4_K_M |
| `Ministral3VisionPreset` | Ministral 3 8B Instruct (Mistral AI, 2512 release) | `mistralai/Ministral-3-8B-Instruct-2512-GGUF` | `Ministral-3-8B-Instruct-2512-BF16-mmproj.gguf` | 262 144 | Q4_K_M |

Supported image formats across all vision presets: JPEG, PNG, BMP, GIF, WebP. Maximum per-attachment size: 50 MB.

## Default preset

`AsposeLLMApi.GetDefaultPreset()` returns a fresh `Qwen25Preset` instance — useful as a sensible starting point when you do not know which preset to pick. For raw parameter values without a full preset, call `await api.GetDefaultParametersAsync()`.

## Picking a preset

| If you want… | Try |
|---|---|
| A balanced general-purpose model | `Qwen25Preset` or `Qwen3Preset` |
| A small, fast model | `Llama32Preset` (3B) or `Phi4Preset` (mini) |
| A long-context model | `Llama32Preset` or `Oss20Preset` (131K), `DeepSeekCoder2Preset` (163K) |
| A coding-focused model | `DeepSeekCoder2Preset` |
| A reasoning-tuned model | `DeepseekR1Qwen3Preset` or `Oss20Preset` (multilingual-reasoner) |
| Image input | `Qwen3VL2BPreset` (small, very long context) or `Qwen25VL3BPreset` (3B) |
| A Mistral family vision model | `Ministral3VisionPreset` |

## What's next

- [Presets](/net/developer-reference/presets/) — preset base class, parameter bags, and override patterns.
- [Custom preset](/net/use-cases/custom-preset/) — extend or replace a built-in preset.
- [Features](/net/product-overview/features/) — full list of capabilities and limits.
- [Hello, world!](/net/hello-world/) — a minimal runnable example using `Qwen25Preset`.

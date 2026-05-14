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

All presets derive from [`PresetCoreBase`](/llm/net/developer-reference/presets/) (namespace `Aspose.LLM.Abstractions.Parameters.Presets`). You can use a preset as-is, override any parameter before calling `Create`, or extend `PresetCoreBase` for a fully custom model.

## Text presets

The text catalog is grouped by size and specialty. All presets ship `Q4_K_M` quantization unless noted otherwise. Every preset can be used out of the box — pass it to `AsposeLLMApi.Create(preset)` and the engine downloads the GGUF from the listed Hugging Face source on first run.

### Large general-purpose (7-8B)

The default tier for production chat. Balanced quality and speed; expect 6-10 GB RAM/VRAM at 32K context.

| Preset | Model | Hugging Face source | Default context |
|---|---|---|---|
| `Qwen25Preset` | Qwen 2.5 7B Instruct | `bartowski/Qwen2.5-7B-Instruct-GGUF` | 32 768 |
| `Qwen3Preset` | Qwen 3 8B | `bartowski/Qwen_Qwen3-8B-GGUF` | 32 768 |
| `Llama31_8BPreset` | Meta Llama 3.1 8B Instruct | `bartowski/Meta-Llama-3.1-8B-Instruct-GGUF` | 32 768 |
| `Mistral7Preset` | Mistral 7B Instruct v0.3 | `bartowski/Mistral-7B-Instruct-v0.3-GGUF` | 32 768 |
| `Granite3_8BPreset` | IBM Granite 3.1 8B Instruct | `bartowski/granite-3.1-8b-instruct-GGUF` | 32 768 |
| `AyaExpanse8BPreset` | Cohere Aya Expanse 8B (multilingual, 23 languages) | `bartowski/aya-expanse-8b-GGUF` | 8 192 |
| `OpenChat3_5Preset` | OpenChat 3.5 (Mistral-7B base) | `TheBloke/openchat-3.5-0106-GGUF` | 8 192 |
| `Gemma3Preset` | Google Gemma 3 | `mradermacher/gemma-3-GGUF` | 8 192 |

### Mid-size (3-6B)

Sweet spot for laptops with a discrete GPU or 16 GB-class systems.

| Preset | Model | Hugging Face source | Default context |
|---|---|---|---|
| `Yi_6BPreset` | 01.AI Yi 1.5 6B Chat | `bartowski/Yi-1.5-6B-Chat-GGUF` | 8 192 |
| `Phi35MiniPreset` | Microsoft Phi 3.5 Mini Instruct (~3.8B) | `bartowski/Phi-3.5-mini-instruct-GGUF` | 32 768 |
| `Phi4Preset` | Microsoft Phi 4 Mini Instruct | `unsloth/Phi-4-mini-instruct-GGUF` | 16 384 |
| `MiniCPM3_4BPreset` | OpenBMB MiniCPM3 4B | `openbmb/MiniCPM3-4B-GGUF` | 32 768 |
| `Llama32Preset` | Meta Llama 3.2 3B Instruct | `bartowski/Llama-3.2-3B-Instruct-GGUF` | 131 072 |
| `Qwen25_3BPreset` | Qwen 2.5 3B Instruct | `Qwen/Qwen2.5-3B-Instruct-GGUF` | 32 768 |

### Small and edge (≤2B)

CPU-only deployments, tutorials, smoke tests, and constrained-memory hosts.

| Preset | Model | Hugging Face source | Default context |
|---|---|---|---|
| `SmolLM2_1_7BPreset` | HuggingFaceTB SmolLM2 1.7B Instruct | `HuggingFaceTB/SmolLM2-1.7B-Instruct-GGUF` | 8 192 |
| `Llama32_1BPreset` | Meta Llama 3.2 1B Instruct (edge sibling of `Llama32Preset`) | `bartowski/Llama-3.2-1B-Instruct-GGUF` | 16 384 |
| `TinyLlamaPreset` | TinyLlama 1.1B Chat v1.0 (smoke-test baseline) | `TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF` | 2 048 |
| `SmallModelPreset` | Qwen 2 0.5B Instruct (CPU-first; ~400 MB on disk) | `QuantFactory/Qwen2-0.5B-Instruct-GGUF` | 4 096 |

`SmallModelPreset` defaults to `GpuLayers = 0`, `OffloadKqv = false`, `FlashAttention = false` so it runs on any laptop without a GPU. Switch to `GpuLayers = -1` to offload everything to GPU when one is available.

### Coding-focused

Trained or fine-tuned on code; pick these over the general models if you need accurate completions and refactoring suggestions.

| Preset | Model | Hugging Face source | Default context | Quantization |
|---|---|---|---|---|
| `DeepSeekCoder2Preset` | DeepSeek-Coder-V2-Lite Instruct | `lmstudio-community/DeepSeek-Coder-V2-Lite-Instruct-GGUF` | 163 840 | IQ3_M |
| `Qwen25Coder7BPreset` | Qwen 2.5 Coder 7B Instruct | `Qwen/Qwen2.5-Coder-7B-Instruct-GGUF` | 32 768 | Q4_K_M |
| `StableCode3BPreset` | Stability AI Stable Code 3B | `TheBloke/stable-code-3b-GGUF` | 16 384 | Q4_K_M |

### Reasoning / chain-of-thought

These models emit explicit step-by-step reasoning. Budget `MaxTokens = 1024-2048` and expect noticeably higher latency than a general 7B preset.

| Preset | Model | Hugging Face source | Default context |
|---|---|---|---|
| `DeepseekR1Qwen3Preset` | DeepSeek-R1 distilled from Qwen 3 8B | `lmstudio-community/DeepSeek-R1-0528-Qwen3-8B-GGUF` | 131 072 |
| `Oss20Preset` | gpt-oss 20B (multilingual reasoner fine-tune) | `mradermacher/gpt-oss-20b-multilingual-reasoner-i1-GGUF` | 131 072 |

### Baseline template

| Preset | Model | Hugging Face source | Default context | Quantization |
|---|---|---|---|---|
| `UnifiedDefaultLlmParameters` | Baseline template (no model source) | — | 4 096 | — |

`UnifiedDefaultLlmParameters` is a conservative CPU-safe template. It sets only context, threads, and sampler defaults; you must set a model source yourself before calling `Create`.

Notes:

- Memory requirements scale with context size and quantization. A 7B Q4_K_M model at 32K context needs roughly 6-8 GB of RAM (or GPU memory) for the weights plus KV cache. Longer contexts and larger models need more — see [Features](/llm/net/product-overview/features/) for memory guidance.
- All presets ship with preset-specific sampler tuning (temperature, repetition penalty, penalty context size, etc.). Override any field on `preset.SamplerParameters` before calling `Create` to change behavior.
- Hugging Face sources are verified publicly accessible at release time (no gated repos, no removed mirrors). If a source becomes unavailable, set `preset.BaseModelSourceParameters` to a different mirror or local file before calling `Create`.

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
| A balanced general-purpose model | `Qwen25Preset`, `Qwen3Preset`, `Llama31_8BPreset`, or `Mistral7Preset` |
| A small, fast model | `Llama32Preset` (3B), `Qwen25_3BPreset` (3B), or `Phi4Preset` (mini) |
| The smallest possible footprint | `SmallModelPreset` (0.5B CPU-first), `TinyLlamaPreset` (1.1B), or `Llama32_1BPreset` (1B) |
| A long-context model | `Llama32Preset` or `Oss20Preset` (131K), `DeepSeekCoder2Preset` (163K) |
| A coding-focused model | `DeepSeekCoder2Preset`, `Qwen25Coder7BPreset`, or `StableCode3BPreset` |
| A reasoning-tuned model | `DeepseekR1Qwen3Preset` or `Oss20Preset` (multilingual-reasoner) |
| Strong multilingual coverage | `AyaExpanse8BPreset` (23 languages) or `Oss20Preset` |
| An enterprise-tuned model | `Granite3_8BPreset` (IBM Granite 3.1) |
| Image input | `Qwen3VL2BPreset` (small, very long context) or `Qwen25VL3BPreset` (3B) |
| A Mistral family vision model | `Ministral3VisionPreset` |

## What's next

- [Presets](/llm/net/developer-reference/presets/) — preset base class, parameter bags, and override patterns.
- [Custom preset](/llm/net/use-cases/custom-preset/) — extend or replace a built-in preset.
- [Features](/llm/net/product-overview/features/) — full list of capabilities and limits.
- [Hello, world!](/llm/net/hello-world/) — a minimal runnable example using `Qwen25Preset`.

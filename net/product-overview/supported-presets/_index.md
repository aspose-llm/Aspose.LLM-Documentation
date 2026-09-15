---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/product-overview/supported-presets/
feedback: LLMNET
version: 26.6.0
title: Supported presets
description: Built-in presets for Aspose.LLM for .NET with Hugging Face model sources, default context sizes, and quantization.
keywords:
- preset
- model
- CPU
- Qwen
- Gemma
- Llama
- Phi
- Mistral
- DeepSeek
- gpt-oss
- Ministral
- Hermes
- Granite
- OLMo
- Devstral
- LFM
- vision
- GGUF
---

Aspose.LLM for .NET ships ready-to-use presets for several popular open-weight model families. Each preset specifies the model source (Hugging Face repository and file name), context size, chat template, and sampler defaults. Pass a preset to `AsposeLLMApi.Create(preset)` and the engine downloads the model and any vision projector on first use.

{{% alert color="warning" %}}
**No built-in LLM is included in Aspose.LLM, so you choose and install your favorite LLM on your own.** A preset is configuration, not a model: a ready-made set of values for one open source model, namely context size, sampling parameters, chat template, and which model file to load. No model weights ship inside the package.

Aspose.LLM supports open source LLMs from every major family, Llama included. See [Supported LLMs](/llm/net/product-overview/supported-llms/) for the full list and the license of each. Any other compatible open source model can be used instead, by pointing the API at it. You are responsible for complying with the license of the model you choose.

This design is what the [AI Governance and Risk Management](https://trust.aspose.com/app-security/ai-governance-and-risk-management/) policy at the Aspose Trust Center relies on: the product integrates AI models, it does not include or provide them.
{{% /alert %}}

All presets derive from [`PresetCoreBase`](/llm/net/developer-reference/presets/) (namespace `Aspose.LLM.Abstractions.Parameters.Presets`). You can use a preset as-is, override any parameter before calling `Create`, or extend `PresetCoreBase` for a fully custom model.

Version 26.6.0 ships **74 model presets**: 41 text presets, 6 vision presets, 27 CPU-tuned twins ([`*PresetCpu`](#cpu-tuned-variants-presetcpu)), plus the `UnifiedDefaultLlmParameters` baseline template.

## Text presets

The text catalog is grouped by size and specialty. All presets ship `Q4_K_M` quantization unless noted otherwise. Every preset can be used out of the box: pass it to `AsposeLLMApi.Create(preset)` and the engine downloads the GGUF from the listed Hugging Face source on first run.

### Large general-purpose (7-8B)

The default tier for production chat. Balanced quality and speed; expect 6-10 GB RAM/VRAM at 32K context.

| Preset | Model | Hugging Face source | Default context | License |
|---|---|---|---|---|
| `Qwen25Preset` | Qwen 2.5 7B Instruct | `bartowski/Qwen2.5-7B-Instruct-GGUF` | 32 768 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Qwen3Preset` | Qwen 3 8B | `bartowski/Qwen_Qwen3-8B-GGUF` | 32 768 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Qwen3_5_9BPreset` | Qwen 3.5 9B | `unsloth/Qwen3.5-9B-GGUF` | 32 768 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Llm31_8BPreset` | Meta Llama 3.1 8B Instruct | `bartowski/Meta-Llama-3.1-8B-Instruct-GGUF` | 32 768 | [Community license](https://developer.meta.com/ai/llama3_1/license/) |
| `Mistral7Preset` | Mistral 7B Instruct v0.3 | `bartowski/Mistral-7B-Instruct-v0.3-GGUF` | 32 768 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Hermes3_8BPreset` | NousResearch Hermes 3 (Llama 3.1 8B base) | `bartowski/Hermes-3-Llama-3.1-8B-GGUF` | 32 768 | [Community license](https://developer.meta.com/ai/llama3_1/license/) (inherited from the base model) |
| `Granite3_8BPreset` | IBM Granite 3.1 8B Instruct | `bartowski/granite-3.1-8b-instruct-GGUF` | 32 768 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `AyaExpanse8BPreset` | Cohere Aya Expanse 8B (multilingual, 23 languages) | `bartowski/aya-expanse-8b-GGUF` | 8 192 | [CC-BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/): **non-commercial use only** |
| `OpenChat3_5Preset` | OpenChat 3.5 (Mistral-7B base) | `TheBloke/openchat-3.5-0106-GGUF` | 8 192 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Olmo2_7BPreset` | AllenAI OLMo 2 7B Instruct (fully-open research model) | `bartowski/OLMo-2-1124-7B-Instruct-GGUF` | 4 096 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) plus a vendor rider referencing the [Gemma Terms of Use](https://ai.google.dev/gemma/terms) |
| `Gemma3Preset` | Google Gemma 3 4B Instruct (text path) | `ggml-org/gemma-3-4b-it-GGUF` | 8 192 | [Gemma Terms of Use](https://ai.google.dev/gemma/terms) |

### Mid-size (3-6B)

Sweet spot for laptops with a discrete GPU or 16 GB-class systems.

| Preset | Model | Hugging Face source | Default context | License |
|---|---|---|---|---|
| `Yi_6BPreset` | 01.AI Yi 1.5 6B Chat | `bartowski/Yi-1.5-6B-Chat-GGUF` | 8 192 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Phi35MiniPreset` | Microsoft Phi 3.5 Mini Instruct (~3.8B) | `bartowski/Phi-3.5-mini-instruct-GGUF` | 32 768 | [MIT](https://opensource.org/license/mit) |
| `Phi4Preset` | Microsoft Phi 4 Mini Instruct | `unsloth/Phi-4-mini-instruct-GGUF` | 16 384 | [MIT](https://opensource.org/license/mit) |
| `MiniCPM3_4BPreset` | OpenBMB MiniCPM3 4B | `openbmb/MiniCPM3-4B-GGUF` | 32 768 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Gemma3nE4BPreset` | Google Gemma 3n E4B-it (per-layer-embedding variant) | `unsloth/gemma-3n-E4B-it-GGUF` | 32 768 | [Gemma Terms of Use](https://ai.google.dev/gemma/terms) |
| `Llm32Preset` | Meta Llama 3.2 3B Instruct | `bartowski/Llama-3.2-3B-Instruct-GGUF` | 131 072 | [Community license](https://developer.meta.com/ai/llama3_2/license/) |
| `Qwen25_3BPreset` | Qwen 2.5 3B Instruct | `Qwen/Qwen2.5-3B-Instruct-GGUF` | 32 768 | Qwen Research License: **non-commercial use only** |
| `Glm4_7FlashPreset` | Zhipu GLM-4.7 Flash | `unsloth/GLM-4.7-Flash-GGUF` | 32 768 | [MIT](https://opensource.org/license/mit) |

### Small and edge (≤2B)

CPU-only deployments, tutorials, smoke tests, and constrained-memory hosts.

| Preset | Model | Hugging Face source | Default context | License |
|---|---|---|---|---|
| `SmolLM2_1_7BPreset` | HuggingFaceTB SmolLM2 1.7B Instruct | `HuggingFaceTB/SmolLM2-1.7B-Instruct-GGUF` | 8 192 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Lfm2_1_2BPreset` | Liquid LFM2 1.2B (hybrid SSM + attention) | `LiquidAI/LFM2-1.2B-GGUF` | 32 768 | [LFM Open License](https://www.liquid.ai/lfm-license): **no commercial grant above USD 10M annual revenue** |
| `Lfm2_5_1_2BPreset` | Liquid LFM 2.5 1.2B Thinking | `unsloth/LFM2.5-1.2B-Thinking-GGUF` | 32 768 | [LFM Open License](https://www.liquid.ai/lfm-license): **no commercial grant above USD 10M annual revenue** |
| `Llm32_1BPreset` | Meta Llama 3.2 1B Instruct (edge sibling of `Llm32Preset`) | `bartowski/Llama-3.2-1B-Instruct-GGUF` | 16 384 | [Community license](https://developer.meta.com/ai/llama3_2/license/) |
| `TinyLlmPreset` | TinyLlama 1.1B Chat v1.0 (smoke-test baseline) | `TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF` | 2 048 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `SmallModelPreset` | Qwen 2 0.5B Instruct (CPU-first; ~400 MB on disk) | `QuantFactory/Qwen2-0.5B-Instruct-GGUF` | 4 096 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |

`SmallModelPreset` defaults to `GpuLayers = 0`, `OffloadKqv = false`, `FlashAttention = false` so it runs on any laptop without a GPU. Switch to `GpuLayers = -1` to offload everything to GPU when one is available.

### Frontier (10B+, large workstations)

Larger models for hosts with 32 GB+ system RAM or substantial VRAM. Run with partial GPU offload when VRAM is limited: the engine streams the rest from system memory. Some entries are intentionally CPU-only; their CPU twin (`*PresetCpu`) caps the context to 4 K to keep the KV cache within laptop-RAM bounds.

| Preset | Model | Hugging Face source | Default context | License |
|---|---|---|---|---|
| `Phi4_14BPreset` | Microsoft Phi 4 14B | `bartowski/phi-4-GGUF` | 16 384 | [MIT](https://opensource.org/license/mit) |
| `MistralSmall3Preset` | Mistral Small 3.1 24B Instruct (2503 release) | `unsloth/Mistral-Small-3.1-24B-Instruct-2503-GGUF` | 32 768 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Qwen3_6_27BPreset` | Qwen 3.6 27B | `unsloth/Qwen3.6-27B-GGUF` | 65 536 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `DevstralSmall2_24BPreset` | Mistral Devstral Small 2 24B Instruct (coding-tuned) | `unsloth/Devstral-Small-2-24B-Instruct-2512-GGUF` | 4 096 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Ernie4_5_21BPreset` | Baidu ERNIE 4.5 21B-A3B Thinking (reasoning MoE) | `bartowski/baidu_ERNIE-4.5-21B-A3B-Thinking-GGUF` | 4 096 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `SeedOss36BPreset` | ByteDance Seed-OSS 36B Instruct | `unsloth/Seed-OSS-36B-Instruct-GGUF` | 4 096 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Llm3_3_70BPreset` | Meta Llama 3.3 70B Instruct | `unsloth/Llama-3.3-70B-Instruct-GGUF` | 4 096 | [Community license](https://developer.meta.com/ai/llama3_3/license/) |
| `GptOss120BPreset` | gpt-oss 120B: **documentation-only**, hard-capped at 50 GB on disk | `unsloth/gpt-oss-120b-GGUF` | 8 192 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |

`GptOss120BPreset` is wired into the SDK so the configuration is documented, but the integration sweep auto-skips it: at ~65 GB on disk it exceeds the 50 GB hard cap. Use it only if you have the disk space and RAM headroom and accept the slow first-run download.

### Coding-focused

Trained or fine-tuned on code; pick these over the general models if you need accurate completions and refactoring suggestions.

| Preset | Model | Hugging Face source | Default context | Quantization | License |
|---|---|---|---|---|---|
| `DeepSeekCoder2Preset` | DeepSeek-Coder-V2-Lite Instruct | `lmstudio-community/DeepSeek-Coder-V2-Lite-Instruct-GGUF` | 163 840 | IQ3_M | [DeepSeek License v1.0](https://github.com/deepseek-ai/DeepSeek-Coder-V2/blob/main/LICENSE-MODEL): use restrictions flow down to you |
| `Qwen25Coder7BPreset` | Qwen 2.5 Coder 7B Instruct | `Qwen/Qwen2.5-Coder-7B-Instruct-GGUF` | 32 768 | Q4_K_M | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Qwen3Coder30BPreset` | Qwen 3 Coder 30B-A3B Instruct (MoE) | `unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF` | 65 536 | Q4_K_M | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `StableCode3BPreset` | Stability AI Stable Code 3B | `TheBloke/stable-code-3b-GGUF` | 16 384 | Q4_K_M | [Stability AI Community License](https://stability.ai/community-license-agreement): **grant ends above USD 1M annual revenue** |

For an even larger coding-tuned model see `DevstralSmall2_24BPreset` in the Frontier section above.

### Reasoning / chain-of-thought

These models emit explicit step-by-step reasoning. Budget `MaxTokens = 1024-2048` and expect noticeably higher latency than a general 7B preset.

| Preset | Model | Hugging Face source | Default context | License |
|---|---|---|---|---|
| `DeepseekR1Qwen3Preset` | DeepSeek-R1 distilled from Qwen 3 8B | `lmstudio-community/DeepSeek-R1-0528-Qwen3-8B-GGUF` | 131 072 | [MIT](https://opensource.org/license/mit) |
| `Oss20Preset` | OpenAI GPT-OSS 20B (native MXFP4) | `ggml-org/gpt-oss-20b-GGUF` | 131 072 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |

### Limited compatibility

These presets are shipped so the wiring is documented and ready, but the current llama.cpp runtime (`b8816`) cannot drive them end-to-end. Calls to `Create` succeed; the integration sweep marks them `Inconclusive` until a future runtime upgrade lands the missing chat-template handling.

| Preset | Model | Hugging Face source | Default context | Status | License |
|---|---|---|---|---|---|
| `NemotronNano4BPreset` | NVIDIA Nemotron Mini 4B Instruct | `bartowski/Nemotron-Mini-4B-Instruct-GGUF` | 32 768 | Chat template `<extra_id_0>` / `<extra_id_1>` not in llama.cpp's built-in templater | NVIDIA open model license: **the upstream record is inconsistent; verify before commercial use** |
| `Gemma4_E4B_ItPreset` | Google Gemma 4 E4B-it | `unsloth/gemma-4-E4B-it-GGUF` | 32 768 | Chat template uses Jinja macros that llama.cpp `b8816` cannot parse | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |

Use these presets at your own risk; once you call `Create`, set a custom `ChatParameters.PromptFormatter` to bypass the broken template.

### Baseline template

| Preset | Model | Hugging Face source | Default context | Quantization | License |
|---|---|---|---|---|---|
| `UnifiedDefaultLlmParameters` | Baseline template (no model source) | - | 4 096 | - | n/a (no model source) |

`UnifiedDefaultLlmParameters` is a conservative CPU-safe template. It sets only context, threads, and sampler defaults; you must set a model source yourself before calling `Create`.

Notes:

- Memory requirements scale with context size and quantization. A 7B Q4_K_M model at 32K context needs roughly 6-8 GB of RAM (or GPU memory) for the weights plus KV cache. Longer contexts and larger models need more: see [Features](/llm/net/product-overview/features/) for memory guidance.
- All presets ship with preset-specific sampler tuning (temperature, repetition penalty, penalty context size, etc.). Override any field on `preset.SamplerParameters` before calling `Create` to change behavior.
- Hugging Face sources are verified publicly accessible at release time (no gated repos, no removed mirrors). If a source becomes unavailable, set `preset.BaseModelSourceParameters` to a different mirror or local file before calling `Create`.

## CPU-tuned variants (`*PresetCpu`)

27 text presets ship a CPU-tuned twin: the parent class name plus a `Cpu` suffix. The twin is a `sealed` subclass of the GPU preset whose constructor calls the parent and then applies one shared set of CPU overrides, so **model identity is inherited unchanged**: same Hugging Face repository, same GGUF file, same quantization, same chat template, same system prompt, same sampler tuning, same `MaxTokens`.

Only the hardware-facing parameters differ:

| Parameter | GPU parent | `*PresetCpu` twin |
|---|---|---|
| `BaseModelInferenceParameters.GpuLayers` | preset-specific layer count (commonly 24-36; `-1` means all layers) | `0`: CPU only |
| `ContextParameters.ContextSize` | preset-specific, up to 65 536 | capped at `4096` (never raised) |
| `ContextParameters.NBatch` | preset-specific | capped at `512` |
| `ContextParameters.NUbatch` | preset-specific | capped at `256` |
| `ContextParameters.FlashAttention` | preset-specific | `false` |
| `ContextParameters.OffloadKqv` | preset-specific | `false` |
| `BaseModelInferenceParameters.UseExtraBuffers` | preset-specific | `false` |
| `BaseModelInferenceParameters.UseMemoryLocking` | preset-specific | `false` |

The caps are one-way: they only ever lower a value. A parent already below a cap keeps its own setting, which is why `TinyLlmPresetCpu` stays at 2 048 rather than being raised to 4 096.

Reach for the twin instead of setting `GpuLayers = 0` by hand: it is the canonical CPU configuration and disables the GPU-side code paths (`FlashAttention`, KV-cache offload, extra buffers, memory locking) that would otherwise be requested and silently ignored or fail on a CPU-only host.

```csharp
using var api = AsposeLLMApi.Create(new Mistral7PresetCpu());
```

A twin loads the same model file as its parent preset, so the license shown for the parent in the tables above applies to the twin unchanged.

### Available twins

Grouped by the parent's tier. "Parent context" is the GPU default; "CPU context" is what the twin actually runs at.

#### Large general-purpose (7-9B)

| CPU preset | Parent | Parent context | CPU context |
|---|---|---|---|
| `Qwen3_5_9BPresetCpu` | `Qwen3_5_9BPreset` | 32 768 | 4 096 |
| `Llm31_8BPresetCpu` | `Llm31_8BPreset` | 32 768 | 4 096 |
| `Mistral7PresetCpu` | `Mistral7Preset` | 32 768 | 4 096 |
| `Hermes3_8BPresetCpu` | `Hermes3_8BPreset` | 32 768 | 4 096 |
| `Granite3_8BPresetCpu` | `Granite3_8BPreset` | 32 768 | 4 096 |
| `AyaExpanse8BPresetCpu` | `AyaExpanse8BPreset` | 8 192 | 4 096 |
| `OpenChat3_5PresetCpu` | `OpenChat3_5Preset` | 8 192 | 4 096 |
| `Olmo2_7BPresetCpu` | `Olmo2_7BPreset` | 4 096 | 4 096 (unchanged) |

#### Mid-size (3-6B)

| CPU preset | Parent | Parent context | CPU context |
|---|---|---|---|
| `Yi_6BPresetCpu` | `Yi_6BPreset` | 8 192 | 4 096 |
| `Phi35MiniPresetCpu` | `Phi35MiniPreset` | 32 768 | 4 096 |
| `MiniCPM3_4BPresetCpu` | `MiniCPM3_4BPreset` | 32 768 | 4 096 |
| `Gemma3nE4BPresetCpu` | `Gemma3nE4BPreset` | 32 768 | 4 096 |
| `Qwen25_3BPresetCpu` | `Qwen25_3BPreset` | 32 768 | 4 096 |
| `Glm4_7FlashPresetCpu` | `Glm4_7FlashPreset` | 32 768 | 4 096 |

#### Small and edge (≤2B)

| CPU preset | Parent | Parent context | CPU context |
|---|---|---|---|
| `SmolLM2_1_7BPresetCpu` | `SmolLM2_1_7BPreset` | 8 192 | 4 096 |
| `Lfm2_1_2BPresetCpu` | `Lfm2_1_2BPreset` | 32 768 | 4 096 |
| `Lfm2_5_1_2BPresetCpu` | `Lfm2_5_1_2BPreset` | 32 768 | 4 096 |
| `Llm32_1BPresetCpu` | `Llm32_1BPreset` | 16 384 | 4 096 |
| `TinyLlmPresetCpu` | `TinyLlmPreset` | 2 048 | 2 048 (unchanged) |

#### Frontier (10B+)

Usable on a CPU-only host, but slow: a 24-27B Q4_K_M model still needs 15-20 GB of resident RAM even at 4 K context, and generation runs at low single-digit tokens/second on a typical laptop CPU. Prefer a smaller twin unless you specifically need the larger model's quality.

| CPU preset | Parent | Parent context | CPU context |
|---|---|---|---|
| `Phi4_14BPresetCpu` | `Phi4_14BPreset` | 16 384 | 4 096 |
| `MistralSmall3PresetCpu` | `MistralSmall3Preset` | 32 768 | 4 096 |
| `Qwen3_6_27BPresetCpu` | `Qwen3_6_27BPreset` | 65 536 | 4 096 |

#### Coding-focused

| CPU preset | Parent | Parent context | CPU context |
|---|---|---|---|
| `Qwen25Coder7BPresetCpu` | `Qwen25Coder7BPreset` | 32 768 | 4 096 |
| `Qwen3Coder30BPresetCpu` | `Qwen3Coder30BPreset` | 65 536 | 4 096 |
| `StableCode3BPresetCpu` | `StableCode3BPreset` | 16 384 | 4 096 |

The 4 K cap bites hardest here: repository-scale prompts that fit the parent's 32-64 K context will not fit the twin. Raise `ContextParameters.ContextSize` after construction if you have the RAM.

#### Limited compatibility

These inherit the parent's runtime limitation: the CPU overrides do not fix the chat-template problem described in [Limited compatibility](#limited-compatibility) above.

| CPU preset | Parent | Parent context | CPU context |
|---|---|---|---|
| `NemotronNano4BPresetCpu` | `NemotronNano4BPreset` | 32 768 | 4 096 |
| `Gemma4_E4B_ItPresetCpu` | `Gemma4_E4B_ItPreset` | 32 768 | 4 096 |

### Presets with no CPU twin

Not every preset has a `Cpu` sibling. Do not guess at a name: `Qwen25PresetCpu`, `Qwen3PresetCpu`, `Gemma3PresetCpu`, `Llama32PresetCpu`, and `Phi4PresetCpu` do not exist and will not compile.

| Preset | Why no twin |
|---|---|
| `SmallModelPreset` | Already CPU-first: ships `GpuLayers = 0`, `OffloadKqv = false`, `FlashAttention = false` by default. |
| `UnifiedDefaultLlmParameters` | Conservative CPU-safe baseline template, not a model preset. |
| `Qwen25Preset`, `Qwen3Preset`, `Gemma3Preset`, `Llm32Preset`, `Phi4Preset` | No twin shipped in 26.6.0. Apply the overrides by hand: see [CPU-only deployment](/llm/net/use-cases/cpu-only-deployment/). |
| `DeepSeekCoder2Preset`, `DeepseekR1Qwen3Preset`, `Oss20Preset` | Long-context (131 K-163 K) models whose value depends on the context the 4 K cap would remove. |
| `DevstralSmall2_24BPreset`, `Ernie4_5_21BPreset`, `SeedOss36BPreset`, `Llm3_3_70BPreset`, `GptOss120BPreset` | Already ship a 4 K-8 K default context; set `GpuLayers = 0` directly for CPU-only runs. |
| All vision presets | The multimodal projector path is not part of the CPU-defaults contract. |

To get twin-equivalent behavior on any preset without one, apply the same overrides yourself:

```csharp
var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.GpuLayers = 0;
preset.BaseModelInferenceParameters.UseExtraBuffers = false;
preset.BaseModelInferenceParameters.UseMemoryLocking = false;
preset.ContextParameters.ContextSize = 4096;
preset.ContextParameters.NBatch = 512;
preset.ContextParameters.NUbatch = 256;
preset.ContextParameters.FlashAttention = false;
preset.ContextParameters.OffloadKqv = false;

using var api = AsposeLLMApi.Create(preset);
```

### Raising the context on a twin

The 4 K cap is applied once, in the constructor. Assigning a larger value afterwards is honored: nothing re-clamps it:

```csharp
var preset = new Llm31_8BPresetCpu();
preset.ContextParameters.ContextSize = 16384;   // opt back in, if RAM allows

using var api = AsposeLLMApi.Create(preset);
```

Budget for it: the KV cache grows linearly with context, so an 8B Q4_K_M model at 16 K needs roughly 3 GB more resident RAM than the same model at 4 K. See [Low-memory tuning](/llm/net/use-cases/low-memory-tuning/) for the full memory model.

## Vision presets

Vision presets configure both the base language model and its multimodal projector (`mmproj`). Pass image bytes via the `media` parameter of `SendMessageAsync` or `SendMessageToSessionAsync`.

| Preset | Model | Hugging Face source | `mmproj` file | Default context | Quantization | License |
|---|---|---|---|---|---|---|
| `Qwen25VL3BPreset` | Qwen 2.5 VL 3B Instruct | `unsloth/Qwen2.5-VL-3B-Instruct-GGUF` | `mmproj-F16.gguf` | 128 000 | Q4_K_M | Qwen Research License: **non-commercial use only** |
| `Qwen3VL2BPreset` | Qwen 3 VL 2B Instruct | `Qwen/Qwen3-VL-2B-Instruct-GGUF` | `mmproj-Qwen3VL-2B-Instruct-Q8_0.gguf` | 262 144 | Q4_K_M | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Gemma3VisionPreset` | Google Gemma 3 4B Instruct (vision path) | `ggml-org/gemma-3-4b-it-GGUF` | `mmproj-model-f16.gguf` | 8 096 | Q4_K_M | [Gemma Terms of Use](https://ai.google.dev/gemma/terms) |
| `Ministral3VisionPreset` | Ministral 3 8B Instruct (Mistral AI, 2512 release) | `mistralai/Ministral-3-8B-Instruct-2512-GGUF` | `Ministral-3-8B-Instruct-2512-BF16-mmproj.gguf` | 262 144 | Q4_K_M | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| `Glm4_6VFlashPreset` | Zhipu GLM-4.6V Flash vision | `unsloth/GLM-4.6V-Flash-GGUF` | bundled `mmproj` in same repo | 32 768 | Q4_K_M | [MIT](https://opensource.org/license/mit) |
| `NemotronOmniPreset` | NVIDIA Nemotron-3-Nano-Omni 30B-A3B Reasoning (multimodal MoE) | `unsloth/NVIDIA-Nemotron-3-Nano-Omni-30B-A3B-Reasoning-GGUF` | `mmproj-F16.gguf` | 8 192 | UD-Q4_K_M | [NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/) |

Supported image formats across all vision presets: JPEG, PNG, BMP, GIF, WebP. Maximum per-attachment size: 50 MB.

## Default preset

`AsposeLLMApi.GetDefaultPreset()` returns a fresh `Qwen25Preset` instance: useful as a sensible starting point when you do not know which preset to pick. For raw parameter values without a full preset, call `await api.GetDefaultParametersAsync()`.

## Picking a preset

| If you want… | Try |
|---|---|
| A balanced general-purpose model | `Qwen25Preset`, `Qwen3Preset`, `Llm31_8BPreset`, or `Mistral7Preset` |
| A small, fast model | `Llm32Preset` (3B) or `Phi4Preset` (mini) |
| The smallest possible footprint | `SmallModelPreset` (0.5B CPU-first), `TinyLlmPreset` (1.1B), or `Llm32_1BPreset` (1B) |
| A long-context model | `Llm32Preset` (131K) or `DeepSeekCoder2Preset` (163K) |
| A coding-focused model | `DeepSeekCoder2Preset`, `Qwen25Coder7BPreset`, `Qwen3Coder30BPreset`, or `DevstralSmall2_24BPreset` |
| A reasoning-tuned model | `DeepseekR1Qwen3Preset`, `Ernie4_5_21BPreset`, or `SeedOss36BPreset` |
| Strong multilingual coverage | `Qwen3Preset` or `MistralSmall3Preset` |
| An enterprise-tuned model | `Granite3_8BPreset` (IBM Granite 3.1) |
| A fully-open research model | `Olmo2_7BPreset` (AllenAI OLMo 2) |
| The largest model that fits a workstation | `Llm3_3_70BPreset`, `SeedOss36BPreset`, or `MistralSmall3Preset` |
| Image input | `Qwen3VL2BPreset` (small, very long context) or `Glm4_6VFlashPreset` |
| A Mistral family vision model | `Ministral3VisionPreset` |
| A multimodal reasoning model | `NemotronOmniPreset` (30B MoE, vision + reasoning) |
| To run without a GPU | The [`*PresetCpu` twin](#cpu-tuned-variants-presetcpu) of any preset above: e.g. `Mistral7PresetCpu`, `Phi35MiniPresetCpu`, `TinyLlmPresetCpu` |
| The lightest CPU-only option | `SmallModelPreset` (0.5B, CPU-first by default) or `TinyLlmPresetCpu` (1.1B, 2 K context) |

## Before you ship

{{% alert color="warning" %}}
**Check the license of the model you selected.** Aspose.LLM supplies the runtime, not the model. Whichever model you load, its terms come from the party that published it and they apply to your product. They are not part of, and are not covered by, your license agreement with Aspose Pty Ltd. Some open source models allow commercial use with no strings attached, others attach conditions such as attribution or an acceptable use policy, and a few exclude commercial use or withdraw it above a revenue threshold. [Supported LLMs](/llm/net/product-overview/supported-llms/) lists the license of every family the SDK ships a preset for.
{{% /alert %}}

## What's next

- [Presets](/llm/net/developer-reference/presets/): preset base class, parameter bags, and override patterns.
- [CPU-only deployment](/llm/net/use-cases/cpu-only-deployment/): running the `*PresetCpu` twins and tuning thread counts.
- [Custom preset](/llm/net/use-cases/custom-preset/): extend or replace a built-in preset.
- [Features](/llm/net/product-overview/features/): full list of capabilities and limits.
- [Hello, world!](/llm/net/hello-world/): a minimal runnable example using `Qwen25Preset`.

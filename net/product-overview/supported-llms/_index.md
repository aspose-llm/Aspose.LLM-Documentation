---
weight: 35
date: "2026-08-27"
author: "Alex Golshtein"
type: docs
url: /net/product-overview/supported-llms/
feedback: LLMNET
version: 26.6.0
title: Supported LLMs
description: Every open source large language model family Aspose.LLM for .NET supports, with the license of each family and whether it permits commercial use.
keywords:
- LLM
- open source
- license
- license
- commercial use
- model
- Llama
- Qwen
- Gemma
- Mistral
- Phi
- DeepSeek
---

Aspose.LLM for .NET runs open source large language models locally. This page lists every model family the library supports out of the box, the license each family is published under, and whether that license permits commercial use.

{{% alert color="warning" %}}
**No built-in LLM is included in Aspose.LLM, so you choose and install your favorite LLM on your own.** The library is a local inference runtime and ships no model weights. You pick the model, the model file is obtained separately and stored on your own machine, and it is covered by the license of the model publisher, not by your license agreement with Aspose Pty Ltd.
{{% /alert %}}

## Supported open source LLM families

Every family below can be used through a ready-made preset. Any other open source model compatible with the runtime can be used as well, by pointing the API at it: see [Creating a preset from scratch](/llm/net/developer-reference/presets/creating-from-scratch/).

The **Commercial use** column reflects the license of the model as published by its author. It is not legal advice, and it is your responsibility to review the license of any model you deploy.

<!-- EDITORS: update the date below every time the license information in this table is re-verified. -->
**License information last verified: 26 August 2026.** Each entry was checked against the exact repository its preset resolves to. Model publishers can change a license at any time, so this date is updated whenever the table is re-verified.

| Family | Publisher | License | Commercial use | Presets |
|---|---|---|---|---|
| Llama 3.1, 3.2, 3.3 | Meta | [Llama Community License](https://developer.meta.com/ai/llama3_2/license/) | Yes, with attribution and acceptable-use conditions | `Llm31_8BPreset`, `Llm32Preset`, `Llm32_1BPreset`, `Llm3_3_70BPreset` |
| Qwen 3, 3.5, 3.6 | Alibaba | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `Qwen3Preset`, `Qwen3_5_9BPreset`, `Qwen3_6_27BPreset`, `Qwen3Coder30BPreset`, `Qwen3VL2BPreset` |
| Qwen 2.5 | Alibaba | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0), except the 3B models | Yes, except the 3B models | `Qwen25Preset`, `Qwen25Coder7BPreset` |
| Qwen 2 | Alibaba | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `SmallModelPreset` (Qwen 2 0.5B) |
| Qwen 2.5 3B and VL 3B | Alibaba | Qwen Research License | **No, research use only** | `Qwen25_3BPreset`, `Qwen25VL3BPreset` |
| Gemma 3n | Google | [Gemma Terms of Use](https://ai.google.dev/gemma/terms) | Yes, subject to the Gemma prohibited use policy | `Gemma3nE4BPreset` |
| Gemma 4 | Google | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `Gemma4_E4B_ItPreset` |
| Mistral, Ministral, Devstral | Mistral AI | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `Mistral7Preset`, `MistralSmall3Preset`, `DevstralSmall2_24BPreset`, `Ministral3VisionPreset` |
| Phi 3.5, 4 | Microsoft | [MIT](https://opensource.org/license/mit) | Yes | `Phi35MiniPreset`, `Phi4Preset`, `Phi4_14BPreset` |
| gpt-oss | OpenAI | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `GptOss120BPreset` |
| DeepSeek Coder V2 | DeepSeek | [DeepSeek License v1.0](https://github.com/deepseek-ai/DeepSeek-Coder-V2/blob/main/LICENSE-MODEL) | Yes, with use restrictions that flow down to you | `DeepSeekCoder2Preset` |
| DeepSeek R1 distill | DeepSeek | [MIT](https://opensource.org/license/mit) | Yes | `DeepseekR1Qwen3Preset` |
| GLM 4.6V, 4.7 | Z.ai (Zhipu) | [MIT](https://opensource.org/license/mit) | Yes | `Glm4_7FlashPreset`, `Glm4_6VFlashPreset` |
| Nemotron Nano Omni | NVIDIA | [NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/) | Yes, with attribution and AI-ethics conditions | `NemotronOmniPreset` |
| Nemotron Mini | NVIDIA | NVIDIA open model license (the upstream license record is inconsistent) | **Verify with the publisher before commercial use** | `NemotronNano4BPreset` |
| Granite 3 | IBM | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `Granite3_8BPreset` |
| ERNIE 4.5 | Baidu | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `Ernie4_5_21BPreset` |
| Yi 1.5 | 01.AI | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `Yi_6BPreset` |
| OLMo 2 | Ai2 | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0), plus a publisher rider referencing the [Gemma Terms of Use](https://ai.google.dev/gemma/terms) | Yes, subject to that rider | `Olmo2_7BPreset` |
| MiniCPM 3 | OpenBMB | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `MiniCPM3_4BPreset` |
| SmolLM2 | Hugging Face | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `SmolLM2_1_7BPreset` |
| OpenChat 3.5 | OpenChat | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `OpenChat3_5Preset` |
| Seed-OSS | ByteDance | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `SeedOss36BPreset` |
| TinyLlama | StatNLP | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) | Yes | `TinyLlmPreset` |
| Hermes 3 | Nous Research | [Llama Community License](https://developer.meta.com/ai/llama3_1/license/), inherited from the base model | Yes, with attribution and acceptable-use conditions | `Hermes3_8BPreset` |
| Aya Expanse | Cohere | [CC-BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) | **No, non-commercial use only** | `AyaExpanse8BPreset` |
| LFM 2, 2.5 | Liquid AI | [LFM Open License](https://www.liquid.ai/lfm-license) | **Only below USD 10M annual revenue** | `Lfm2_1_2BPreset`, `Lfm2_5_1_2BPreset` |
| Stable Code | Stability AI | [Stability AI Community License](https://stability.ai/community-license-agreement) | **Only below USD 1M annual revenue** | `StableCode3BPreset` |
| Under source review | n/a | Not established for the current model source | **No, do not use commercially until the review completes** | `Gemma3Preset`, `Gemma3VisionPreset`, `Oss20Preset` |

## Reading the license column

Most families are published under **Apache-2.0** or **MIT**. These place no restrictions on commercial use beyond keeping the copyright notice.

Some families use a **publisher license** that permits commercial use but attaches conditions which reach you as the deployer: an attribution notice, an acceptable use policy, or a naming rule for derivative models. Read the linked license before shipping.

One row is marked **under source review**: the repository those presets currently resolve to could not be license-verified, and the presets are being retargeted to official sources. Until that completes, treat them as unavailable for commercial use.

A few families **restrict or exclude commercial use**, either outright or above a revenue threshold. They are marked in bold above. Presets for these models remain available for evaluation and research, and they are not recommended for production. If you need one of them commercially, obtain a license directly from its publisher.

## Bringing your own model

You are not limited to the families listed here. Any model in GGUF format that the inference runtime can load will work. Point a preset at your own model file or repository, or derive a new preset: see [Creating a preset from scratch](/llm/net/developer-reference/presets/creating-from-scratch/) and [Model source parameters](/llm/net/developer-reference/parameters/model-source/).

## Trademarks

The model and family names on this page are used solely to identify the models a preset targets. They are trademarks or trade names of their respective owners, and their use implies no affiliation with those owners and no endorsement by them. Llama is a trademark of Meta Platforms, Inc. Gemma is a trademark of Google LLC.

## Before you ship

{{% alert color="warning" %}}
**Check the license of the model you selected.** Aspose.LLM supplies the runtime, not the model. Whichever model you load, its terms come from the party that published it and they apply to your product. They are not part of, and are not covered by, your license agreement with Aspose Pty Ltd. The table above states the license of every family, but it is a summary verified on the date shown above the table: read the linked license itself before you deploy, and re-read it when you change the model a preset points at. See also [Supported presets](/llm/net/product-overview/supported-presets/) for the per-preset breakdown.
{{% /alert %}}

---
weight: 90
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/tensor-split/
feedback: LLMNET
version: 26.5.0
title: TensorSplit
description: Proportion of the model placed on each GPU in Aspose.LLM for .NET — balances load across unequal-VRAM GPUs.
keywords:
- TensorSplit
- multi-GPU
- VRAM balance
- unequal GPUs
---

`TensorSplit` is an array of floats, one per GPU, that controls the proportion of the model placed on each GPU during multi-GPU split. Values are normalized — `[2.0, 1.0]` means 2/3 on GPU 0 and 1/3 on GPU 1.

## Quick reference

| | |
|---|---|
| **Type** | `float[]?` |
| **Default** | `null` (equal distribution across GPUs) |
| **Range** | Array length = GPU count; values positive |
| **Category** | Multi-GPU configuration |
| **Field on** | `ModelInferenceParameters.TensorSplit` |

## What it does

When [`SplitMode`](/llm/net/developer-reference/parameters/model-inference/split-mode/) is `LAYER` or `ROW`, the engine distributes layers (or row blocks) across GPUs according to `TensorSplit`. Each GPU gets a share proportional to its entry in the array.

- `null` — equal distribution. Splits evenly regardless of VRAM.
- `[2.0, 1.0]` — 2:1 split. First GPU gets twice the share.
- `[1.0, 1.0, 1.0]` — explicit equal across 3 GPUs.
- `[3.0, 2.0, 1.0]` — 3:2:1 split across 3 GPUs (50 %, 33 %, 17 %).

The array length should match the number of GPUs visible to the process (after any `CUDA_VISIBLE_DEVICES` / `ROCR_VISIBLE_DEVICES` filtering).

## When to change it

| Scenario | Value |
|---|---|
| Single GPU | Not applicable |
| Multi-GPU, equal VRAM | `null` (equal default is correct) |
| Multi-GPU, unequal VRAM (24 GB + 12 GB) | `[2.0, 1.0]` |
| Multi-GPU with one GPU reserved for other work | Smaller share for that GPU |

## Example

```csharp
using Aspose.LLM.Abstractions.Parameters;

var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_LAYER;
preset.BaseModelInferenceParameters.GpuLayers = 999;
preset.BaseModelInferenceParameters.TensorSplit = new float[] { 2.0f, 1.0f };
// 2/3 of layers on GPU 0 (larger VRAM), 1/3 on GPU 1.

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`SplitMode`](/llm/net/developer-reference/parameters/model-inference/split-mode/) — `TensorSplit` applies only when mode is `LAYER` or `ROW`.
- [`GpuLayers`](/llm/net/developer-reference/parameters/model-inference/gpu-layers/) — total layers on GPUs are distributed per `TensorSplit`.
- [`MainGpu`](/llm/net/developer-reference/parameters/model-inference/main-gpu/) — ignored when `TensorSplit` is active.

## What's next

- [SplitMode](/llm/net/developer-reference/parameters/model-inference/split-mode/) — split strategy selector.
- [CUDA multi-GPU](/llm/net/developer-reference/acceleration/cuda/) — NVIDIA multi-GPU setup.
- [GPU deployment use case](/llm/net/use-cases/gpu-deployment-cuda/) — runnable example.

---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/split-mode/
feedback: LLMNET
version: 26.5.0
title: SplitMode
description: Multi-GPU model distribution in Aspose.LLM for .NET — None for single GPU, Layer for layer-wise split, Row for tensor parallelism.
keywords:
- SplitMode
- multi-GPU
- layer split
- tensor parallelism
- row split
---

`SplitMode` determines how the model is distributed across multiple GPUs. Essential only on multi-GPU hosts; on single-GPU systems, `None` is correct.

## Quick reference

| | |
|---|---|
| **Type** | `LlamaSplitMode?` enum |
| **Default** | `null` (use native default) |
| **Values** | `LLAMA_SPLIT_MODE_NONE`, `LLAMA_SPLIT_MODE_LAYER`, `LLAMA_SPLIT_MODE_ROW` |
| **Category** | GPU distribution |
| **Field on** | `ModelInferenceParameters.SplitMode` |

## What it does

| Value | Behavior |
|---|---|
| `LLAMA_SPLIT_MODE_NONE` (`0`) | Single GPU. Whole model on [`MainGpu`](/net/developer-reference/parameters/model-inference/main-gpu/). |
| `LLAMA_SPLIT_MODE_LAYER` (`1`) | Split layers across GPUs. KV cache follows layers. Good default for multi-GPU. |
| `LLAMA_SPLIT_MODE_ROW` (`2`) | Split layers and rows across GPUs. Uses tensor parallelism when supported. Fastest on high-bandwidth GPU interconnects (NVLink). |

On fully connected multi-GPU setups (NVLink, consumer PCIe with good topology), `ROW` is often fastest. On PCIe-only consumer setups, `LAYER` is safer.

## When to change it

| Scenario | Value |
|---|---|
| Single GPU | `LLAMA_SPLIT_MODE_NONE` (or `null`) |
| Multi-GPU default | `LLAMA_SPLIT_MODE_LAYER` |
| High-bandwidth multi-GPU (NVLink) | `LLAMA_SPLIT_MODE_ROW` |
| Testing multi-GPU setup | Start with `LAYER`, try `ROW` if stable |

## Example

```csharp
using Aspose.LLM.Abstractions.Parameters;

var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_LAYER;
preset.BaseModelInferenceParameters.GpuLayers = 999;
preset.BaseModelInferenceParameters.TensorSplit = new float[] { 2.0f, 1.0f };
// 2:1 split for unequal-VRAM GPUs (e.g., 24 GB + 12 GB).

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`MainGpu`](/net/developer-reference/parameters/model-inference/main-gpu/) — only applies when `SplitMode = None`.
- [`TensorSplit`](/net/developer-reference/parameters/model-inference/tensor-split/) — per-device allocation; applies to `LAYER` / `ROW`.
- [`GpuLayers`](/net/developer-reference/parameters/model-inference/gpu-layers/) — total layers on GPUs; distributed per split mode.
- HIP / Vulkan — support both split modes with varying driver maturity; test your specific setup.

## What's next

- [TensorSplit](/net/developer-reference/parameters/model-inference/tensor-split/) — fine-grained per-GPU ratios.
- [MainGpu](/net/developer-reference/parameters/model-inference/main-gpu/) — single-GPU device index.
- [CUDA acceleration](/net/developer-reference/acceleration/cuda/) — multi-GPU NVIDIA setup.

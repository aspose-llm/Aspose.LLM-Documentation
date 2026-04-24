---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/main-gpu/
feedback: LLMNET
version: 26.5.0
title: MainGpu
description: GPU device index in Aspose.LLM for .NET — selects which GPU holds the model when SplitMode is None.
keywords:
- MainGpu
- GPU index
- device selection
- CUDA_VISIBLE_DEVICES
---

`MainGpu` selects the GPU device index used when [`SplitMode`](/net/developer-reference/parameters/model-inference/split-mode/) is `None`. On multi-GPU hosts, this picks which device holds the entire model.

## Quick reference

| | |
|---|---|
| **Type** | `int?` |
| **Default** | `null` (use device 0) |
| **Range** | `0` to (GPU count - 1) |
| **Category** | GPU configuration |
| **Field on** | `ModelInferenceParameters.MainGpu` |

## What it does

- `null` or `0` — use GPU 0.
- `1`, `2`, etc. — use that GPU.

`MainGpu` is ignored when `SplitMode` is `LAYER` or `ROW` — those modes distribute the model across multiple GPUs without a single "main" device.

On single-GPU hosts, the field is effectively always 0.

## When to change it

| Scenario | Value |
|---|---|
| Single GPU or default | `null` or `0` |
| Multi-GPU, pin model to specific device | That device index |

Use the standard environment variables to constrain visibility globally:

- CUDA: `CUDA_VISIBLE_DEVICES=1` makes device 1 appear as device 0 to the process.
- HIP: `ROCR_VISIBLE_DEVICES=1` similarly.

## Example

```csharp
using Aspose.LLM.Abstractions.Parameters;

var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_NONE;
preset.BaseModelInferenceParameters.MainGpu = 1;  // use GPU 1
preset.BaseModelInferenceParameters.GpuLayers = 999;

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`SplitMode`](/net/developer-reference/parameters/model-inference/split-mode/) — `MainGpu` only applies when mode is `None`.
- [`GpuLayers`](/net/developer-reference/parameters/model-inference/gpu-layers/) — layers go to `MainGpu` when split is `None`.

## What's next

- [SplitMode](/net/developer-reference/parameters/model-inference/split-mode/) — multi-GPU distribution.
- [GpuLayers](/net/developer-reference/parameters/model-inference/gpu-layers/) — primary offload control.
- [CUDA acceleration](/net/developer-reference/acceleration/cuda/) — multi-GPU NVIDIA setup.

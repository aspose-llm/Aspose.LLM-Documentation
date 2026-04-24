---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/gpu-layers/
feedback: LLMNET
version: 26.5.0
title: GpuLayers
description: Number of model layers to offload to GPU in Aspose.LLM for .NET — 0 for CPU-only, 999 for full offload, partial for memory-tight GPUs.
keywords:
- GpuLayers
- GPU offload
- model layers
- VRAM
---

`GpuLayers` sets how many transformer layers run on GPU. Each offloaded layer lives in VRAM; the rest stay in system RAM. The primary knob for GPU acceleration.

## Quick reference

| | |
|---|---|
| **Type** | `int?` |
| **Default** | `null` (use native default) |
| **Range** | `0` = CPU only; `999` = full offload; partial values offload first N layers |
| **Category** | Load / offload |
| **Field on** | `ModelInferenceParameters.GpuLayers` |

## What it does

Each transformer layer either lives in VRAM (GPU inference for that layer) or in system RAM (CPU inference). The engine processes each token through all layers in sequence, using the respective backend at each step.

| Value | Behavior |
|---|---|
| `0` | CPU only. No VRAM allocated for weights. |
| `N` (where 1 ≤ N < model's layer count) | Partial offload. First N layers on GPU. |
| `≥ model's layer count` (idiomatic `999`) | Full offload. All layers on GPU. |
| `null` | Native default; usually matches "all layers" on GPU-capable builds. |

Partial offload is useful when the model doesn't fit entirely in VRAM. The transition between GPU and CPU layers costs some throughput but lets you run larger models than would otherwise fit.

## When to change it

| Scenario | Value |
|---|---|
| CPU-only inference | `0` |
| Full GPU offload (idiomatic) | `999` |
| 8B model on 12 GB GPU | Typically `24` – `32` (verify per model) |
| 70B model on 24 GB GPU | Partial; pair with quantization |
| Apple Silicon Metal | `999` (unified memory — no separate budget) |

Pair with a GPU-capable binary via [`BinaryManagerParameters.PreferredAcceleration`](/net/developer-reference/parameters/binary-manager/).

## Example

```csharp
using Aspose.LLM.Abstractions.Acceleration;

var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.GpuLayers = 999;  // full offload

using var api = AsposeLLMApi.Create(preset);
```

Partial offload for a memory-tight GPU:

```csharp
preset.BaseModelInferenceParameters.GpuLayers = 24;
preset.ContextParameters.OffloadKqv = false;  // keep KV on CPU to save more VRAM
```

## Interactions

- [`BinaryManagerParameters.PreferredAcceleration`](/net/developer-reference/parameters/binary-manager/) — must point at a GPU-capable backend.
- [`MainGpu`](/net/developer-reference/parameters/model-inference/main-gpu/) — which GPU (for single-GPU mode).
- [`SplitMode`](/net/developer-reference/parameters/model-inference/split-mode/) — how to split across multiple GPUs.
- [`TensorSplit`](/net/developer-reference/parameters/model-inference/tensor-split/) — per-GPU allocation ratios.
- [`OffloadKqv`](/net/developer-reference/parameters/context/offload-kqv/) — related, but for KV cache not weights.

## What's next

- [MainGpu](/net/developer-reference/parameters/model-inference/main-gpu/) — single-GPU selector.
- [SplitMode](/net/developer-reference/parameters/model-inference/split-mode/) — multi-GPU.
- [Acceleration overview](/net/developer-reference/acceleration/) — backend-specific setup.
- [GPU deployment use case](/net/use-cases/gpu-deployment-cuda/) — runnable example.

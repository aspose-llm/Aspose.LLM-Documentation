---
weight: 10
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/multimodal-context/use-gpu/
feedback: LLMNET
version: 26.5.0
title: UseGpu
description: Offload the vision projector to GPU in Aspose.LLM for .NET — null defers to mtmd's auto-detection; explicit true/false forces a choice.
keywords:
- UseGpu
- vision projector
- mmproj
- GPU offload
- multimodal
---

`UseGpu` controls whether the `mtmd` layer runs the vision projector on GPU. Relevant only for vision presets; ignored on text-only.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (use native default — typically true if a GPU is present) |
| **Category** | Multimodal context |
| **Field on** | `MultimodalContextParameters.UseGpu` |

## What it does

- `null` — delegate to `mtmd`'s auto-detection. Usually correct.
- `true` — force GPU. Requires a supported GPU.
- `false` — force CPU. Keeps VRAM for the base model on memory-tight GPUs.

The projector is typically small (200 MB – 2 GB). GPU offload is fast on modest hardware; the main reason to keep it on CPU is freeing VRAM for the base model's KV cache.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Keep all VRAM for the base model | `false` |
| Force projector to GPU | `true` |

## Example

```csharp
var preset = new Qwen3VL2BPreset();
preset.MtmdContextParameters.UseGpu = false;           // projector on CPU
preset.BaseModelInferenceParameters.GpuLayers = 999;   // base model fully on GPU
```

## Interactions

- [`GpuLayers`](/net/developer-reference/parameters/model-inference/gpu-layers/) — base model offload; projector placement is independent.
- [`PrintTimings`](/net/developer-reference/parameters/multimodal-context/print-timings/) — diagnose projector performance.

## What's next

- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — projector diagnostics.
- [Vision presets](/net/developer-reference/multimodal/vision-presets/) — built-in presets.
- [Multimodal context hub](/net/developer-reference/parameters/multimodal-context/) — all mtmd knobs.

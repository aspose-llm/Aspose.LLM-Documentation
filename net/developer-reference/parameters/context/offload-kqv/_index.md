---
weight: 220
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/offload-kqv/
feedback: LLMNET
version: 26.5.0
title: OffloadKqv
description: Offload the KQV (attention) computation and KV cache to GPU in Aspose.LLM for .NET — true by default on GPU builds.
keywords:
- OffloadKqv
- KQV
- attention
- GPU offload
- KV cache
---

`OffloadKqv` controls whether the KQV (attention) computation and the KV cache itself live on GPU memory. On GPU-enabled builds, the default is to offload; disable only when fighting for VRAM.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (uses native default — typically `true` on GPU builds) |
| **Category** | KV cache / GPU |
| **Field on** | `ContextParameters.OffloadKqv` |

## What it does

- `true` — KV cache tensors and attention computation live on the GPU. Benefits throughput; uses more VRAM.
- `false` — KV cache stays on CPU even when layers are offloaded via [`GpuLayers`](/net/developer-reference/parameters/model-inference/gpu-layers/). Reduces VRAM usage; slower because GPU must read KV from host memory.
- `null` — native default (usually `true` on GPU builds, irrelevant on CPU builds).

Disabling is useful on GPUs where the weights fit but the KV cache would push you into OOM at long contexts.

## When to change it

| Scenario | Value |
|---|---|
| Default GPU inference | `null` (true) |
| Short on VRAM at long context — trade speed for memory | `false` |
| CPU-only | `null` (irrelevant) |

## Example

```csharp
var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.GpuLayers = 999;    // full offload
preset.ContextParameters.OffloadKqv = false;            // but keep KV on CPU to save VRAM
preset.ContextParameters.ContextSize = 131072;          // long context
```

## Interactions

- [`GpuLayers`](/net/developer-reference/parameters/model-inference/gpu-layers/) — with `OffloadKqv = false`, GPU layers access KV from CPU — slower but saves VRAM.
- [`TypeK`](/net/developer-reference/parameters/context/type-k/), [`TypeV`](/net/developer-reference/parameters/context/type-v/) — quantizing KV reduces memory regardless of placement.
- [`FlashAttentionMode`](/net/developer-reference/parameters/context/flash-attention-mode/) — FA reduces KV memory pressure.

## What's next

- [TypeK](/net/developer-reference/parameters/context/type-k/), [TypeV](/net/developer-reference/parameters/context/type-v/) — KV dtype.
- [GpuLayers](/net/developer-reference/parameters/model-inference/gpu-layers/) — weight offload.
- [Out of memory troubleshooting](/net/troubleshooting/out-of-memory/) — memory pressure recipes.

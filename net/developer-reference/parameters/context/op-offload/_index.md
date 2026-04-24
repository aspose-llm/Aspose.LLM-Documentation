---
weight: 260
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/op-offload/
feedback: LLMNET
version: 26.5.0
title: OpOffload
description: Offload host tensor operations to device in Aspose.LLM for .NET — supplementary to GpuLayers; leave at default unless instructed.
keywords:
- OpOffload
- tensor operations
- GPU
- offload
---

`OpOffload` toggles offloading of host-side tensor operations to the GPU device. This is supplementary to [`GpuLayers`](/net/developer-reference/parameters/model-inference/gpu-layers/) and affects specific small operations that would otherwise run on CPU even with GPU offload active.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (use native default) |
| **Category** | GPU offload (auxiliary) |
| **Field on** | `ContextParameters.OpOffload` |

## What it does

Some tensor operations (embedding lookups, small reductions) are relatively cheap and traditionally run on the host. `OpOffload` lets the engine offload them to the device too, in exchange for minimal host-device overhead.

- `null` — native default. Modern GPU backends usually benefit from `true`.
- `true` — offload.
- `false` — keep on host.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Benchmarking GPU-centric paths | `true` |
| Debugging device-specific issues | `false` |

## Example

```csharp
var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.GpuLayers = 999;
preset.ContextParameters.OpOffload = true;  // ensure all operations on device
```

## Interactions

- [`GpuLayers`](/net/developer-reference/parameters/model-inference/gpu-layers/) — primary offload control.
- [`OffloadKqv`](/net/developer-reference/parameters/context/offload-kqv/) — KV specific.

## What's next

- [GpuLayers](/net/developer-reference/parameters/model-inference/gpu-layers/) — primary layer offload.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/acceleration/metal/
feedback: LLMNET
version: 26.5.0
title: Metal
description: Run Aspose.LLM for .NET on Apple Silicon with Metal acceleration — unified memory, GPU offload, and Apple M-series performance notes.
keywords:
- Metal
- Apple Silicon
- M1
- M2
- M3
- M4
- unified memory
- macOS
---

Metal is Aspose.LLM for .NET's preferred backend on Apple Silicon Macs (M1, M2, M3, M4). It leverages the unified memory architecture — the CPU and GPU share the same physical RAM — so there is no separate VRAM budget to worry about.

## Requirements

- **Hardware**: Apple Silicon Mac (M-series chip). Intel Macs are not supported.
- **OS**: macOS 11 (Big Sur) or later.
- **No driver install**: Metal ships with macOS; nothing to configure.

## Select Metal

```csharp
using Aspose.LLM.Abstractions.Acceleration;

var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.Metal;
preset.BaseModelInferenceParameters.GpuLayers = 999;

using var api = AsposeLLMApi.Create(preset);
```

On first run, the SDK downloads the Metal variant of `llama.cpp` binaries (typically 100-200 MB). Subsequent runs use the cache.

On Apple Silicon with `PreferredAcceleration = null`, auto-detection picks Metal by default — the explicit setting is redundant but harmless.

## Unified memory

Because RAM and GPU memory are the same physical chip, `GpuLayers = 999` does not create a separate VRAM claim — it just tells the Metal backend to run the layers on the GPU compute units. The total memory footprint is the same whether you run on CPU or Metal, but Metal is significantly faster for matrix math.

You do not need to worry about "VRAM fits" calculations on Apple Silicon. If the model plus KV cache fit in system RAM, they fit for Metal too.

Typical memory ceilings per Mac:

| Mac model | Unified memory options |
|---|---|
| M1 / M2 / M3 base | 8 GB, 16 GB |
| M1 / M2 / M3 Pro | 16 GB, 32 GB |
| M1 / M2 / M3 Max | 32 GB, 64 GB, 96 GB |
| M2 / M3 Ultra | 64 GB, 128 GB, 192 GB |
| M4 / M4 Pro / M4 Max | 16 GB up to 128 GB |

A 7B Q4_K_M model needs ~8-12 GB including KV cache — comfortable on any 16 GB Mac. A 70B Q4 needs 40+ GB and realistically wants an M2/M3 Ultra or M4 Max with 64 GB or more.

## Single-chip only

Multi-GPU is not applicable on Apple Silicon. `MainGpu`, `SplitMode`, and `TensorSplit` have no effect — leave them at defaults.

## Performance tips

- **Full offload** — `GpuLayers = 999` always. Partial offload is rarely useful because there is no separate VRAM budget.
- **Flash Attention** — `ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled`. Metal implements flash attention kernels efficiently.
- **Prefer smaller quantizations** — Apple Silicon's memory bandwidth, not compute, is often the bottleneck. Q4 models run noticeably faster than Q8 on the same hardware for this reason.

## Power management

Metal respects macOS power policy. On battery, macOS may throttle GPU clocks and reduce inference speed. Plug into AC for sustained throughput when benchmarking or running long jobs.

## Common issues

| Symptom | Likely cause | Fix |
|---|---|---|
| "This backend is not supported" on Intel Mac | Intel hardware. | Use CPU acceleration instead; upgrade to Apple Silicon for GPU. |
| Slower than expected | macOS throttling on battery. | Connect to AC power; close GPU-heavy apps. |
| Out-of-memory | Combined model + KV cache exceeds system RAM. | Smaller preset, lower context size, quantize KV cache. |

## What's next

- [Model inference parameters](/net/developer-reference/parameters/model-inference/) — `GpuLayers` configuration.
- [Context parameters](/net/developer-reference/parameters/context/) — flash attention and KV cache settings.
- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — `PreferredAcceleration`.

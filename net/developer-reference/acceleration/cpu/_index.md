---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/acceleration/cpu/
feedback: LLMNET
version: 26.5.0
title: CPU
description: Run Aspose.LLM for .NET on CPU only — AVX512, AVX2, and NoAVX variants, threading, and performance expectations.
keywords:
- CPU
- AVX2
- AVX512
- AVX
- NoAVX
- threading
- NThreads
---

CPU is the fallback backend for Aspose.LLM for .NET when no supported GPU is available. It works on every supported platform and requires no special drivers or runtimes. Throughput is lower than GPU inference, but reasonable for small models (3B-7B parameters) on modern CPUs.

## Requirements

- **CPU**: any x64 CPU. ARM64 is supported on Linux and macOS (Apple Silicon). The SDK picks the optimal CPU variant automatically based on detected instruction sets.
- **OS**: Windows 10+, Linux (glibc 2.28+), macOS 11+. Intel Macs use CPU; Apple Silicon Macs prefer [Metal](/net/developer-reference/acceleration/metal/).

## CPU variants

The SDK downloads one of four CPU variants depending on your CPU's instruction sets:

| Variant | Requires | Typical CPUs |
|---|---|---|
| `AVX512` | AVX-512 instructions | Intel Xeon Scalable 2nd+, recent Core (Cannon Lake+), AMD Zen 4 |
| `AVX2` (default fallback) | AVX2 instructions | Most CPUs from 2014 onwards |
| `AVX` | AVX instructions | Sandy Bridge / Ivy Bridge era (2011-2013) |
| `NoAVX` | x64 baseline | Pre-2011 CPUs (very slow) |

Auto-detection picks the highest available variant. To force a specific variant:

```csharp
using Aspose.LLM.Abstractions.Acceleration;

preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.AVX2; // or AVX512, AVX, NoAVX
preset.BaseModelInferenceParameters.GpuLayers = 0;
```

CPU download size: typically 80-150 MB.

## Thread configuration

On CPU, threading is the primary lever for throughput. Two settings:

- **`ContextParameters.NThreads`** — threads for generation (token-by-token decode). Typically `half of ProcessorCount`.
- **`ContextParameters.NThreadsBatch`** — threads for prompt processing (initial tokenization and context fill). Typically `all ProcessorCount`.

```csharp
preset.ContextParameters.NThreads = 8;
preset.ContextParameters.NThreadsBatch = 16;
```

When `NThreads` is `null`, the engine falls back to `EngineParameters.DefaultThreads` (default: `ProcessorCount - 1`).

### Why different thread counts

- **Prompt processing** is embarrassingly parallel — more threads help.
- **Generation** is sequential at the token level and bound by memory bandwidth. Beyond a certain point (often around 8-12 threads), adding threads does not help and sometimes slows generation due to NUMA or cache contention.

Benchmark on your specific hardware for the best numbers. Start with `NThreads = ProcessorCount / 2` and `NThreadsBatch = ProcessorCount`, then tune.

## Performance expectations

Rough tokens-per-second for a 7B Q4_K_M model on CPU only:

| CPU class | Approximate t/s |
|---:|---:|
| Old x64 (2015) | 1-3 |
| Modern Core i5 / Ryzen 5 | 5-10 |
| Modern Core i7 / Ryzen 7 / Xeon | 8-15 |
| High-end Core i9 / Threadripper / Xeon Platinum | 12-25 |

These are ballpark; real numbers depend on clock speed, memory bandwidth, AVX level, and how busy the CPU is with other work.

For sustained CPU inference, expect 5-15 t/s on mainstream desktop CPUs. That is acceptable for chat UIs and batch jobs, but slow for real-time applications.

## Memory

Model weights plus KV cache live in system RAM. Typical memory for a 7B Q4_K_M model with 32K context: 8-12 GB. See [System requirements](/net/system-requirements/#memory) for per-preset estimates.

## Performance tips

- **Use AVX512 when available** — on Zen 4 or recent Xeon, AVX512 is 20-40 % faster than AVX2 for the same model.
- **Flash Attention** — `ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled` helps on long contexts even on CPU.
- **Quantize the KV cache** — `TypeV = GgmlType.Q8_0` halves V-cache memory with minimal quality impact.
- **Smaller models first** — 3B models (Llama 3.2, Phi 4 mini) run 2-3× faster than 7B models on the same CPU.
- **Disable other CPU-heavy work** during inference. CPU throughput is sensitive to contention.

## Common issues

| Symptom | Likely cause | Fix |
|---|---|---|
| Very slow inference | Using `NoAVX` or `AVX` variant on a modern CPU. | Verify `PreferredAcceleration` auto-detection; force `AVX2` or `AVX512` explicitly. |
| High CPU usage, low throughput | Too many threads — contention. | Reduce `NThreads`; try `NThreads = ProcessorCount / 2`. |
| Stalls between tokens | Memory pressure or page thrashing. | Check memory; enable `UseMemoryMapping = true`. |
| Crashes on `AVX512` variant | CPU reports AVX-512 but has a bug in legacy modes. | Drop to `AVX2`. |

## What's next

- [Model inference parameters](/net/developer-reference/parameters/model-inference/) — `GpuLayers = 0` pattern for CPU-only.
- [Context parameters](/net/developer-reference/parameters/context/) — `NThreads` and `NThreadsBatch`.
- [Metal](/net/developer-reference/acceleration/metal/) — macOS-preferred alternative on Apple Silicon.

---
weight: 60
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/binary-manager/preferred-acceleration/
feedback: LLMNET
version: 26.5.0
title: PreferredAcceleration
description: Force a specific acceleration backend in Aspose.LLM for .NET — null auto-detects; set to CUDA, HIP, Metal, Vulkan, or a CPU variant to pin.
keywords:
- PreferredAcceleration
- AccelerationType
- CUDA
- Vulkan
- CPU fallback
---

`PreferredAcceleration` forces the SDK to download a specific acceleration variant of the native binaries. Set it to override auto-detection.

## Quick reference

| | |
|---|---|
| **Type** | `AccelerationType?` enum |
| **Default** | `null` (auto-detect best available) |
| **Values** | `None`, `CUDA`, `HIP`, `Metal`, `Vulkan`, `Kompute`, `OpenCL`, `SYCL`, `AVX512`, `AVX2`, `AVX`, `OpenBLAS`, `NoAVX` |
| **Category** | Native binary source |
| **Field on** | `BinaryManagerParameters.PreferredAcceleration` |

## What it does

At binary download time, the manager picks an asset matching your host and `PreferredAcceleration`. When `null`, the auto-detection priority is: CUDA > HIP > Metal > Vulkan > best CPU AVX level.

- `null` (default) — auto-detect.
- `CUDA` — NVIDIA GPUs (Windows / Linux).
- `HIP` — AMD GPUs on Linux.
- `Metal` — Apple Silicon.
- `Vulkan` — cross-vendor GPU; Windows AMD users; Intel iGPUs.
- `AVX2` / `AVX512` — force CPU at a specific instruction level.
- `NoAVX` — legacy x64 without AVX.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| CPU-only despite having a GPU | `AVX2` or `AVX512` |
| Force CUDA on a CUDA-capable host | `CUDA` |
| Windows + AMD GPU | `Vulkan` (HIP is Linux-only) |
| Cross-vendor single-codepath deployment | `Vulkan` |
| Force a specific AVX level to test performance | `AVX2` vs `AVX512` |

## Example

```csharp
using Aspose.LLM.Abstractions.Acceleration;

var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.GpuLayers = 999;

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`GpuLayers`](/net/developer-reference/parameters/model-inference/gpu-layers/) — pair a GPU acceleration with a non-zero `GpuLayers` to actually use the GPU.
- [`SystemSpecification`](/net/developer-reference/parameters/binary-manager/system-specification/) — lower-level override; `PreferredAcceleration` is the recommended path.

## What's next

- [Acceleration overview](/net/developer-reference/acceleration/) — per-backend setup.
- [Supported acceleration](/net/product-overview/supported-acceleration/) — platform × backend matrix.
- [GPU not detected troubleshooting](/net/troubleshooting/gpu-not-detected/) — when auto-detection falls back to CPU.

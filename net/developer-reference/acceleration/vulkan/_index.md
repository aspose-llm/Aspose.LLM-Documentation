---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/acceleration/vulkan/
feedback: LLMNET
version: 26.5.0
title: Vulkan
description: Run Aspose.LLM for .NET with Vulkan acceleration — cross-platform, cross-vendor GPU backend that works on NVIDIA, AMD, and Intel hardware without vendor-specific runtimes.
keywords:
- Vulkan
- GPU
- cross-platform
- NVIDIA
- AMD
- Intel
- integrated GPU
---

Vulkan is a cross-vendor GPU backend that works on most modern discrete and integrated GPUs. It does not need CUDA or ROCm installed — only a recent graphics driver with Vulkan support. Use it when CUDA or HIP are unavailable, or when you want one codepath that works across NVIDIA, AMD, and Intel hardware.

## Requirements

- **GPU**: any GPU with Vulkan 1.2 or later support. This covers most discrete GPUs from 2018+ and integrated GPUs from Intel (Xe, UHD), AMD (RDNA, Vega), and NVIDIA (Maxwell+).
- **Driver**:
  - NVIDIA: 470 or later.
  - AMD: modern Adrenalin or Mesa RADV on Linux.
  - Intel: current Arc / Xe driver.
- **OS**: Windows 10+ or Linux (glibc 2.28+). Not supported on macOS — Metal is the macOS equivalent.

Verify Vulkan support:

- **Windows**: install Vulkan SDK's `vulkaninfoSDK.exe` or run a Vulkan demo app.
- **Linux**: `vulkaninfo` from the `vulkan-tools` package.

## Select Vulkan

```csharp
using Aspose.LLM.Abstractions.Acceleration;

var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.Vulkan;
preset.BaseModelInferenceParameters.GpuLayers = 999;

using var api = AsposeLLMApi.Create(preset);
```

The SDK downloads the Vulkan variant (typically 200-400 MB) on first run.

## When to prefer Vulkan

- **Cross-vendor deployments** — the same binary works on NVIDIA, AMD, and Intel GPUs without swapping backends.
- **Integrated GPUs** — Intel Xe, AMD integrated Radeon, and NVIDIA integrated GPUs work via Vulkan, no vendor-specific runtime.
- **Containers without CUDA/ROCm** — Vulkan runs without NVIDIA Container Toolkit or ROCm Docker setup.
- **Older hardware** — many GPUs that no longer get CUDA updates still have Vulkan drivers.

## When to prefer CUDA or HIP instead

For dedicated NVIDIA or AMD workloads, vendor-specific backends are usually faster:

- [CUDA](/net/developer-reference/acceleration/cuda/) is typically 20-40 % faster than Vulkan on NVIDIA.
- [HIP](/net/developer-reference/acceleration/hip-rocm/) is faster than Vulkan on AMD for most workloads.

Use Vulkan when portability trumps raw speed.

## Multi-GPU

Vulkan supports multi-GPU setups via `SplitMode` and `TensorSplit` like CUDA, but driver support for multi-device Vulkan is less mature. Test on your specific hardware before committing — single-GPU Vulkan is well-trodden; multi-GPU Vulkan is hit-or-miss.

```csharp
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_LAYER;
preset.BaseModelInferenceParameters.GpuLayers = 999;
// Optional TensorSplit for unequal GPU sizes.
```

## Performance tips

- **Flash Attention** — supported on most Vulkan drivers; enable via `ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled`.
- **Integrated GPU memory** — iGPUs use system RAM for GPU memory. Short contexts and heavy KV quantization help fit within the shared pool.
- **Driver version matters** — recent Vulkan drivers have significant perf improvements for LLM inference. Keep drivers current.

## Common issues

| Symptom | Likely cause | Fix |
|---|---|---|
| Vulkan binary downloaded but runs on CPU | `GpuLayers = 0`, or Vulkan driver missing. | Set `GpuLayers = 999`; verify Vulkan with `vulkaninfo`. |
| Crash at model load | Very old driver or unsupported GPU. | Update driver; fall back to CPU if hardware is pre-Vulkan-1.2. |
| Significantly slower than CUDA on NVIDIA | Expected — use CUDA instead for best performance. | Switch `PreferredAcceleration` to `CUDA`. |

## What's next

- [CUDA](/net/developer-reference/acceleration/cuda/) — faster on NVIDIA.
- [HIP / ROCm](/net/developer-reference/acceleration/hip-rocm/) — faster on AMD.
- [CPU](/net/developer-reference/acceleration/cpu/) — last-resort fallback.
- [Model inference parameters](/net/developer-reference/parameters/model-inference/) — GPU offload configuration.

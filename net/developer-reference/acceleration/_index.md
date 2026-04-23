---
weight: 55
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/acceleration/
feedback: LLMNET
version: 26.5.0
title: Acceleration
description: Hardware acceleration backends supported by Aspose.LLM for .NET — CUDA, HIP (ROCm), Metal, Vulkan, and CPU variants. How auto-detection works and how to force a specific backend.
keywords:
- acceleration
- CUDA
- HIP
- ROCm
- Metal
- Vulkan
- CPU
- GPU
- AVX
---

Aspose.LLM for .NET wraps `llama.cpp` and uses its native binaries for every inference. Those binaries ship per-platform, per-acceleration variants — one for each GPU backend, plus CPU variants at different AVX levels. The SDK downloads the matching variant on first use and caches it locally.

You do not configure backends at compile time — the choice is made at runtime by `BinaryManager`. You can let the SDK auto-detect the best option for your host, or force a specific backend via `BinaryManagerParameters.PreferredAcceleration`.

## Supported backends

| Backend | Platforms | GPU vendor | Typical priority |
|---|---|---|---|
| [CUDA](/net/developer-reference/acceleration/cuda/) | Windows, Linux | NVIDIA | Highest on NVIDIA hosts |
| [HIP / ROCm](/net/developer-reference/acceleration/hip-rocm/) | Linux | AMD | High on AMD hosts |
| [Metal](/net/developer-reference/acceleration/metal/) | macOS (Apple Silicon) | Apple | Highest on Apple Silicon |
| [Vulkan](/net/developer-reference/acceleration/vulkan/) | Windows, Linux | Any Vulkan-capable GPU | Fallback GPU; cross-vendor |
| [CPU](/net/developer-reference/acceleration/cpu/) | All | — | Fallback when no GPU is available |

## Auto-detection

When `BinaryManagerParameters.PreferredAcceleration` is `null` (the default), the SDK picks the best available backend for your host in this priority order:

1. **CUDA** — if an NVIDIA GPU with a recent driver is present.
2. **HIP** — if a ROCm-capable AMD GPU is present.
3. **Metal** — on Apple Silicon.
4. **Vulkan** — if a Vulkan-capable GPU is present.
5. **CPU** — with the highest AVX level available (`AVX512 > AVX2 > AVX > NoAVX`).

The detection runs during `AsposeLLMApi.Create`, before the native binary download. The result is reflected in the downloaded asset's name.

## Forcing a backend

Set `BinaryManagerParameters.PreferredAcceleration` to an `AccelerationType` value:

```csharp
using Aspose.LLM.Abstractions.Acceleration;

var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.GpuLayers = 999;

using var api = AsposeLLMApi.Create(preset);
```

The full enum:

```csharp
public enum AccelerationType
{
    None,
    CUDA,
    HIP,
    Metal,
    Vulkan,
    Kompute,
    OpenCL,
    SYCL,
    AVX512,
    AVX2,
    AVX,
    OpenBLAS,
    NoAVX,
}
```

`Kompute`, `OpenCL`, `SYCL`, and `OpenBLAS` are included for completeness — verify availability for your target; they are less common.

## Matching GPU offload

`PreferredAcceleration` chooses the **binary**. `BaseModelInferenceParameters.GpuLayers` chooses how many layers run on the GPU. Both settings need to agree:

| PreferredAcceleration | Recommended GpuLayers |
|---|---|
| CUDA, HIP, Metal, Vulkan | `999` (full offload) or a partial count fitting VRAM |
| AVX512, AVX2, AVX, NoAVX | `0` (CPU only) |

Setting a GPU binary with `GpuLayers = 0` works but wastes the GPU — the model runs on CPU anyway. Setting a CPU binary with `GpuLayers = 999` silently keeps the model on CPU since there is no GPU runtime available.

## Memory by backend

GPU inference needs VRAM for model weights (proportional to `GpuLayers`), plus the KV cache (proportional to `ContextSize × layers on GPU`), plus intermediate buffers.

| Backend | Memory location | Constraint |
|---|---|---|
| CUDA | VRAM on the chosen NVIDIA GPU (`MainGpu` + `TensorSplit`) | GPU memory |
| HIP | VRAM on the chosen AMD GPU | GPU memory |
| Metal | Unified memory (RAM/VRAM shared on Apple Silicon) | System RAM |
| Vulkan | VRAM on chosen GPU | GPU memory |
| CPU | System RAM | System RAM |

See [System requirements](/net/system-requirements/#memory) for per-preset memory ranges.

## First-run download size

Each acceleration variant is a different archive on the GitHub release. Sizes are approximate and vary by release:

| Backend | Typical download |
|---|---|
| CUDA | 400-800 MB |
| HIP | 300-500 MB |
| Metal | 100-200 MB |
| Vulkan | 200-400 MB |
| CPU (AVX2/AVX512) | 80-150 MB |

Once downloaded, the variant is cached at `BinaryManagerParameters.BinaryPath`. Changing `PreferredAcceleration` triggers a new download on next `Create`.

## What's next

- [CUDA](/net/developer-reference/acceleration/cuda/) — NVIDIA GPUs.
- [HIP / ROCm](/net/developer-reference/acceleration/hip-rocm/) — AMD GPUs.
- [Metal](/net/developer-reference/acceleration/metal/) — Apple Silicon.
- [Vulkan](/net/developer-reference/acceleration/vulkan/) — cross-platform GPU.
- [CPU](/net/developer-reference/acceleration/cpu/) — when no GPU is available.
- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — `PreferredAcceleration` and friends.

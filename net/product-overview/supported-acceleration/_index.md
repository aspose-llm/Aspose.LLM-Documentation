---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/product-overview/supported-acceleration/
feedback: LLMNET
version: 26.5.0
title: Supported acceleration
description: Hardware acceleration backends supported by Aspose.LLM for .NET — CUDA, HIP, Metal, Vulkan, and CPU — with a platform × backend matrix.
keywords:
- acceleration
- CUDA
- HIP
- Metal
- Vulkan
- CPU
- GPU
- platform support
---

Aspose.LLM for .NET ships native `llama.cpp` binaries for five acceleration backends. At first `AsposeLLMApi.Create`, `BinaryManager` picks the matching variant for your platform and downloads it from GitHub. You can also force a specific backend via `BinaryManagerParameters.PreferredAcceleration`.

This page is an at-a-glance sheet for planning deployments. For per-backend setup (drivers, `GpuLayers`, multi-GPU split), see the detail pages.

## Platform × backend matrix

| Backend | Windows | Linux | macOS | Typical use |
|---|:---:|:---:|:---:|---|
| [**CUDA**](/net/developer-reference/acceleration/cuda/) | ✅ | ✅ | ❌ | NVIDIA GPUs. Highest throughput on modern nVidia cards. |
| [**HIP / ROCm**](/net/developer-reference/acceleration/hip-rocm/) | ❌ | ✅ | ❌ | AMD Instinct and RDNA 3 Radeon cards. |
| [**Metal**](/net/developer-reference/acceleration/metal/) | ❌ | ❌ | ✅ | Apple Silicon (M1/M2/M3/M4). |
| [**Vulkan**](/net/developer-reference/acceleration/vulkan/) | ✅ | ✅ | ❌ | Cross-vendor GPU fallback. NVIDIA, AMD, Intel. Windows AMD users. |
| [**CPU**](/net/developer-reference/acceleration/cpu/) | ✅ | ✅ | ✅ | No GPU required. AVX512/AVX2/AVX/NoAVX variants. |

## Auto-detection

When `PreferredAcceleration` is `null` (default), the SDK picks in this order:

1. **CUDA** — if an NVIDIA GPU with driver 525+ is present.
2. **HIP** — if a ROCm-capable AMD GPU is present (Linux only).
3. **Metal** — on Apple Silicon.
4. **Vulkan** — any Vulkan-capable GPU.
5. **CPU** — highest AVX level available (`AVX512 > AVX2 > AVX > NoAVX`).

Override by setting `BinaryManagerParameters.PreferredAcceleration`. See [Binary manager parameters](/net/developer-reference/parameters/binary-manager/).

## First-run download sizes

Binaries are downloaded once per preset's `ReleaseTag` and cached locally.

| Backend | Typical download |
|---|---|
| CUDA | 400-800 MB |
| HIP | 300-500 MB |
| Vulkan | 200-400 MB |
| Metal | 100-200 MB |
| CPU (AVX2/AVX512) | 80-150 MB |

## Picking a backend

| Scenario | Try |
|---|---|
| NVIDIA host (dev or server) | CUDA (best throughput) |
| AMD host on Linux with ROCm | HIP |
| AMD host on Windows | Vulkan |
| Apple Silicon Mac | Metal (auto-detected) |
| Intel iGPU | Vulkan |
| No GPU or GPU not supported | CPU |
| Cross-vendor deployments from one codepath | Vulkan |

## What's next

- [CUDA](/net/developer-reference/acceleration/cuda/), [HIP / ROCm](/net/developer-reference/acceleration/hip-rocm/), [Metal](/net/developer-reference/acceleration/metal/), [Vulkan](/net/developer-reference/acceleration/vulkan/), [CPU](/net/developer-reference/acceleration/cpu/) — per-backend setup.
- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — how `PreferredAcceleration` and `BinaryPath` work.
- [System requirements](/net/system-requirements/) — OS / driver / memory prerequisites.
- [GPU not detected](/net/troubleshooting/gpu-not-detected/) — common pitfalls.

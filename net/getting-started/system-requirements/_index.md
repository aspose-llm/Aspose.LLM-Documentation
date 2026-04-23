---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/system-requirements/
feedback: LLMNET
version: 26.5.0
title: System requirements
description: Supported .NET runtimes, operating systems, CPU and GPU requirements, and disk/RAM sizing for Aspose.LLM for .NET.
keywords:
- requirements
- .NET
- .NET 8
- .NET 10
- Windows
- Linux
- macOS
- GPU
- CUDA
- Metal
- RAM
---

Plan the .NET runtime, operating system, and hardware for your Aspose.LLM for .NET deployment. Requirements depend on the preset you pick: small models run on modest hardware, while 20B-parameter models need substantial RAM and preferably GPU acceleration.

## Supported .NET runtimes

Aspose.LLM targets **.NET Standard 2.0**. It runs on any .NET runtime that supports .NET Standard 2.0, including:

- **.NET 10** (current release)
- **.NET 8 LTS** (recommended for new projects)
- **.NET 6** and later
- **.NET Framework 4.7.2** and later

New projects should use **.NET 8 LTS** or **.NET 10**. All examples in this documentation assume `dotnet` CLI tooling and modern .NET runtime features.

.NET Framework 4.7.2 is the minimum Framework version the SDK is validated against. Earlier Framework versions may work because the SDK builds against .NET Standard 2.0, but they are not tested and not supported.

## Supported operating systems

| OS | Versions | Notes |
|---|---|---|
| Windows | 10 version 1809 and later, Windows Server 2019+ | x64 and ARM64 |
| Linux | Debian 10+, Ubuntu 20.04+, RHEL/Alma/Rocky 8+ (glibc 2.28+) | x64 and ARM64; other distributions with compatible glibc work |
| macOS | 11 (Big Sur) and later | Intel x64 and Apple Silicon (arm64); Metal acceleration on Apple Silicon |

## CPU

A modern x64 CPU with AVX2 support runs any preset in CPU-only mode. `BinaryManager` selects an AVX-level-specific native binary at runtime:

- **AVX512** — fastest; requires a recent Intel / AMD CPU.
- **AVX2** — the default fallback on most modern CPUs.
- **No-AVX** — available for older CPUs and compatibility scenarios; inference is slow.

ARM64 CPUs are supported on Linux and macOS via native ARM binaries.

## GPU (optional)

GPU acceleration is optional but strongly recommended for models larger than 3B parameters. Supported backends:

| Backend | Platforms | Requires |
|---|---|---|
| **CUDA** | Windows, Linux | NVIDIA driver 525+, CUDA-capable GPU (compute capability 5.0+) |
| **HIP / ROCm** | Linux | AMD ROCm 6+, supported AMD GPU |
| **Metal** | macOS (Apple Silicon) | Apple Silicon M1 or later |
| **Vulkan** | Windows, Linux | Vulkan 1.2+ driver, GPU with Vulkan support |

Select a backend via `BinaryManagerParameters.PreferredAcceleration` on the preset. When unset, the SDK picks the best available backend for your system. Control the amount of GPU offload with `BaseModelInferenceParameters.GpuLayers`.

## Memory

RAM requirements scale with model size, quantization, and context size. The table below lists typical working-set sizes for the preset's default quantization and context:

| Preset example | Model size | Default context | RAM (CPU) / VRAM (GPU) |
|---|---|---:|---|
| `Llama32Preset` | 3B Q4_K_M | 131 072 | 6-8 GB |
| `Phi4Preset` | Mini Q4_K_M | 16 384 | 4-6 GB |
| `Qwen25Preset` | 7B Q4_K_M | 32 768 | 8-12 GB |
| `Qwen3Preset` | 8B Q4_K_M | 32 768 | 8-12 GB |
| `DeepseekR1Qwen3Preset` | 8B Q4_K_M | 131 072 | 12-16 GB |
| `DeepSeekCoder2Preset` | MoE IQ3_M | 163 840 | 12-20 GB |
| `Oss20Preset` | 20B Q4_K_M | 131 072 | 16-24 GB |
| `Qwen25VL3BPreset` | VL 3B UD-IQ2_XXS | 128 000 | 4-6 GB (plus mmproj) |
| `Ministral3VisionPreset` | 8B Q4_K_M | 262 144 | 12-20 GB (plus mmproj) |

Ranges assume default sampler and batch settings. Actual usage varies with the number of active sessions, the `CacheCleanupStrategy`, and KV cache quantization (`ContextParameters.TypeK` / `TypeV`).

For running the full default context on very long sessions, add 1-4 GB for the KV cache on top of the model weight size.

## Disk

Two distinct caches:

| Cache | Typical size | Location |
|---|---|---|
| Model files (downloaded from Hugging Face) | 2-15 GB per model | `EngineParameters.ModelCachePath` (default: per-user cache folder) |
| Native `llama.cpp` binaries | 100-500 MB per acceleration variant | `BinaryManagerParameters.BinaryPath` (default: per-user cache folder) |

Plan for at least 20-30 GB of free disk space when experimenting with multiple presets.

## Network

Internet access is required on **first** use:

- `BinaryManager` fetches native binaries from `github.com/ggml-org/llama.cpp/releases`.
- `ModelManager` downloads model files (and `mmproj` for vision presets) from `huggingface.co`.

Subsequent runs use the local caches. For offline or firewalled environments, pre-download both caches and point the preset's `BinaryPath` and `ModelCachePath` at them — details will be covered in the offline deployment use case.

## Development

Any IDE or editor with .NET support works:

- **Visual Studio** 2022 (v17.8+) — full experience on Windows.
- **JetBrains Rider** — full experience cross-platform.
- **VS Code** with the C# Dev Kit — light-weight cross-platform option.
- **`dotnet` CLI** alone — works for build, run, publish.

## What's next

- [Installation](/net/installation/) — add the NuGet package to your project.
- [Licensing](/net/licensing/) — apply a license before running inference.
- [Hello, world!](/net/hello-world/) — first runnable example.

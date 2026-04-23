---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/acceleration/hip-rocm/
feedback: LLMNET
version: 26.5.0
title: HIP / ROCm
description: Run Aspose.LLM for .NET on AMD GPUs with HIP (ROCm) acceleration — supported GPUs, driver requirements, and tuning notes.
keywords:
- HIP
- ROCm
- AMD
- GPU
- Instinct
- Radeon
---

HIP (and its underlying ROCm stack) is the AMD-specific GPU backend for Aspose.LLM for .NET. It is Linux-only and targeted at AMD Instinct data-center GPUs and recent Radeon consumer cards.

## Requirements

- **GPU**: ROCm-supported AMD GPU.
  - **Instinct**: MI100, MI210, MI250, MI300 series.
  - **Radeon**: RDNA 3 (RX 7900 series) and newer are officially supported. Some RDNA 2 cards (RX 6800/6900) work with `HSA_OVERRIDE_GFX_VERSION` workarounds.
- **OS**: Linux with ROCm 6.x. Ubuntu 22.04 LTS and RHEL 9 are the commonly tested hosts.
- **Driver**: install the ROCm stack via the official AMD packages. Verify with `rocminfo`.
- **No Windows support**: AMD has Windows ROCm in preview, but Aspose.LLM's HIP binaries currently target Linux only. On Windows with AMD GPUs, use [Vulkan](/net/developer-reference/acceleration/vulkan/) instead.

Verify ROCm:

```bash
rocminfo | grep "Name:"
```

The output should list your GPU by name (e.g., `gfx1100` for RX 7900 XTX).

## Select HIP

```csharp
using Aspose.LLM.Abstractions.Acceleration;

var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.HIP;
preset.BaseModelInferenceParameters.GpuLayers = 999;

using var api = AsposeLLMApi.Create(preset);
```

The SDK downloads the HIP variant (typically 300-500 MB) on first run.

## Multi-GPU

HIP supports multi-GPU across AMD cards of the same ROCm generation.

```csharp
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_LAYER;
preset.BaseModelInferenceParameters.GpuLayers = 999;
// Optionally tune TensorSplit per-GPU VRAM.
```

Use `ROCR_VISIBLE_DEVICES` to control which GPUs the process sees.

## RDNA 2 workaround

Some RDNA 2 cards (e.g., RX 6800 XT, RX 6900 XT) are not officially supported by ROCm, but work with a GFX version override:

```bash
export HSA_OVERRIDE_GFX_VERSION=10.3.0
dotnet run
```

Apply this before starting your process. The override tricks ROCm into treating an RX 6800/6900 as a supported variant. Quality is fine; performance is slightly lower than on supported cards.

## Performance tips

- **Flash Attention** — supported on recent ROCm releases; enable via `ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled`.
- **Partial offload** — for consumer Radeon cards with limited VRAM (16-24 GB), tune `GpuLayers` just below full to leave room for KV cache.
- **KV quantization** — aggressive `TypeV = GgmlType.Q8_0` or even `Q4_0` claws back meaningful VRAM on long contexts.

## Windows AMD users: use Vulkan

Aspose.LLM does not ship Windows HIP binaries. If your AMD GPU is on Windows, use [Vulkan](/net/developer-reference/acceleration/vulkan/) — AMD's Vulkan driver is excellent and gets you GPU acceleration without ROCm.

## Common issues

| Symptom | Likely cause | Fix |
|---|---|---|
| `rocblas_status_internal_error` on load | Incompatible ROCm version. | Match ROCm 6.x; upgrade if older. |
| Unsupported GPU at startup | Card not on ROCm's support list. | Use Vulkan as fallback, or try `HSA_OVERRIDE_GFX_VERSION`. |
| Inference on CPU despite HIP binary | `GpuLayers = 0` or ROCm runtime missing. | Set `GpuLayers = 999`; verify with `rocminfo`. |
| Multi-GPU instability | Mixing different RDNA generations. | Stick to same-generation GPUs; try `LLAMA_SPLIT_MODE_LAYER`. |

## What's next

- [Vulkan](/net/developer-reference/acceleration/vulkan/) — Windows alternative for AMD.
- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — `PreferredAcceleration` selection.
- [Model inference parameters](/net/developer-reference/parameters/model-inference/) — GPU offload and multi-GPU settings.

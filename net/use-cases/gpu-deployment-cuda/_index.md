---
weight: 110
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/gpu-deployment-cuda/
feedback: LLMNET
version: 26.5.0
title: GPU deployment with CUDA
description: Deploy Aspose.LLM for .NET on NVIDIA GPUs — select CUDA, offload all layers, configure multi-GPU split, and size VRAM for model + KV cache.
keywords:
- CUDA
- NVIDIA
- GPU
- multi-GPU
- TensorSplit
- GpuLayers
- VRAM
---

CUDA is the fastest backend for Aspose.LLM for .NET on NVIDIA hardware. This use case walks through a production GPU deployment — single GPU, multi-GPU, and VRAM sizing.

## When to use this pattern

- Production host with NVIDIA GPU.
- You want maximum throughput for a given model size.
- Multi-GPU server where you need to split a large model.
- Cloud VM with CUDA-capable instance (e.g., g5, p3/p4).

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).
- NVIDIA GPU with compute capability 5.0+.
- NVIDIA driver 525 or later.
- [System requirements](/net/system-requirements/) → GPU section for the full matrix.

Verify with `nvidia-smi`:

```bash
nvidia-smi
# Check Driver Version >= 525 and note GPU model + VRAM.
```

## Single GPU, full offload

The common case: one GPU, whole model on it.

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Acceleration;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.GpuLayers = 999; // full offload

using var api = AsposeLLMApi.Create(preset);

string reply = await api.SendMessageAsync("Explain Kubernetes in one paragraph.");
Console.WriteLine(reply);
```

`GpuLayers = 999` is idiomatic for "put everything on the GPU". The engine caps the value at the model's actual layer count.

## Single GPU, partial offload

For a model that does not quite fit in VRAM, offload the first N layers and keep the rest on CPU. Find the sweet spot by benchmarking.

```csharp
var preset = new Qwen3Preset(); // 8B, 32 layers
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;

// First 28 layers on GPU, last 4 on CPU.
preset.BaseModelInferenceParameters.GpuLayers = 28;

using var api = AsposeLLMApi.Create(preset);
```

The KV cache also consumes VRAM proportional to GPU-resident layers. Leave ~1-2 GB of VRAM headroom.

## Multi-GPU split

Two or more NVIDIA GPUs — distribute the model across them.

```csharp
using Aspose.LLM.Abstractions.Parameters;

var preset = new Oss20Preset(); // 20B — typically needs 16-24 GB

preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_LAYER;
preset.BaseModelInferenceParameters.GpuLayers = 999;

// For equal-VRAM GPUs, leave TensorSplit null (equal split).
// For unequal GPUs, balance by VRAM:
preset.BaseModelInferenceParameters.TensorSplit = new float[] { 1.0f, 1.0f }; // 50/50

using var api = AsposeLLMApi.Create(preset);
```

Split modes:

| Mode | Good for |
|---|---|
| `LLAMA_SPLIT_MODE_NONE` | Single GPU; all on `MainGpu`. |
| `LLAMA_SPLIT_MODE_LAYER` | General multi-GPU. Splits layers across devices. |
| `LLAMA_SPLIT_MODE_ROW` | Multi-GPU with high-bandwidth interconnect (NVLink). Best throughput with tensor parallelism. |

## Unequal VRAM

If the cards differ (e.g., 24 GB RTX 4090 + 12 GB RTX 3080), bias the split toward the larger:

```csharp
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_LAYER;
preset.BaseModelInferenceParameters.TensorSplit = new float[] { 2.0f, 1.0f };
// 2:1 proportion — values normalized.
```

## Select a specific GPU

`MainGpu` picks the device when split mode is `None`. On hosts with multiple GPUs where you only want one for inference:

```csharp
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_NONE;
preset.BaseModelInferenceParameters.MainGpu = 1; // index 1
preset.BaseModelInferenceParameters.GpuLayers = 999;
```

Or use the standard NVIDIA env var to hide GPUs from the process:

```bash
CUDA_VISIBLE_DEVICES=1 dotnet run
```

## VRAM sizing

VRAM budget breakdown:

- Model weights (scales with quantization and layer count).
- KV cache (scales with `ContextSize × layers on GPU`).
- Intermediate buffers (50-500 MB).

Example: Qwen 3 8B Q4_K_M at 32K context on full GPU offload:

- Weights: ~5 GB
- KV cache (32K tokens × 32 layers, F16 by default): ~4 GB
- Buffers: ~300 MB
- Total: ~9-10 GB → fits on a 12 GB card comfortably

To cut KV cache memory, quantize the V tensor:

```csharp
preset.ContextParameters.TypeV = GgmlType.Q8_0; // half V-cache size
```

## Full example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Acceleration;
using Aspose.LLM.Abstractions.Models;
using Aspose.LLM.Abstractions.Parameters;
using Aspose.LLM.Abstractions.Parameters.Presets;

internal class CudaDemo
{
    public static async Task Main()
    {
        var license = new Aspose.LLM.License();
        license.SetLicense("Aspose.LLM.lic");

        var preset = new Qwen3Preset();

        // CUDA backend.
        preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;

        // Full GPU offload.
        preset.BaseModelInferenceParameters.GpuLayers = 999;
        preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_NONE;

        // Save VRAM on long sessions.
        preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
        preset.ContextParameters.TypeV = GgmlType.Q8_0;

        using var api = AsposeLLMApi.Create(preset);

        var sw = System.Diagnostics.Stopwatch.StartNew();
        string reply = await api.SendMessageAsync(
            "Write a 200-word overview of microservices architecture trade-offs.");
        sw.Stop();

        Console.WriteLine(reply);
        Console.WriteLine($"\nGenerated in {sw.Elapsed.TotalSeconds:F1}s ({reply.Length} chars).");
    }
}
```

## Throughput expectations

Full-offload tokens-per-second for a 7B Q4_K_M model at 32K context:

| GPU | t/s |
|---|---:|
| RTX 3060 (12 GB) | 40-60 |
| RTX 3090 / 4070 | 60-90 |
| RTX 4080 / A6000 | 80-120 |
| RTX 4090 | 100-140 |
| A100 40 GB | 120-160 |
| H100 | 180-250+ |

Numbers vary with batch size, context depth, driver version, and prompt length.

## Cloud deployment

Common CUDA-capable cloud instances:

| Provider | Instance type | GPU |
|---|---|---|
| AWS | `g5.xlarge` up | A10G |
| AWS | `p3`, `p4` | V100, A100 |
| Azure | `NC-series` | T4, A100 |
| GCP | `A2`, `G2` | A100, L4 |

Ensure the image ships with NVIDIA driver 525+ and CUDA runtime. Most GPU AMIs are current; for custom base images, install `nvidia-driver-525` (or later) before deploying.

## Common issues

- **CUDA binary downloaded, inference runs on CPU** — `GpuLayers = 0` or not set. Set to `999`.
- **`cudaErrorInsufficientDriver`** — upgrade driver to 525+.
- **Out-of-memory at load** — partial offload or smaller preset.
- **Slower multi-GPU than expected** — switch from `LAYER` to `ROW` if GPUs have NVLink; otherwise `LAYER` is usually best.

## What's next

- [CUDA acceleration](/net/developer-reference/acceleration/cuda/) — full CUDA reference.
- [Model inference parameters](/net/developer-reference/parameters/model-inference/) — `GpuLayers`, `TensorSplit`, `SplitMode`, `MainGpu`.
- [Context parameters](/net/developer-reference/parameters/context/) — flash attention and KV dtype.

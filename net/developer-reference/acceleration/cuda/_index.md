---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/acceleration/cuda/
feedback: LLMNET
version: 26.5.0
title: CUDA
description: Run Aspose.LLM for .NET on NVIDIA GPUs with CUDA acceleration — driver requirements, compute capability, single and multi-GPU setup.
keywords:
- CUDA
- NVIDIA
- GPU
- driver
- compute capability
- TensorSplit
- MainGpu
---

CUDA is the fastest backend for Aspose.LLM for .NET on NVIDIA GPUs. It supports single-GPU and multi-GPU setups, aggressive memory offload, and the full `llama.cpp` feature set.

## Requirements

- **GPU**: NVIDIA with compute capability 5.0 or higher.
- **Driver**: version 525 or later.
- **CUDA runtime**: the one bundled with the downloaded native binary (typically CUDA 11.7 or 12.x). You do **not** install CUDA separately — the SDK's binary ships with the runtime.
- **OS**: Windows 10+ or Linux (glibc 2.28+). Not supported on macOS.

Verify driver and GPU with `nvidia-smi`:

```bash
nvidia-smi
```

Check the `Driver Version` (≥ 525) and the GPU model. Every modern GeForce, Quadro, Tesla, or data-center GPU qualifies.

## Select CUDA

```csharp
using Aspose.LLM.Abstractions.Acceleration;

var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.GpuLayers = 999; // full offload

using var api = AsposeLLMApi.Create(preset);
```

On the first run, the SDK downloads the CUDA variant of `llama.cpp` binaries (typically 400-800 MB) and caches it at `BinaryManagerParameters.BinaryPath`.

## Single GPU

With one GPU, the defaults work. The engine places the offloaded layers on GPU 0.

```csharp
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.GpuLayers = 999;
```

## Specific GPU selection

On hosts with multiple GPUs, `MainGpu` picks which one gets the entire model when split mode is `None`.

```csharp
using Aspose.LLM.Abstractions.Parameters;

preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_NONE;
preset.BaseModelInferenceParameters.MainGpu = 1;    // use GPU index 1
preset.BaseModelInferenceParameters.GpuLayers = 999;
```

Use `CUDA_VISIBLE_DEVICES` environment variable to constrain which GPUs the process sees — standard NVIDIA tooling.

## Multi-GPU split

Distribute the model across multiple GPUs.

```csharp
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_LAYER;
preset.BaseModelInferenceParameters.GpuLayers = 999;
```

Split modes:

| Mode | Behavior |
|---|---|
| `LLAMA_SPLIT_MODE_NONE` | Single GPU only. Whole model on `MainGpu`. |
| `LLAMA_SPLIT_MODE_LAYER` | Split layers across GPUs. Good default for multi-GPU. |
| `LLAMA_SPLIT_MODE_ROW` | Split rows — enables tensor parallelism where supported. Fastest on setups with high-bandwidth GPU interconnects (NVLink). |

For unequal GPU memory sizes, set `TensorSplit` to balance the load:

```csharp
// 24 GB GPU + 12 GB GPU: 2:1 split.
preset.BaseModelInferenceParameters.TensorSplit = new float[] { 2.0f, 1.0f };
```

Values are normalized to sum to 1.

## Partial offload on memory-tight GPUs

If the model does not fit entirely in VRAM, offload the first N layers and leave the rest on CPU.

```csharp
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.GpuLayers = 28;  // first 28 layers on GPU
```

Benchmark to find the right split — "offload until VRAM is ~1-2 GB short of full" — because the KV cache also claims VRAM proportional to GPU layer count.

## Memory tips

- **KV cache quantization** — set `ContextParameters.TypeV = GgmlType.Q8_0` to halve V-cache memory with minor quality impact.
- **Flash Attention** — `ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled` reduces memory at long contexts.
- **Shorter context** — drop `ContextParameters.ContextSize` if you do not need the preset's default length.

## Common issues

| Symptom | Likely cause | Fix |
|---|---|---|
| `cudaErrorInsufficientDriver` in logs | Driver too old. | Upgrade NVIDIA driver to 525+. |
| CUDA binary downloaded but inference is on CPU | `GpuLayers = 0` or not set. | Set `GpuLayers = 999`. |
| Out-of-memory on model load | VRAM too small for full offload. | Lower `GpuLayers`, enable flash attention, quantize KV. |
| Slower than expected on multi-GPU | `SplitMode = None`. | Switch to `LAYER` or `ROW` depending on interconnect. |

## What's next

- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — `PreferredAcceleration`.
- [Model inference parameters](/net/developer-reference/parameters/model-inference/) — `GpuLayers`, `SplitMode`, `TensorSplit`, `MainGpu`.
- [Vulkan](/net/developer-reference/acceleration/vulkan/) — cross-vendor GPU alternative.

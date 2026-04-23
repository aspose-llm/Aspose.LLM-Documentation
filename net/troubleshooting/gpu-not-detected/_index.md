---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/troubleshooting/gpu-not-detected/
feedback: LLMNET
version: 26.5.0
title: GPU not detected
description: Aspose.LLM for .NET runs on CPU when you expected GPU — diagnose driver, acceleration selection, binary mismatch, GpuLayers setting.
keywords:
- GPU not detected
- CPU fallback
- CUDA
- HIP
- Metal
- driver
- GpuLayers
---

You wanted the SDK to use the GPU, but inference is slow and `nvidia-smi` (or equivalent) shows no activity from the process. This page walks through the detection pipeline and common misconfigurations.

## Symptom

- Inference runs at CPU speed (5-15 tokens/sec) despite a GPU being present.
- `nvidia-smi` does not list the Aspose.LLM process.
- `rocm-smi` shows zero utilization.
- Logs do not mention CUDA / HIP / Metal / Vulkan initialization.

## Cause

The SDK picks a backend in two stages:

1. **`BinaryManager`** downloads a native binary matching `BinaryManagerParameters.PreferredAcceleration` (or auto-detection). The binary dictates what GPU APIs are available.
2. **`Engine`** respects `BaseModelInferenceParameters.GpuLayers` — if `0`, the model stays on CPU even if the binary supports GPU.

Either stage can silently fall back to CPU.

## Resolution

### 1. Check the downloaded binary

Enable debug logging and look for the binary selection line in logs:

```
[BinaryManager] resolved asset: llama-b8816-bin-win-cuda-cu12.4-x64.zip
```

If the asset name says `cpu` or does not mention CUDA/HIP/Metal/Vulkan, the `BinaryManager` did not detect the GPU. Fix by forcing:

```csharp
using Aspose.LLM.Abstractions.Acceleration;

preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
```

Then clear the binary cache at `BinaryManagerParameters.BinaryPath` and re-run to download the GPU variant.

### 2. Check the driver

On Linux / Windows with NVIDIA:

```bash
nvidia-smi
# Must show Driver Version >= 525 and the GPU model.
```

If `nvidia-smi` does not find the GPU, the driver is not installed or the GPU is not accessible (containerized environment without `--gpus all` flag, or the host has no NVIDIA GPU).

On Linux with AMD:

```bash
rocminfo
# Must list your GPU under Agent information.
```

On macOS:

```bash
system_profiler SPDisplaysDataType | grep "Chipset Model"
# Must show Apple M-series for Metal support.
```

### 3. Verify `GpuLayers`

Even with the right binary, `GpuLayers = 0` forces CPU. Set it explicitly:

```csharp
preset.BaseModelInferenceParameters.GpuLayers = 999;
```

`999` is the idiomatic "full offload". The engine caps at the model's actual layer count.

### 4. Check for conflicting environment variables

NVIDIA environment variables can hide GPUs from the process:

```bash
echo $CUDA_VISIBLE_DEVICES
# If set to empty or -1, no GPU is visible.
```

Unset or set to `0` (or a valid GPU index):

```bash
unset CUDA_VISIBLE_DEVICES
# or
export CUDA_VISIBLE_DEVICES=0
```

For HIP: `ROCR_VISIBLE_DEVICES` and `HIP_VISIBLE_DEVICES` play the same role.

### 5. Container / WSL2 specifics

**Docker**: you must start containers with `--gpus all` (NVIDIA) or `--device=/dev/kfd --device=/dev/dri` (AMD ROCm). Without these flags, the container has no GPU access.

**WSL2** on Windows: install NVIDIA driver on the Windows side; install CUDA inside WSL following NVIDIA's WSL2 guide. Older Windows + WSL combinations do not support CUDA in WSL — upgrade Windows 11 and WSL.

### 6. Fall back to Vulkan

If CUDA / HIP setup is impractical (custom kernels, container limitations), try Vulkan:

```csharp
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.Vulkan;
preset.BaseModelInferenceParameters.GpuLayers = 999;
```

Vulkan runs on NVIDIA, AMD, and Intel GPUs with standard drivers. Performance is 20-40 % below CUDA but better than CPU.

### 7. Windows users with AMD — use Vulkan

Aspose.LLM does not ship HIP binaries for Windows. On Windows with AMD, Vulkan is the only GPU path.

## Prevention

- During deployment, assert GPU is active with a small probe:

  ```csharp
  // After Create, a short inference should be fast on GPU.
  var sw = System.Diagnostics.Stopwatch.StartNew();
  string reply = await api.SendMessageAsync("Say ok.");
  sw.Stop();
  if (sw.Elapsed.TotalSeconds > 2)
      _logger.LogWarning("Inference is slow - GPU may not be active.");
  ```

- Log the chosen acceleration at startup for auditability.
- Monitor GPU utilization in production (Datadog, Prometheus) to catch silent CPU fallback.

## What's next

- [Acceleration](/net/developer-reference/acceleration/) — detailed per-backend setup.
- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — `PreferredAcceleration`.
- [Model inference parameters](/net/developer-reference/parameters/model-inference/) — `GpuLayers`, `SplitMode`, `MainGpu`.

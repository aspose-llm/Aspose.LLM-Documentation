---
weight: 70
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/troubleshooting/performance-issues/
feedback: LLMNET
version: 26.5.0
title: Performance issues
description: Diagnose slow throughput, high first-token latency, and performance spikes in Aspose.LLM for .NET.
keywords:
- performance
- throughput
- latency
- slow
- first token
- TTFT
- thread contention
---

The SDK loads and runs, but throughput is below expectations or first-token latency is too high. This page covers the levers that affect performance.

## Symptom

- Fewer tokens per second than expected for your hardware.
- First-token latency of several seconds even after warm-up.
- Performance spikes — fast for a while, then slow.
- Occasional stalls mid-response.

## Cause

- Wrong acceleration backend or silent CPU fallback.
- Suboptimal threading configuration.
- Flash attention not enabled.
- Memory pressure (swap thrashing, KV cache too large for VRAM).
- Competing CPU-heavy processes on the same host.
- Context size too large for the hardware.
- First request of a new session (fresh prefill).

## Resolution

### 1. Confirm the right backend

Enable debug logging and confirm the binary variant and acceleration:

```
[BinaryManager] resolved asset: llama-b8816-bin-win-cuda-cu12.4-x64.zip
[Engine] inference on CUDA with 32/32 layers offloaded
```

If the variant says `cpu` while you have a GPU — see [GPU not detected](/net/troubleshooting/gpu-not-detected/).

### 2. Verify `GpuLayers`

Make sure `GpuLayers` is high enough to offload the model:

```csharp
preset.BaseModelInferenceParameters.GpuLayers = 999;
```

Partial offload (e.g., `GpuLayers = 20`) on an 8B model keeps half on CPU — the GPU cannot accelerate what is not on it.

### 3. Enable flash attention

Near-universal win; rarely hurts:

```csharp
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
```

Particularly important for contexts beyond 8K tokens.

### 4. Tune threads on CPU

For CPU inference, `NThreads` and `NThreadsBatch` matter:

```csharp
preset.ContextParameters.NThreads = Environment.ProcessorCount / 2;
preset.ContextParameters.NThreadsBatch = Environment.ProcessorCount;
```

See [CPU acceleration](/net/developer-reference/acceleration/cpu/) for the full rationale. Benchmark on your specific host to find the sweet spot; adding threads past 8-12 on generation often hurts.

### 5. Warm up sessions

First token on a fresh session includes prefill time (tokenizing and evaluating the system prompt + history). Amortize by reusing sessions:

```csharp
// Reuse one session per user instead of creating fresh each request.
```

See [Reduce first-token latency](/net/how-to/reduce-first-token-latency/).

### 6. Shrink `ContextSize` if you do not use the full window

Longer contexts are slower per token even when mostly empty — KV scans scale with position count. Drop `ContextSize` to the actual max you need:

```csharp
preset.ContextParameters.ContextSize = 8192;
```

### 7. Check for competing CPU load

Another CPU-heavy process on the same host steals throughput:

- `top` / `htop` on Linux/macOS.
- Task Manager → Details on Windows.

Serialize inference work with other CPU-heavy tasks; do not run AV scans, backup jobs, or compilers alongside.

### 8. Watch for memory pressure

Swap thrashing silently destroys throughput:

```bash
free -h
# Check "swap used". Nonzero during inference = bad.
```

If the host is swapping, reduce memory footprint (smaller model, shorter context, KV quantization).

### 9. Check for thermal throttling

Sustained high load heats the CPU and GPU; thermal throttling drops clocks and cuts throughput.

- On laptops, plug into AC power.
- Verify cooling — clean dust, check fan RPM.
- On CPU: `watch -n 1 'cat /proc/cpuinfo | grep MHz'` (Linux).
- On NVIDIA GPU: `nvidia-smi -q -d CLOCK` (look for `Current`-vs-`Base` clock).

### 10. Mirostat / dynatemp overhead

Advanced samplers like Mirostat and dynamic temperature add small per-token overhead. If you are chasing the last 10 % of throughput, disable them:

```csharp
preset.SamplerParameters.Mirostat = 0;
preset.SamplerParameters.DynatempRange = 0;
```

## Measure

Before optimizing, establish a baseline:

```csharp
var sw = System.Diagnostics.Stopwatch.StartNew();
string reply = await api.SendMessageAsync(prompt);
sw.Stop();

int approxTokens = reply.Split(' ').Length * 4 / 3; // rough conversion
double tps = approxTokens / sw.Elapsed.TotalSeconds;
Console.WriteLine($"~{tps:F1} tok/s");
```

Run with a representative prompt size; numbers vary wildly by prompt length.

## Reference throughput numbers

7B Q4_K_M at 4K context:

| Hardware | Expected |
|---|---|
| CPU (i5, AVX2) | 5-10 t/s |
| CPU (i7/i9, AVX-512) | 8-15 t/s |
| RTX 3060 (CUDA) | 40-60 t/s |
| RTX 4090 (CUDA) | 100-140 t/s |
| Apple M2 Pro (Metal) | 30-50 t/s |
| Apple M3 Max (Metal) | 50-80 t/s |

If your numbers are substantially below these, work through the resolution steps in order.

## What's next

- [Tune for speed vs quality](/net/how-to/tune-for-speed-vs-quality/) — speed-biased configuration.
- [Reduce first-token latency](/net/how-to/reduce-first-token-latency/) — cut TTFT.
- [Acceleration](/net/developer-reference/acceleration/) — backend-specific tuning.
- [Context parameters](/net/developer-reference/parameters/context/) — batch sizes and threading.

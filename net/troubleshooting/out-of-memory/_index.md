---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/troubleshooting/out-of-memory/
feedback: LLMNET
version: 26.5.0
title: Out of memory
description: Fix out-of-memory errors in Aspose.LLM for .NET — GPU VRAM, system RAM, KV cache growth, swap thrashing.
keywords:
- out of memory
- OOM
- VRAM
- RAM
- KV cache
- memory allocation
---

Out-of-memory failures happen at model load, during long sessions, or when running several models on the same host. The remedy depends on which memory pool is exhausted.

## Symptom

- At `Create`: `InvalidOperationException` mentioning memory allocation, `cudaErrorOutOfMemory`, or `rocblas` memory errors.
- During generation: unexpectedly slow responses (swap thrashing on Linux/macOS).
- On Windows: a `System.OutOfMemoryException` or the process being killed.
- On GPU: `nvidia-smi` showing the process using close to or exceeding the card's VRAM before the failure.

## Cause

- Model weights plus KV cache exceed the available memory pool.
- KV cache grows as sessions accumulate — each active session claims a slice of `ContextSize`.
- Multiple sessions, long prompts, and long responses compound.
- On Apple Silicon (unified memory), system RAM is the shared ceiling.

## Resolution

### 1. Identify which pool ran out

| What you see | Ran out |
|---|---|
| `cudaErrorOutOfMemory`, `hipErrorOutOfMemory` | GPU VRAM |
| `System.OutOfMemoryException`, Linux OOM killer, kernel panic | System RAM |
| Swap activity (`free -h` shows swap in use), very slow generation | RAM exhausted, OS paging |

### 2. Quick wins

Apply these changes and re-test.

**Smaller context:**

```csharp
preset.ContextParameters.ContextSize = 4096; // down from 32K or 131K
```

**Quantize V cache:**

```csharp
preset.ContextParameters.TypeV = GgmlType.Q8_0;
```

**Enable flash attention:**

```csharp
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
```

**Partial GPU offload:**

```csharp
// Offload some layers, leave rest on CPU.
preset.BaseModelInferenceParameters.GpuLayers = 20;
```

See [Low memory tuning](/net/use-cases/low-memory-tuning/) for the full recipe.

### 3. Switch to a smaller preset

If quick wins do not help, step down to a smaller model:

| From | To | Savings |
|---|---|---|
| `Qwen25Preset` (7B) | `Llama32Preset` (3B) | ~3-4 GB |
| `Oss20Preset` (20B) | `Qwen25Preset` (7B) | ~6-8 GB |
| Any F16/Q8 preset | Q4_K_M equivalent | 50 % |

### 4. Cap concurrent sessions

In multi-user hosts, cap the active session count. A back-of-envelope budget:

```
max_sessions = (available_memory - model_weights - overhead) / per_session_kv_budget
```

Use [Estimate memory requirements](/net/how-to/estimate-memory-requirements/) for concrete numbers.

Evict idle sessions by periodically disposing `AsposeLLMApi` and recreating it. The current SDK does not provide an explicit per-session evict API.

### 5. Recycle the API instance

In long-running hosts, KV cache and native buffers can grow beyond predictions. Periodically:

1. Dispose the current `AsposeLLMApi`.
2. Wait for native memory to release.
3. Create a new instance.

Expect a 5-30 second restart cost on warm caches.

### 6. On unified memory (Apple Silicon)

There is no separate VRAM to optimize — everything is RAM. Apply system-RAM reductions: smaller model, shorter context, KV quantization.

## Prevention

- Measure peak memory during load tests. Budget against the measured peak, not theoretical estimates.
- Run with `EnableDebugLogging = true` in staging and watch `[KV]` lines to track cache growth.
- Size the host for your expected session concurrency at your chosen preset — do not size for the minimum case.

## What's next

- [Low memory tuning](/net/use-cases/low-memory-tuning/) — recipes for memory-constrained hosts.
- [Estimate memory requirements](/net/how-to/estimate-memory-requirements/) — predictive sizing.
- [Context parameters](/net/developer-reference/parameters/context/) — KV cache dtype and flash attention.

---
weight: 100
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/cpu-only-deployment/
feedback: LLMNET
version: 26.5.0
title: CPU-only deployment
description: Run Aspose.LLM for .NET without a GPU — pick a small preset, tune threads, and set realistic performance expectations.
keywords:
- CPU only
- no GPU
- AVX2
- AVX512
- threading
- NThreads
- cpu deployment
---

CPU-only deployments trade inference speed for simplicity: no GPU drivers, no CUDA or ROCm runtimes, no hardware-specific dependencies. With the right preset and thread tuning, a modern desktop or server CPU delivers reasonable throughput for chat-style workloads.

## When to use this pattern

- The host has no GPU, or no GPU supported by Aspose.LLM.
- Container or serverless environments where GPU access is unavailable.
- Cost-sensitive deployments where GPU instances are overkill.
- Reproducible environments where you want to avoid GPU driver variance.

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).
- A CPU with AVX2 support (most CPUs from 2014+).

## Pick a preset sized for CPU

Small and mid-size presets run well on CPU. Large models (20B+) are usable but slow.

| Preset | Model size | Expected t/s on modern CPU |
|---|---|---:|
| `Llama32Preset` | 3B Q4_K_M | 10-20 |
| `Phi4Preset` | Mini Q4_K_M | 10-18 |
| `Qwen25Preset` | 7B Q4_K_M | 5-12 |
| `Qwen3Preset` | 8B Q4_K_M | 5-10 |
| `Oss20Preset` | 20B Q4_K_M | 2-5 |

Estimates assume 8-core modern CPU with AVX2. AVX-512 adds 20-40 %.

## Configure CPU-only

Force CPU execution with two settings on the preset:

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Acceleration;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Llama32Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.AVX2;
preset.BaseModelInferenceParameters.GpuLayers = 0;

using var api = AsposeLLMApi.Create(preset);
```

`AVX2` is a safe default; use `AVX512` when you know the CPU supports it (Intel Xeon Scalable 2nd+, AMD Zen 4+, recent Intel Core).

## Tune threads

Two knobs:

- `ContextParameters.NThreads` — threads for generation (token-by-token decode). Typically half of `ProcessorCount`.
- `ContextParameters.NThreadsBatch` — threads for prompt processing. Typically all cores.

```csharp
preset.ContextParameters.NThreads = 6;        // generation
preset.ContextParameters.NThreadsBatch = 12;  // prompt ingestion
```

Why different counts:

- Prompt processing is embarrassingly parallel — more threads help.
- Generation is sequential at the token level and bound by memory bandwidth. Adding threads past 8-12 rarely helps and sometimes hurts.

Benchmark on your target hardware. Start with `NThreads = ProcessorCount / 2`, `NThreadsBatch = ProcessorCount`, then adjust.

## Save memory

CPU inference uses system RAM for everything — model weights, KV cache, intermediate buffers. Several levers:

```csharp
// Shorter context — less KV memory.
preset.ContextParameters.ContextSize = 4096;

// Smaller KV dtype — halves V-cache memory.
preset.ContextParameters.TypeV = GgmlType.Q8_0;

// Memory mapping avoids doubling RAM use during load.
preset.BaseModelInferenceParameters.UseMemoryMapping = true;
```

Budget: a 7B Q4_K_M model with 8K context needs ~6-8 GB. Use 16 GB+ of RAM for comfort.

## Full example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Acceleration;
using Aspose.LLM.Abstractions.Models;
using Aspose.LLM.Abstractions.Parameters.Presets;

internal class CpuOnlyDemo
{
    public static async Task Main()
    {
        var license = new Aspose.LLM.License();
        license.SetLicense("Aspose.LLM.lic");

        var preset = new Llama32Preset();

        // Force CPU.
        preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.AVX2;
        preset.BaseModelInferenceParameters.GpuLayers = 0;
        preset.BaseModelInferenceParameters.UseMemoryMapping = true;

        // Threads.
        preset.ContextParameters.NThreads = Math.Max(1, Environment.ProcessorCount / 2);
        preset.ContextParameters.NThreadsBatch = Environment.ProcessorCount;

        // Save memory.
        preset.ContextParameters.ContextSize = 8192;
        preset.ContextParameters.TypeV = GgmlType.Q8_0;

        using var api = AsposeLLMApi.Create(preset);

        string[] questions =
        {
            "What is the capital of Portugal?",
            "Name three Portuguese dishes.",
            "How long is the flight from London to Lisbon?",
        };

        foreach (string q in questions)
        {
            var sw = System.Diagnostics.Stopwatch.StartNew();
            string reply = await api.SendMessageAsync(q);
            sw.Stop();

            Console.WriteLine($"Q: {q}");
            Console.WriteLine($"A: {reply}");
            Console.WriteLine($"(took {sw.Elapsed.TotalSeconds:F1}s)");
            Console.WriteLine();
        }
    }
}
```

## Performance expectations

Tokens-per-second ranges are ballpark; your numbers depend on CPU clock, memory speed, AVX level, and background load:

| Hardware | 3B Q4 | 7B Q4 |
|---|---:|---:|
| Laptop i5 (4 cores, AVX2) | 8-12 t/s | 3-6 t/s |
| Desktop Ryzen 5/i5 (6 cores, AVX2) | 10-18 t/s | 5-10 t/s |
| Desktop Ryzen 7/i7 (8 cores, AVX2 or AVX-512) | 15-25 t/s | 8-14 t/s |
| High-end workstation (Threadripper, Xeon, AVX-512) | 20-40 t/s | 12-20 t/s |

For real-time chat UIs, aim for 8+ t/s. Below that, users notice visible lag per token.

## Tips

- **Close other CPU-heavy work** during inference. Competing threads tank throughput.
- **Disable turbo boost sparingly** — sustained AVX work throttles CPU clocks; undervolting or cooler upgrades can help.
- **Benchmark first with a representative prompt**, not a one-word test.
- **Enable Flash Attention** on long contexts: `ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled`.

## What's next

- [CPU acceleration](/net/developer-reference/acceleration/cpu/) — AVX variants and threading detail.
- [Context parameters](/net/developer-reference/parameters/context/) — `NThreads`, `NThreadsBatch`, KV dtype.
- [Low-memory tuning](/net/use-cases/low-memory-tuning/) — further memory optimization.

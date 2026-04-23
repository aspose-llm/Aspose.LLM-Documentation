---
weight: 140
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/low-memory-tuning/
feedback: LLMNET
version: 26.5.0
title: Low memory tuning
description: Fit Aspose.LLM for .NET into a tight memory budget — small model, short context, KV cache quantization, aggressive offload, and memory mapping.
keywords:
- low memory
- small memory
- embedded
- edge
- KV quantization
- mmap
- context size
---

When memory is tight — edge devices, small VMs, constrained containers, laptops with 16 GB RAM — pick a small preset and tune several levers together. This use case walks through the moves that matter most.

## When to use this pattern

- Deploying on a laptop with 8-16 GB of RAM.
- Shared containers with a 4-8 GB RAM cap.
- Edge devices (NUCs, small ARM boards) with limited memory.
- Sharing a GPU with other processes, leaving only a few GB of VRAM free.

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).

## Pick a small model

Start with the smallest preset that meets your quality bar:

| Preset | Model size | Memory footprint (Q4 + default context) |
|---|---|---|
| `Llama32Preset` | 3B | ~6-8 GB |
| `Phi4Preset` | Mini | ~4-6 GB |
| `Qwen25VL3BPreset` | 3B VL | ~4-6 GB + projector |
| `Qwen3VL2BPreset` | 2B VL | ~3-5 GB + projector |

Larger presets (7B, 8B, 20B) push past 8 GB easily — avoid them when memory is scarce.

## Shrink context size

Built-in presets often declare very long contexts. Shorten to what you actually need:

```csharp
var preset = new Llama32Preset();
preset.ContextParameters.ContextSize = 4096; // default is 131072
```

KV cache scales with context; shorter context directly reduces memory.

## Quantize the KV cache

```csharp
using Aspose.LLM.Abstractions.Models;

preset.ContextParameters.TypeK = GgmlType.F16;    // keep K precise
preset.ContextParameters.TypeV = GgmlType.Q8_0;   // V to 8-bit
```

This halves V-cache memory with minor quality impact. For even tighter budgets:

```csharp
preset.ContextParameters.TypeV = GgmlType.Q4_0;   // aggressive
```

Be aware: aggressive KV quantization degrades output quality, especially at long ranges. Validate on your workload.

## Enable memory mapping

`UseMemoryMapping = true` (the default) lets the OS page model weights from disk on demand rather than copying them all into RAM at load time. Keep this enabled:

```csharp
preset.BaseModelInferenceParameters.UseMemoryMapping = true;
```

On very tight memory, disable memory locking (if enabled):

```csharp
preset.BaseModelInferenceParameters.UseMemoryLocking = false; // explicit
```

## Enable flash attention

Even on short contexts, flash attention reduces memory during generation:

```csharp
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
```

## Drop `MaxTokens`

Cap response length so the model does not generate long outputs that extend the KV cache needlessly:

```csharp
preset.ChatParameters.MaxTokens = 256;
```

## Full minimal-memory example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Acceleration;
using Aspose.LLM.Abstractions.Models;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Llama32Preset();

// CPU-only to avoid GPU memory.
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.AVX2;
preset.BaseModelInferenceParameters.GpuLayers = 0;
preset.BaseModelInferenceParameters.UseMemoryMapping = true;

// Short context.
preset.ContextParameters.ContextSize = 4096;

// Aggressive KV quantization.
preset.ContextParameters.TypeK = GgmlType.F16;
preset.ContextParameters.TypeV = GgmlType.Q8_0;

// Flash attention.
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;

// Short response cap.
preset.ChatParameters.MaxTokens = 256;

// Conservative threading.
preset.ContextParameters.NThreads = 2;
preset.ContextParameters.NThreadsBatch = 4;

using var api = AsposeLLMApi.Create(preset);

string reply = await api.SendMessageAsync("Summarize cloud computing in one sentence.");
Console.WriteLine(reply);
```

Expected footprint: 4-5 GB RAM, fits comfortably in an 8 GB machine.

## Monitor actual usage

Check real memory usage after load:

- **Windows**: Task Manager → Process details → Memory.
- **Linux**: `top`, `htop`, or `/proc/<pid>/status`.
- **macOS**: Activity Monitor → memory.

The real number includes OS page cache of memory-mapped model files — it may look high but most of it is reclaimable under memory pressure.

## Trade-offs

| What you cut | Pain |
|---|---|
| Context size | Less room for long prompts; no long-form analysis. |
| KV quantization to Q8_0 | Minor quality drop on long contexts. |
| KV quantization to Q4_0 | Noticeable quality drop. |
| `MaxTokens` | Truncated answers; bad for reasoning models (see [Chat parameters](/net/developer-reference/parameters/chat/)). |
| Small model | Lower reasoning quality. |
| CPU-only | Slower throughput (5-15 t/s vs 40-100+ on GPU). |

Start by cutting context and enabling KV quantization. Only if memory is still tight should you drop to an even smaller model.

## What's next

- [Context parameters](/net/developer-reference/parameters/context/) — every knob that affects memory.
- [CPU-only deployment](/net/use-cases/cpu-only-deployment/) — when GPU memory is simply unavailable.
- [Long context tuning](/net/use-cases/long-context-tuning/) — the opposite direction.

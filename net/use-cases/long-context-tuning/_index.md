---
weight: 130
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/long-context-tuning/
feedback: LLMNET
version: 26.5.0
title: Long context tuning
description: Configure Aspose.LLM for .NET for 128K-262K contexts — pick the right preset, enable flash attention, quantize the KV cache, and manage memory.
keywords:
- long context
- 128K
- 262K
- YaRN
- flash attention
- KV cache
- rope scaling
---

Several built-in presets support very long contexts: `Llama32Preset` (131K), `Oss20Preset` (131K), `DeepSeekCoder2Preset` (163K), `Qwen3VL2BPreset` (262K), `Ministral3VisionPreset` (262K). Running them at full context takes specific tuning — flash attention, KV dtype, sometimes YaRN.

## When to use this pattern

- Summarize or query long documents (books, code repositories, legal contracts).
- Multi-hour conversations that must not lose earlier turns.
- Search-augmented QA where the retrieved context is large.
- Vision + text with many image turns.

## Prerequisites

- A preset with a long default context, or a custom preset with your target `ContextSize`.
- Enough memory. At 131K context, a 7B model's KV cache alone is 8-16 GB depending on KV dtype. Budget accordingly.

## Pick a long-context preset

| Preset | Default context | Notes |
|---|---:|---|
| `Llama32Preset` | 131 072 | 3B; good fit for long-document summarization. |
| `Oss20Preset` | 131 072 | 20B; better reasoning at length. |
| `DeepSeekCoder2Preset` | 163 840 | Code-focused. |
| `Qwen3VL2BPreset` | 262 144 | Vision + long context. |
| `Ministral3VisionPreset` | 262 144 | 8B vision + 262K. |

All of these already have sensible long-context configuration baked in. You mostly need to decide how much of the context to actually use and pay the memory cost.

## Enable flash attention

Flash attention is an order-of-magnitude improvement for long-context memory and speed. Enable unconditionally when it is supported:

```csharp
using Aspose.LLM.Abstractions.Models;

preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
```

Without flash attention, KV reads scale quadratically with context size — it is effectively unworkable past ~32K tokens.

## Quantize the KV cache

F16 KV cache at 131K context is ~8 GB on a 7B model. Q8_0 halves that. Q4 halves again.

```csharp
preset.ContextParameters.TypeK = GgmlType.F16;   // keep K precise
preset.ContextParameters.TypeV = GgmlType.Q8_0;  // quantize V aggressively
```

V is less sensitive than K — quantizing V to Q8_0 has minor quality impact; quantizing K to Q8 has a visible impact. Start with `TypeV = Q8_0` and leave `TypeK = F16` unless memory demands more.

## Use the full context window

If you want to actually use 131K tokens, ensure `ContextSize` matches:

```csharp
preset.ContextParameters.ContextSize = 131072;
```

The built-in preset already sets this; you only override when trimming for memory.

## Memory planning

Approximate KV cache size for a model with `L` layers, `H` heads of dim `D`, context `C`, dtype `T`:

```
KV bytes ≈ 2 × L × H × D × C × T_bytes
```

Rough table for a 7B model (32 layers, 32 heads × 128 dim) at 131K context:

| KV dtype | Per-token bytes | 131K tokens | Total |
|---|---:|---:|---:|
| F16 (2 bytes) | 32 × 32 × 128 × 2 × 2 = 524 288 | 131 072 | ~64 GB |

That number is misleading because `llama.cpp` implements grouped-query attention and compressed layouts; real-world KV cache for a 7B at 131K is more like **8-16 GB F16**. Benchmark on your hardware rather than computing from first principles.

## Full example — document summarization

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Models;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Llama32Preset(); // 3B, 131K context
preset.ChatParameters.SystemPrompt =
    "You summarize long documents. Produce a concise 5-bullet summary of the key points.";
preset.ChatParameters.MaxTokens = 512;

// Enable flash attention — essential at long contexts.
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;

// Quantize V cache to save VRAM.
preset.ContextParameters.TypeV = GgmlType.Q8_0;

// Full GPU offload.
preset.BaseModelInferenceParameters.GpuLayers = 999;

using var api = AsposeLLMApi.Create(preset);

string longDocument = File.ReadAllText("contract.txt"); // up to ~80K tokens
string summary = await api.SendMessageAsync(
    $"Summarize this document:\n\n{longDocument}");

Console.WriteLine(summary);
```

## Multi-turn long-context

When a conversation runs for a long time, combine long context with a cache cleanup strategy that preserves the original question:

```csharp
preset.ChatParameters.CacheCleanupStrategy =
    CacheCleanupStrategy.KeepSystemPromptAndFirstUserMessage;
```

The model keeps the system prompt and the initial user turn regardless of how much middle history evicts.

## YaRN scaling (for stretching beyond training context)

If your custom preset uses a model trained on fewer tokens than you want to run at, YaRN can interpolate positions. Use only when:

- Your model's GGUF metadata specifies a smaller training context than you need.
- You are prepared for some quality degradation at very long reaches.

```csharp
preset.ContextParameters.RopeScalingType = RopeScalingType.Yarn;
preset.ContextParameters.YarnOrigCtx = 32768; // the model's original training length
preset.ContextParameters.ContextSize = 131072; // where you want to go
```

Built-in presets with long contexts do not need this — they are trained on or calibrated for their declared `ContextSize`.

## Performance at long contexts

Throughput drops as context grows. A typical pattern:

- At 4K tokens of prompt: 80-100 t/s on GPU.
- At 32K: 40-60 t/s.
- At 131K: 10-25 t/s.

Budget time for the **first** response — filling a 131K context takes tens of seconds to minutes even on fast GPUs. Subsequent turns in the same session reuse the KV cache and are much faster.

## What's next

- [Context parameters](/net/developer-reference/parameters/context/) — full list of context knobs.
- [Low-memory tuning](/net/use-cases/low-memory-tuning/) — when long context is not worth the memory cost.
- [Cache management](/net/developer-reference/cache-management/) — keep sessions fresh over time.

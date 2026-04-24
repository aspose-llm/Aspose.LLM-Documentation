---
weight: 160
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/flash-attention-mode/
feedback: LLMNET
version: 26.5.0
title: FlashAttentionMode
description: Flash attention mode in Aspose.LLM for .NET — Auto lets the runtime pick, Enabled forces flash attention, Disabled turns it off.
keywords:
- FlashAttentionMode
- flash attention
- fused kernel
- performance
- memory
---

`FlashAttentionMode` controls flash attention — a fused-kernel optimization that reduces memory usage and speeds up attention, especially at long contexts. Prefer `FlashAttentionMode` over the legacy [`FlashAttention`](/net/developer-reference/parameters/context/flash-attention/) boolean.

## Quick reference

| | |
|---|---|
| **Type** | `FlashAttentionType?` enum |
| **Default** | `null` (use model / runtime default) |
| **Values** | `Auto` (-1), `Disabled` (0), `Enabled` (1) |
| **Category** | Attention |
| **Field on** | `ContextParameters.FlashAttentionMode` |

## What it does

Flash attention implements attention in a single fused kernel that tiles the computation. This avoids materializing the full `N × N` attention matrix in memory, which is a big win at long contexts.

| Value | Behavior |
|---|---|
| `Auto` (`-1`) | Runtime picks based on backend support. Recommended. |
| `Disabled` (`0`) | Never use flash attention. Slower and more memory at long contexts. |
| `Enabled` (`1`) | Force flash attention. Fails on backends that do not support it. |

On long contexts (> 8K tokens), flash attention is typically 20-40 % faster and meaningfully reduces peak memory. At short contexts the benefit is small.

## When to change it

| Scenario | Value |
|---|---|
| Default (recommended) | `Auto` |
| Explicit opt-in for benchmarking | `Enabled` |
| Debugging / disabling suspected flash-attention bug | `Disabled` |
| Backend without flash attention support | Runtime picks `Disabled` automatically when `Auto` |

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
preset.ContextParameters.ContextSize = 32768;  // long context benefits most from FA

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — larger contexts benefit more from flash attention.
- [`TypeK`](/net/developer-reference/parameters/context/type-k/), [`TypeV`](/net/developer-reference/parameters/context/type-v/) — flash attention works with quantized KV cache.
- [`FlashAttention`](/net/developer-reference/parameters/context/flash-attention/) — legacy boolean; prefer `FlashAttentionMode`.
- Acceleration backends — CUDA, Metal, HIP, Vulkan all support flash attention on recent drivers.

## What's next

- [FlashAttention](/net/developer-reference/parameters/context/flash-attention/) — legacy field; documented for completeness.
- [ContextSize](/net/developer-reference/parameters/context/context-size/) — the axis where FA matters most.
- [Long context tuning](/net/use-cases/long-context-tuning/) — practical recipe.

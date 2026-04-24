---
weight: 200
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/type-k/
feedback: LLMNET
version: 26.5.0
title: TypeK
description: KV cache dtype for the K (keys) tensor in Aspose.LLM for .NET — F16 default, quantized types save memory at long contexts.
keywords:
- TypeK
- KV cache
- K tensor
- quantization
- memory
---

`TypeK` is the data type used to store the **K (keys)** tensor of the KV cache. Choosing a smaller dtype reduces KV cache memory at the cost of slight quality loss.

## Quick reference

| | |
|---|---|
| **Type** | `GgmlType?` enum (39 values) |
| **Default** | `null` (use native default, usually `F16`) |
| **Common values** | `F32`, `F16`, `BF16`, `Q8_0`, `Q5_1`, `Q4_0` |
| **Category** | KV cache |
| **Field on** | `ContextParameters.TypeK` |

## What it does

For each layer, the engine stores one K tensor of shape `(heads × seq_len × head_dim)`. The dtype controls memory per element:

| Dtype | Bits | Relative size | Quality impact |
|---|---:|---:|---|
| `F32` | 32 | 1.0× | None. Rarely worth the memory. |
| `F16` | 16 | 0.5× | Default. Minimal impact. |
| `BF16` | 16 | 0.5× | Alternative to F16. Slightly different numerical range. |
| `Q8_0` | 8 | 0.25× | Very small quality loss for substantial savings. |
| `Q5_1` | 5 | ~0.16× | Noticeable at long contexts. |
| `Q4_0` | 4 | ~0.125× | Aggressive; use only under strong memory pressure. |

**Rule of thumb**: K is more sensitive to precision than V. Prefer to quantize V more aggressively than K.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` (F16) |
| Long context, mild memory pressure | `F16` (keep K high precision) |
| Long context, tight memory | `Q8_0` |
| Extreme memory constraint | `Q5_1` (accept quality drop) |

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ContextParameters.TypeK = GgmlType.F16;   // keep K precise
preset.ContextParameters.TypeV = GgmlType.Q8_0;  // quantize V

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`TypeV`](/net/developer-reference/parameters/context/type-v/) — companion V-cache dtype.
- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — cache cost scales with context; quantization pays off more at long contexts.
- [`FlashAttentionMode`](/net/developer-reference/parameters/context/flash-attention-mode/) — works with quantized KV on recent backends.

## What's next

- [TypeV](/net/developer-reference/parameters/context/type-v/) — V-cache dtype.
- [Low memory tuning](/net/use-cases/low-memory-tuning/) — practical quantization recipes.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

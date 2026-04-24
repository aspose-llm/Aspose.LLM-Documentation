---
weight: 210
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/type-v/
feedback: LLMNET
version: 26.5.0
title: TypeV
description: KV cache dtype for the V (values) tensor in Aspose.LLM for .NET — tolerates more aggressive quantization than TypeK; Q8_0 is a safe default for tight memory.
keywords:
- TypeV
- KV cache
- V tensor
- quantization
- memory
---

`TypeV` is the data type for the **V (values)** tensor of the KV cache. V tolerates more aggressive quantization than K — `Q8_0` is often a safe default when memory is tight.

## Quick reference

| | |
|---|---|
| **Type** | `GgmlType?` enum (39 values) |
| **Default** | `null` (use native default, usually `F16`) |
| **Common values** | `F32`, `F16`, `BF16`, `Q8_0`, `Q5_1`, `Q4_0` |
| **Category** | KV cache |
| **Field on** | `ContextParameters.TypeV` |

## What it does

Identical mechanics to [`TypeK`](/net/developer-reference/parameters/context/type-k/) — stored shape, memory scaling, and options are the same. The difference is sensitivity: attention output depends on V through a weighted sum, which averages over many tokens, so quantization error averages out. Attention scores depend on K through a dot product where individual errors survive more.

**Rule of thumb**: Quantize V one step more aggressively than K.

| Configuration | K | V |
|---|---|---|
| Default balanced | `F16` | `F16` |
| Save memory with minimal quality loss | `F16` | `Q8_0` |
| Tight memory | `Q8_0` | `Q8_0` |
| Very tight | `Q8_0` | `Q5_1` |
| Extreme | `Q8_0` | `Q4_0` |

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` (F16) |
| Save memory with minimal quality loss | `Q8_0` |
| Tight memory | `Q5_1` |
| Extreme memory pressure | `Q4_0` |

V quantization at `Q8_0` is often indistinguishable from `F16` on benchmark tasks. Start there when memory is tight.

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ContextParameters.TypeV = GgmlType.Q8_0;
// V quantized; K left at default F16.

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`TypeK`](/net/developer-reference/parameters/context/type-k/) — companion K-cache dtype.
- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — memory savings scale with context.
- [`FlashAttentionMode`](/net/developer-reference/parameters/context/flash-attention-mode/) — compatible with quantized V.

## What's next

- [TypeK](/net/developer-reference/parameters/context/type-k/) — companion.
- [Low memory tuning](/net/use-cases/low-memory-tuning/) — recipes for tight-memory deployments.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

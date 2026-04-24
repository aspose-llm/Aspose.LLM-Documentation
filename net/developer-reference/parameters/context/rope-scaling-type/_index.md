---
weight: 70
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/rope-scaling-type/
feedback: LLMNET
version: 26.5.0
title: RopeScalingType
description: RoPE position encoding scaling algorithm in Aspose.LLM for .NET — choose None, Linear, YaRN, or LongRoPE for extending context beyond the model's training window.
keywords:
- RopeScalingType
- RoPE
- position encoding
- YaRN
- LongRoPE
- linear scaling
---

`RopeScalingType` selects the algorithm used to scale RoPE (Rotary Position Embedding) when the effective context size exceeds the model's training window. Different algorithms produce different quality trade-offs at long contexts.

## Quick reference

| | |
|---|---|
| **Type** | `RopeScalingType?` enum |
| **Default** | `null` (use model default) |
| **Values** | `Unspecified`, `None`, `Linear`, `Yarn`, `LongRope` |
| **Category** | Position encoding |
| **Field on** | `ContextParameters.RopeScalingType` |

## What it does

Transformer models use RoPE to encode token positions. The frequencies RoPE uses are trained on a specific maximum context. To go beyond that trained maximum, the position encoding must be scaled.

| Value | Behavior |
|---|---|
| `Unspecified` (`-1`) | Use whatever the model's GGUF metadata specifies. |
| `None` (`0`) | No scaling; use raw RoPE. Only valid within the trained context. |
| `Linear` (`1`) | Linear interpolation of positions. Simple, moderate quality loss. |
| `Yarn` (`2`) | YaRN (Yet another RoPE extensioN) — higher quality at long contexts. |
| `LongRope` (`3`) | LongRoPE algorithm for very extended contexts. |

Most built-in presets leave this as `Unspecified` — the model's metadata declares its own preferred scaling. Override only when you push the model past its declared maximum.

## When to change it

| Scenario | Value |
|---|---|
| Default | `Unspecified` (model's metadata wins) |
| Run a model within its native context window | `None` or `Unspecified` |
| Extend context 2-4× training window, simple scaling acceptable | `Linear` |
| Extend context with better quality | `Yarn` |
| Push toward 1M+ contexts | `LongRope` (if model supports it) |

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Llama32Preset();
preset.ContextParameters.ContextSize = 131072;
preset.ContextParameters.RopeScalingType = RopeScalingType.Yarn;
preset.ContextParameters.YarnOrigCtx = 8192;  // the model's original training context

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`RopeFreqBase`](/net/developer-reference/parameters/context/rope-freq-base/), [`RopeFreqScale`](/net/developer-reference/parameters/context/rope-freq-scale/) — apply on top of the chosen scaling.
- [`YarnExtFactor`](/net/developer-reference/parameters/context/yarn-ext-factor/), [`YarnAttnFactor`](/net/developer-reference/parameters/context/yarn-attn-factor/), [`YarnBetaFast`](/net/developer-reference/parameters/context/yarn-beta-fast/), [`YarnBetaSlow`](/net/developer-reference/parameters/context/yarn-beta-slow/), [`YarnOrigCtx`](/net/developer-reference/parameters/context/yarn-orig-ctx/) — only used when `RopeScalingType = Yarn`.
- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — larger than the model's training window requires RoPE scaling.

## What's next

- [YarnOrigCtx](/net/developer-reference/parameters/context/yarn-orig-ctx/) — the model's native context length.
- [Long context tuning](/net/use-cases/long-context-tuning/) — practical recipes.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

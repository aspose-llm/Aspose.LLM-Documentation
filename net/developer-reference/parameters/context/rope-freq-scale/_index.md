---
weight: 90
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/rope-freq-scale/
feedback: LLMNET
version: 26.5.0
title: RopeFreqScale
description: RoPE frequency scaling factor in Aspose.LLM for .NET — multiplicative factor applied to RoPE frequencies; null or 0 uses the model default.
keywords:
- RopeFreqScale
- RoPE
- frequency scale
- linear scaling
---

`RopeFreqScale` is a multiplicative scaling factor applied to RoPE frequencies. It implements simple linear scaling of positions — equivalent to `Linear` [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) at the value set here.

## Quick reference

| | |
|---|---|
| **Type** | `float?` |
| **Default** | `null` (use model default) |
| **Range** | `0` – `1.0`; `< 1.0` stretches the context window |
| **Category** | Position encoding |
| **Field on** | `ContextParameters.RopeFreqScale` |

## What it does

RoPE frequencies are scaled by `RopeFreqScale`. A scale of `1.0` is no scaling. Smaller values stretch the effective context:

- `RopeFreqScale = 1.0` — no scaling.
- `RopeFreqScale = 0.5` — effective context doubled (2× training window). Moderate quality loss.
- `RopeFreqScale = 0.25` — 4× training window. More quality loss.

This is the simplest context-extension approach. More sophisticated algorithms (`Yarn`, `LongRope`) produce better quality at the same effective extension.

## When to change it

| Scenario | Value |
|---|---|
| Default (use model metadata) | `null` |
| Simple 2× extension | `0.5` with `RopeScalingType = Linear` |
| Prefer better algorithms | Use `RopeScalingType = Yarn` instead |

Modern built-in presets target models that already ship with proper scaling metadata. Override only when adapting a model without adequate metadata.

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ContextParameters.RopeScalingType = RopeScalingType.Linear;
preset.ContextParameters.RopeFreqScale = 0.5f;  // 2x linear extension
```

## Interactions

- [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) — `Linear` uses this scale; `Yarn`/`LongRope` have their own knobs.
- [`RopeFreqBase`](/net/developer-reference/parameters/context/rope-freq-base/) — base frequency.
- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — the target extended context size.

## What's next

- [RopeScalingType](/net/developer-reference/parameters/context/rope-scaling-type/) — algorithm selector.
- [YarnOrigCtx](/net/developer-reference/parameters/context/yarn-orig-ctx/) — better long-context extension via YaRN.
- [Long context tuning](/net/use-cases/long-context-tuning/) — practical recipes.

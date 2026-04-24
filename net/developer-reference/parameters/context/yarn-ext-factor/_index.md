---
weight: 100
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/yarn-ext-factor/
feedback: LLMNET
version: 26.5.0
title: YarnExtFactor
description: YaRN extrapolation mix factor in Aspose.LLM for .NET — negative or null uses the model default; most users leave this alone.
keywords:
- YarnExtFactor
- YaRN
- extrapolation
- RoPE
---

`YarnExtFactor` is the YaRN extrapolation mix factor. It blends between base RoPE and NTK-aware scaling in the YaRN algorithm. Relevant only when [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) is `Yarn`.

## Quick reference

| | |
|---|---|
| **Type** | `float?` |
| **Default** | `null` (negative / model default) |
| **Range** | `0.0` – `1.0`; negative means "from model" |
| **Category** | YaRN position encoding |
| **Field on** | `ContextParameters.YarnExtFactor` |

## What it does

YaRN combines position interpolation and extrapolation. `YarnExtFactor` controls the mix between the two. The model's GGUF metadata usually sets this correctly for the intended scaling factor; overriding is rarely useful.

- `null` or negative — use the model's value.
- `0.0` — pure interpolation.
- `1.0` — pure extrapolation.
- Intermediate — blend.

## When to change it

| Scenario | Value |
|---|---|
| Default (recommended) | `null` |
| Experimental YaRN tuning | Per upstream YaRN paper recipe |

## Example

```csharp
var preset = new Llama32Preset();
preset.ContextParameters.RopeScalingType = RopeScalingType.Yarn;
// preset.ContextParameters.YarnExtFactor = null; // default — use model's value
```

## Interactions

- [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) — must be `Yarn`.
- [`YarnAttnFactor`](/net/developer-reference/parameters/context/yarn-attn-factor/), [`YarnBetaFast`](/net/developer-reference/parameters/context/yarn-beta-fast/), [`YarnBetaSlow`](/net/developer-reference/parameters/context/yarn-beta-slow/), [`YarnOrigCtx`](/net/developer-reference/parameters/context/yarn-orig-ctx/) — other YaRN knobs.

## What's next

- [YarnOrigCtx](/net/developer-reference/parameters/context/yarn-orig-ctx/) — the most commonly set YaRN knob.
- [RopeScalingType](/net/developer-reference/parameters/context/rope-scaling-type/) — enables YaRN.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

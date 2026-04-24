---
weight: 120
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/yarn-beta-fast/
feedback: LLMNET
version: 26.5.0
title: YarnBetaFast
description: YaRN low correction dim in Aspose.LLM for .NET — the "fast" wavelength corner where YaRN transitions between interpolation modes.
keywords:
- YarnBetaFast
- YaRN
- correction dimension
- RoPE
---

`YarnBetaFast` is the "fast" boundary of YaRN's correction range — the position-dimension index below which no correction is applied. Relevant only when [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) is `Yarn`.

## Quick reference

| | |
|---|---|
| **Type** | `float?` |
| **Default** | `null` (use model default) |
| **Range** | Typical `30` – `50` (dimension index) |
| **Category** | YaRN position encoding |
| **Field on** | `ContextParameters.YarnBetaFast` |

## What it does

YaRN applies different treatment to different RoPE dimensions based on their frequency (wavelength). Below `YarnBetaFast`, positions are treated with raw extrapolation (no correction). Between `YarnBetaFast` and [`YarnBetaSlow`](/net/developer-reference/parameters/context/yarn-beta-slow/), YaRN blends the two regimes.

- `null` — use model default (typical value in YaRN papers is `32`).
- Specific float — override.

Rarely touched. The model's metadata carries sensible values.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Research / YaRN tuning | Per paper recipe |

## Example

```csharp
// Default — do not override.
var preset = new Llama32Preset();
preset.ContextParameters.RopeScalingType = RopeScalingType.Yarn;
```

## Interactions

- [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) — must be `Yarn`.
- [`YarnBetaSlow`](/net/developer-reference/parameters/context/yarn-beta-slow/) — upper boundary of the blend range.

## What's next

- [YarnBetaSlow](/net/developer-reference/parameters/context/yarn-beta-slow/) — companion upper boundary.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

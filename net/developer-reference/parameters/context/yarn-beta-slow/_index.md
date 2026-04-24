---
weight: 130
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/yarn-beta-slow/
feedback: LLMNET
version: 26.5.0
title: YarnBetaSlow
description: YaRN high correction dim in Aspose.LLM for .NET — the "slow" wavelength corner where YaRN transitions between interpolation modes.
keywords:
- YarnBetaSlow
- YaRN
- correction dimension
- RoPE
---

`YarnBetaSlow` is the "slow" boundary of YaRN's correction range — the dimension index above which interpolation is fully applied. Relevant only when [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) is `Yarn`.

## Quick reference

| | |
|---|---|
| **Type** | `float?` |
| **Default** | `null` (use model default) |
| **Range** | Typical `1` |
| **Category** | YaRN position encoding |
| **Field on** | `ContextParameters.YarnBetaSlow` |

## What it does

Pairs with [`YarnBetaFast`](/net/developer-reference/parameters/context/yarn-beta-fast/) to define the transition window between extrapolation and interpolation inside YaRN:

- Below `YarnBetaFast`: pure extrapolation.
- Between `YarnBetaFast` and `YarnBetaSlow`: blend.
- Above `YarnBetaSlow`: pure interpolation.

Typical YaRN recipe values: `YarnBetaFast = 32`, `YarnBetaSlow = 1`.

## When to change it

| Scenario | Value |
|---|---|
| Default (recommended) | `null` |
| Research / YaRN tuning | Per paper recipe |

## Example

```csharp
var preset = new Llama32Preset();
preset.ContextParameters.RopeScalingType = RopeScalingType.Yarn;
// preset.ContextParameters.YarnBetaSlow = null; // default
```

## Interactions

- [`YarnBetaFast`](/net/developer-reference/parameters/context/yarn-beta-fast/) — lower boundary.
- [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) — must be `Yarn`.

## What's next

- [YarnBetaFast](/net/developer-reference/parameters/context/yarn-beta-fast/) — companion lower boundary.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

---
weight: 140
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/dynatemp-exponent/
feedback: LLMNET
version: 26.5.0
title: DynatempExponent
description: Dynatemp curve exponent in Aspose.LLM for .NET — shapes how entropy maps to temperature; linear at 1.0, more aggressive at higher values.
keywords:
- DynatempExponent
- dynamic temperature
- sampler
- entropy curve
---

`DynatempExponent` controls the shape of the entropy-to-temperature mapping in dynamic temperature sampling. It only matters when [`DynatempRange`](/net/developer-reference/parameters/sampler/dynatemp-range/) is enabled (non-zero).

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `1.0` (linear mapping) |
| **Range** | `> 0` |
| **Category** | Adaptive temperature |
| **Field on** | `SamplerParameters.DynatempExponent` |

## What it does

Dynatemp maps the normalized entropy `e ∈ [0, 1]` at each step to an offset within `DynatempRange`. `DynatempExponent` reshapes the mapping:

- `DynatempExponent = 1.0` — linear mapping. Temperature scales proportionally with entropy.
- `DynatempExponent > 1.0` — convex. Only very high entropy triggers significant temperature increases; medium entropy stays near the base.
- `DynatempExponent < 1.0` — concave. Small entropy changes trigger larger temperature swings.

The exact formula from `llama.cpp`: the step's temperature is the base `Temperature` adjusted by an offset proportional to `entropy^DynatempExponent` over `DynatempRange`.

## When to change it

| Scenario | Value |
|---|---|
| Default linear mapping | `1.0` |
| Only react to very uncertain steps | `1.5` – `2.0` |
| React to mild uncertainty | `0.7` – `0.9` |

Most users leave `DynatempExponent = 1.0`. Change only after experimenting with `DynatempRange` alone and finding the linear mapping unsuitable for your workload.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.Temperature = 0.7f;
preset.SamplerParameters.DynatempRange = 0.3f;
preset.SamplerParameters.DynatempExponent = 1.5f;
// Temperature rises significantly only on high-entropy steps; low-to-medium
// entropy stays close to 0.7.

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`DynatempRange`](/net/developer-reference/parameters/sampler/dynatemp-range/) — must be non-zero for `DynatempExponent` to have any effect.
- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/) — the base temperature dynatemp varies around.

## What's next

- [DynatempRange](/net/developer-reference/parameters/sampler/dynatemp-range/) — enables dynatemp.
- [Temperature](/net/developer-reference/parameters/sampler/temperature/) — the base.
- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs.

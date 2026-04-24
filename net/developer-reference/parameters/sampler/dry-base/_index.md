---
weight: 180
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/dry-base/
feedback: LLMNET
version: 26.5.0
title: DryBase
description: DRY exponent base in Aspose.LLM for .NET — controls how fast the DRY penalty grows as the repeated match extends.
keywords:
- DryBase
- DRY
- sampler
- exponent
---

`DryBase` is the exponent base of the DRY penalty growth formula. It determines how quickly the penalty ramps up as a repeated phrase extends. `DryBase` only has effect when [`DryMultiplier`](/net/developer-reference/parameters/sampler/dry-multiplier/) is enabled (positive).

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `1.75` |
| **Range** | `> 1.0`; typical `1.5` – `2.5` |
| **Category** | Phrase-level repetition penalty |
| **Field on** | `SamplerParameters.DryBase` |

## What it does

The DRY penalty applied to a token that would extend a repeated match is:

```
penalty = DryMultiplier × DryBase^(match_length - DryAllowedLength)
```

Larger `DryBase` means the penalty grows faster. A 5-token match with `DryBase = 1.75` produces `1.75^5 ≈ 16` multiplied by `DryMultiplier`; at `DryBase = 2.5` the same match yields `2.5^5 ≈ 98`.

- `DryBase = 1.5` — gentle growth; long repeats punished but not extremely.
- `DryBase = 1.75` (default) — balanced; the value recommended by the original DRY paper.
- `DryBase = 2.0+` — rapid growth; even medium-length repeats are heavily penalized.

## When to change it

| Scenario | Value |
|---|---|
| Default | `1.75` |
| Gentle phrase anti-repetition | `1.5` |
| Aggressive (model keeps finding ways to repeat) | `2.0` – `2.5` |

Most workloads leave `DryBase` alone and tune [`DryMultiplier`](/net/developer-reference/parameters/sampler/dry-multiplier/) and [`DryAllowedLength`](/net/developer-reference/parameters/sampler/dry-allowed-length/). Change `DryBase` as a last resort if the standard combination does not stop repeats.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.DryMultiplier = 0.8f;
preset.SamplerParameters.DryBase = 2.0f;    // faster growth
preset.SamplerParameters.DryAllowedLength = 2;

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`DryMultiplier`](/net/developer-reference/parameters/sampler/dry-multiplier/) — must be positive for `DryBase` to matter.
- [`DryAllowedLength`](/net/developer-reference/parameters/sampler/dry-allowed-length/) — sets the exponent's zero-point.
- [`DryPenaltyLastN`](/net/developer-reference/parameters/sampler/dry-penalty-last-n/) — window of history DRY scans.
- [`DrySequenceBreakers`](/net/developer-reference/parameters/sampler/dry-sequence-breakers/) — tokens that reset match detection.

## What's next

- [DryMultiplier](/net/developer-reference/parameters/sampler/dry-multiplier/) — enables DRY.
- [DryAllowedLength](/net/developer-reference/parameters/sampler/dry-allowed-length/) — match-length threshold.
- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs.

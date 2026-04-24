---
weight: 60
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/min-keep/
feedback: LLMNET
version: 26.5.0
title: MinKeep
description: Minimum candidate-count floor in Aspose.LLM for .NET — guarantees the sampler always has at least MinKeep tokens after all filters run.
keywords:
- MinKeep
- sampler
- minimum candidates
- safety net
---

`MinKeep` sets a floor on the number of candidate tokens that survive after every filter (`TopK`, `TopP`, `MinP`, `TypicalP`, etc.) runs. If filters would leave fewer than `MinKeep` tokens, the engine widens them until at least `MinKeep` tokens remain.

## Quick reference

| | |
|---|---|
| **Type** | `int` |
| **Default** | `1` |
| **Range** | `1` and above |
| **Category** | Safety / filter floor |
| **Field on** | `SamplerParameters.MinKeep` |

## What it does

After all filters run, count the surviving candidates. If the count is below `MinKeep`, the engine relaxes the filters to bring it back up to `MinKeep` — typically by reverting the last applied filter or by keeping the top-`MinKeep` candidates regardless of probability.

- `MinKeep = 1` (default) — guarantees at least one token survives. Safe default; prevents the pathological case where every filter combines to produce zero candidates.
- `MinKeep = 5` — always keep at least five candidates. Adds variety floor at the cost of filter tightness.

`MinKeep` is a backstop rather than a primary tuning knob. Most users leave it at the default.

## When to change it

| Scenario | Value |
|---|---|
| Default safe floor | `1` |
| Guarantee variety even under strict filters | `3` – `5` |
| Very aggressive tightness with unusual filter combos | Keep at `1` or lower |

Raising `MinKeep` effectively weakens the other filters in corner cases where they combine too tightly. Lowering is not useful — `1` is already the minimum meaningful value.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.TopK = 5;       // tight
preset.SamplerParameters.TopP = 0.3f;    // tight
preset.SamplerParameters.MinP = 0.2f;    // tight
preset.SamplerParameters.MinKeep = 3;    // but at least 3 candidates always survive

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`TopK`](/net/developer-reference/parameters/sampler/top-k/) — `MinKeep` floors the candidate count regardless.
- [`TopP`](/net/developer-reference/parameters/sampler/top-p/) — same — nucleus can't cut below `MinKeep`.
- [`MinP`](/net/developer-reference/parameters/sampler/min-p/) — same.
- [`TypicalP`](/net/developer-reference/parameters/sampler/typical-p/) — same.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — uses its own candidate-management algorithm; `MinKeep` is not relevant when Mirostat is active.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [TopK](/net/developer-reference/parameters/sampler/top-k/) — count-based filter that respects `MinKeep`.
- [TopP](/net/developer-reference/parameters/sampler/top-p/) — cumulative-mass filter that respects `MinKeep`.

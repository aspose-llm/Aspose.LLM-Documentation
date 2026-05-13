---
weight: 190
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/dry-allowed-length/
feedback: LLMNET
version: 26.5.0
title: DryAllowedLength
description: Minimum repeat length before DRY engages in Aspose.LLM for .NET — matches shorter than this are allowed without penalty.
keywords:
- DryAllowedLength
- DRY
- sampler
- match length
- threshold
---

`DryAllowedLength` is the minimum consecutive-token match length before the DRY penalty starts firing. Matches up to this length pass without penalty; longer matches get the exponentially growing DRY penalty.

## Quick reference

| | |
|---|---|
| **Type** | `int` |
| **Default** | `2` |
| **Range** | `1` and above |
| **Category** | Phrase-level repetition penalty |
| **Field on** | `SamplerParameters.DryAllowedLength` |

## What it does

DRY scans recent generation for consecutive-token sequences that match earlier content. Matches strictly longer than `DryAllowedLength` trigger the penalty, which grows by `DryBase` for each token beyond the threshold:

```
penalty_factor = DryMultiplier × DryBase^(match_length - DryAllowedLength)
```

- `DryAllowedLength = 2` (default) — any 3+ token repeat gets penalized. Catches most phrase repeats while allowing natural short patterns.
- `DryAllowedLength = 3` — allows short 3-token patterns to repeat unpenalized. Safer for code / formatted output.
- `DryAllowedLength = 5` — only long repeats are penalized. Use when the default causes awkward word choices.
- `DryAllowedLength = 1` — very aggressive; any 2-token match penalized.

## When to change it

| Scenario | Value |
|---|---|
| Default | `2` |
| Code or formatted output with natural repeats | `4` – `6` |
| Model loops even on short phrases | `1` – `2` |
| Balanced creative writing | `3` |

`DryAllowedLength` is the best knob to tune when DRY starts producing strange word choices — raise it so DRY leaves short natural patterns alone.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.DryMultiplier = 0.8f;
preset.SamplerParameters.DryAllowedLength = 4;  // allow short repeats

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`DryMultiplier`](/llm/net/developer-reference/parameters/sampler/dry-multiplier/) — must be positive for DRY to be active.
- [`DryBase`](/llm/net/developer-reference/parameters/sampler/dry-base/) — sets how fast penalty grows beyond this length.
- [`DrySequenceBreakers`](/llm/net/developer-reference/parameters/sampler/dry-sequence-breakers/) — tokens that reset the match counter.

## What's next

- [DryMultiplier](/llm/net/developer-reference/parameters/sampler/dry-multiplier/) — enables DRY.
- [DryBase](/llm/net/developer-reference/parameters/sampler/dry-base/) — growth rate past this length.
- [Sampler parameters hub](/llm/net/developer-reference/parameters/sampler/) — all sampler knobs.

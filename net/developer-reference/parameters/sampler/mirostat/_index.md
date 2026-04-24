---
weight: 220
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/mirostat/
feedback: LLMNET
version: 26.5.0
title: Mirostat
description: Adaptive entropy-targeting sampler in Aspose.LLM for .NET — mode 0 disables, 1 = Mirostat 1.0, 2 = Mirostat 2.0 (usually preferred).
keywords:
- Mirostat
- adaptive sampler
- entropy
- perplexity
- target entropy
---

`Mirostat` is an adaptive sampler that targets a specific output entropy (roughly perplexity) instead of relying on fixed temperature and nucleus filters. When enabled, it bypasses `Temperature`, `TopP`, `TopK`, and `MinP`.

## Quick reference

| | |
|---|---|
| **Type** | `int` |
| **Default** | `0` (disabled) |
| **Range** | `0` = disabled, `1` = Mirostat 1.0, `2` = Mirostat 2.0 |
| **Category** | Adaptive sampler |
| **Field on** | `SamplerParameters.Mirostat` |

## What it does

Mirostat monitors the entropy of the output distribution and adjusts the sampling process to match a target entropy (`MirostatTau`). Higher measured entropy → Mirostat tightens. Lower → Mirostat relaxes.

- `Mirostat = 0` (default) — disabled. Standard `Temperature` + `TopP` + `TopK` + `MinP` pipeline is used.
- `Mirostat = 1` — Mirostat 1.0. Original algorithm from the paper.
- `Mirostat = 2` — Mirostat 2.0. Simplified and usually preferred; faster convergence.

When Mirostat is enabled, the standard filters (`TopP`, `TopK`, `MinP`, `TypicalP`, `TopNSigma`) are effectively bypassed. `Temperature` tuning is ignored — Mirostat manages its own temperature-like adjustments internally.

## When to change it

| Scenario | Value |
|---|---|
| Default — use standard filters | `0` |
| Adaptive perplexity-targeting (preferred) | `2` |
| Older Mirostat 1.0 (rarely preferred) | `1` |

Consider Mirostat when:
- You want consistent perplexity across very long outputs.
- Fixed temperature produces output that is either too tight or too loose in different parts of the same generation.
- Research / experimentation with entropy-aware sampling.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.Mirostat = 2;           // Mirostat 2.0
preset.SamplerParameters.MirostatTau = 5.0f;     // target entropy
preset.SamplerParameters.MirostatEta = 0.1f;     // learning rate

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`MirostatTau`](/net/developer-reference/parameters/sampler/mirostat-tau/) — target entropy.
- [`MirostatEta`](/net/developer-reference/parameters/sampler/mirostat-eta/) — learning rate.
- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/), [`TopP`](/net/developer-reference/parameters/sampler/top-p/), [`TopK`](/net/developer-reference/parameters/sampler/top-k/), [`MinP`](/net/developer-reference/parameters/sampler/min-p/), [`TypicalP`](/net/developer-reference/parameters/sampler/typical-p/), [`TopNSigma`](/net/developer-reference/parameters/sampler/top-n-sigma/) — all bypassed when Mirostat is active.
- [`DynatempRange`](/net/developer-reference/parameters/sampler/dynatemp-range/) — alternative entropy-aware sampler; do not combine.
- [`Seed`](/net/developer-reference/parameters/sampler/seed/) — still affects the RNG; reproducibility works with Mirostat.

## What's next

- [MirostatTau](/net/developer-reference/parameters/sampler/mirostat-tau/) — entropy target.
- [MirostatEta](/net/developer-reference/parameters/sampler/mirostat-eta/) — learning rate.
- [DynatempRange](/net/developer-reference/parameters/sampler/dynatemp-range/) — alternative adaptive approach.

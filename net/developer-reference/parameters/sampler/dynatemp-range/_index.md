---
weight: 130
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/dynatemp-range/
feedback: LLMNET
version: 26.5.0
title: DynatempRange
description: Dynamic temperature range in Aspose.LLM for .NET — varies Temperature per step based on distribution entropy; enables dynatemp when > 0.
keywords:
- DynatempRange
- dynamic temperature
- entropy
- sampler
- adaptive
---

`DynatempRange` enables **dynamic temperature** (dynatemp). Instead of a fixed `Temperature`, the engine varies temperature per step based on how confident the model is at that step. Low-entropy (confident) steps drop the temperature; high-entropy (uncertain) steps raise it.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `0.0` (disabled) |
| **Range** | `0.0` = disabled, typical active values `0.1` – `0.5` |
| **Category** | Adaptive temperature |
| **Field on** | `SamplerParameters.DynatempRange` |

## What it does

At each step, `llama.cpp` computes the Shannon entropy of the current token distribution. It then adjusts `Temperature` within `[Temperature - DynatempRange/2, Temperature + DynatempRange/2]`:

- When entropy is low (model is confident about the next token), temperature drops toward the lower bound — preserves the confident choice.
- When entropy is high (many plausible tokens), temperature rises toward the upper bound — encourages variety where the model has no strong preference.

The exact shape of the entropy-to-temperature curve is controlled by [`DynatempExponent`](/net/developer-reference/parameters/sampler/dynatemp-exponent/).

- `DynatempRange = 0.0` (default) — dynatemp disabled; `Temperature` is used as-is.
- `DynatempRange = 0.3` — moderate adaptive variation.
- `DynatempRange = 0.5` — wide swing; near-greedy on confident steps, high variety on uncertain steps.

## When to change it

| Scenario | Value |
|---|---|
| Default (disabled) | `0.0` |
| Slight adaptation | `0.2` |
| Classical dynatemp recipe | `0.3` – `0.4` |
| Aggressive adaptation | `0.5` |

Dynatemp shines on mixed-content generation (structured plus creative) where fixed `Temperature` struggles. For pure factual tasks, leave disabled. For pure creative writing, fixed high `Temperature` is simpler.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.Temperature = 0.8f;          // mid-point
preset.SamplerParameters.DynatempRange = 0.3f;        // effective range [0.65, 0.95]
preset.SamplerParameters.DynatempExponent = 1.0f;     // linear entropy-to-temperature curve

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/) — the base / mid-point around which dynatemp varies.
- [`DynatempExponent`](/net/developer-reference/parameters/sampler/dynatemp-exponent/) — shapes the entropy-to-temperature curve.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — alternative entropy-aware sampler; do not combine.

## What's next

- [DynatempExponent](/net/developer-reference/parameters/sampler/dynatemp-exponent/) — the curve-shape knob.
- [Temperature](/net/developer-reference/parameters/sampler/temperature/) — the fixed baseline.
- [Mirostat](/net/developer-reference/parameters/sampler/mirostat/) — alternative adaptive sampler.

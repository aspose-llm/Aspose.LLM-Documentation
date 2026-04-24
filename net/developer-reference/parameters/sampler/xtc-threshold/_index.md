---
weight: 160
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/xtc-threshold/
feedback: LLMNET
version: 26.5.0
title: XtcThreshold
description: Probability threshold for the XTC sampler in Aspose.LLM for .NET — tokens above this probability can be excluded when XTC fires.
keywords:
- XtcThreshold
- XTC
- Exclude Top Choices
- sampler
- threshold
---

`XtcThreshold` is the probability cutoff for XTC (eXclude Top Choices). Only tokens whose probability is **above** `XtcThreshold` are candidates for XTC exclusion. This field has no effect unless [`XtcProbability`](/net/developer-reference/parameters/sampler/xtc-probability/) is enabled.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `0.0` |
| **Range** | `0.0` – `1.0` |
| **Category** | Advanced / diversity |
| **Field on** | `SamplerParameters.XtcThreshold` |

## What it does

When XTC fires at a step, the engine excludes every token whose probability exceeds `XtcThreshold`:

- `XtcThreshold = 0.0` — any token with non-zero probability can be excluded. Very aggressive.
- `XtcThreshold = 0.1` — only tokens above 10 % probability are exclusion candidates. Mild.
- `XtcThreshold = 0.3` — only dominant tokens (>30 % probability) get excluded. Very selective.
- `XtcThreshold = 1.0` — nothing can be excluded; XTC effectively does nothing.

Raise `XtcThreshold` when XTC is producing too much noise (excluding too many tokens); lower it for stronger diversity injection.

## When to change it

| Scenario | Value |
|---|---|
| Default | `0.0` |
| Target only dominant tokens | `0.1` – `0.2` |
| Target only near-certain tokens | `0.3` – `0.5` |
| Effectively disable (keep `XtcProbability` for logging) | `1.0` |

`XtcThreshold` tunes "which tokens XTC may touch". `XtcProbability` tunes "how often XTC runs at all".

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.XtcProbability = 0.3f;
preset.SamplerParameters.XtcThreshold = 0.2f;
// 30 % of steps, exclude tokens with probability > 20 %.

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`XtcProbability`](/net/developer-reference/parameters/sampler/xtc-probability/) — gate; `XtcThreshold` only matters when XTC fires.
- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/), [`TopP`](/net/developer-reference/parameters/sampler/top-p/), [`TopK`](/net/developer-reference/parameters/sampler/top-k/) — apply before XTC.

## What's next

- [XtcProbability](/net/developer-reference/parameters/sampler/xtc-probability/) — enables XTC.
- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs.

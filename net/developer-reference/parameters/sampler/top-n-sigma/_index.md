---
weight: 80
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/top-n-sigma/
feedback: LLMNET
version: 26.5.0
title: TopNSigma
description: Top-N-sigma sampling in Aspose.LLM for .NET — keeps tokens within N standard deviations of the top logit. Experimental llama.cpp filter.
keywords:
- TopNSigma
- sigma filter
- sampler
- experimental
- standard deviation
---

`TopNSigma` is an experimental filter from recent `llama.cpp` versions. It keeps only tokens whose logit is within `N` standard deviations of the top logit, discarding the rest.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `-1.0` (disabled) |
| **Range** | `> 0` enables (typical `1.0` – `3.0`); `≤ 0` disables |
| **Category** | Advanced / experimental filter |
| **Field on** | `SamplerParameters.TopNSigma` |

## What it does

Compute the standard deviation of the logit distribution at a generation step. Take the maximum logit (`logit_max`). Keep only tokens whose logit is at least `logit_max - N × stddev`. Discard the rest.

This filter adapts automatically to distribution shape: on peaked distributions it keeps few tokens (the tail is far from the mean); on flat distributions it keeps many (the whole distribution fits within `N` sigmas).

- `TopNSigma = -1` (default) — disabled.
- `TopNSigma = 1.0` — tight; keeps only tokens very close to the top.
- `TopNSigma = 2.0` — moderate; keeps tokens within two standard deviations.
- `TopNSigma = 3.0` — wide; covers ~99.7 % of a normal distribution.

This is a newer filter and interactions with other knobs are less well-studied than classic `TopP` / `TopK`. Reserve for experimentation.

## When to change it

| Scenario | Value |
|---|---|
| Default (disabled) | `-1.0` |
| Experimental usage | `1.5` – `2.5` |

Stick with `TopP` + `TopK` + `MinP` for production unless you have a specific reason.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.TopP = 1.0f;          // disable nucleus
preset.SamplerParameters.TopK = 0;             // disable top-K
preset.SamplerParameters.TopNSigma = 2.0f;     // use sigma-based filter instead

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/) — applied before `TopNSigma`.
- [`TopP`](/net/developer-reference/parameters/sampler/top-p/) — can coexist; experimental combinations are not well-studied.
- [`TopK`](/net/developer-reference/parameters/sampler/top-k/) — can coexist.
- [`MinP`](/net/developer-reference/parameters/sampler/min-p/) — can coexist.
- [`MinKeep`](/net/developer-reference/parameters/sampler/min-keep/) — floor applies.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — bypasses `TopNSigma` when active.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [TypicalP](/net/developer-reference/parameters/sampler/typical-p/) — another experimental filter.
- [TopP](/net/developer-reference/parameters/sampler/top-p/) — the standard alternative.

---
weight: 70
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/typical-p/
feedback: LLMNET
version: 26.5.0
title: TypicalP
description: Locally-typical sampling in Aspose.LLM for .NET — keeps tokens whose log-probability is close to the expected entropy of the distribution.
keywords:
- TypicalP
- locally typical
- sampler
- entropy-aware filter
- alternative to nucleus
---

`TypicalP` enables **locally-typical sampling**. Instead of keeping the most likely tokens (`TopP`) or the top count (`TopK`), this filter keeps tokens whose log-probability is close to the expected entropy of the distribution at that step.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `-1.0` (disabled) |
| **Range** | `0.0` – `1.0` enables; `≤ 0` disables |
| **Category** | Advanced filter |
| **Field on** | `SamplerParameters.TypicalP` |

## What it does

Locally-typical sampling is an alternative tail-trimming strategy based on information theory. For each token, the engine computes how "typical" its log-probability is relative to the distribution's expected entropy. Tokens whose log-probability deviates too far from the expectation — either too high or too low — are discarded, and the engine keeps tokens whose cumulative typicality reaches `TypicalP`.

- `TypicalP = -1` (default) — disabled.
- `TypicalP = 0.95` — keep tokens covering 95 % of the typical mass.
- `TypicalP = 0.5` — keep only very-typical tokens; tighter than nucleus sampling at the same threshold.

Locally-typical sampling tends to produce output that "sounds like the training distribution" — less repetitive than greedy, less random than high-temperature sampling.

## When to change it

| Scenario | Value |
|---|---|
| Default (disabled) | `-1.0` |
| Alternative to `TopP` for naturalness | `0.95` |
| Experiment with information-theoretic sampling | `0.5` – `0.9` |

Most production workloads use `TopP` + `TopK` + `MinP`. `TypicalP` is worth trying when standard filters produce repetitive output or when research / experimentation calls for it.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.TopP = 1.0f;        // disable nucleus
preset.SamplerParameters.TypicalP = 0.95f;   // use locally-typical instead

using var api = AsposeLLMApi.Create(preset);
string reply = await api.SendMessageAsync("Tell a short story.");
Console.WriteLine(reply);
```

## Interactions

- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/) — applied before `TypicalP`.
- [`TopP`](/net/developer-reference/parameters/sampler/top-p/) — both can be active; most users pick one or the other.
- [`TopK`](/net/developer-reference/parameters/sampler/top-k/) — compatible as a count cap.
- [`MinP`](/net/developer-reference/parameters/sampler/min-p/) — compatible.
- [`MinKeep`](/net/developer-reference/parameters/sampler/min-keep/) — floor applies.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — bypasses all standard filters including `TypicalP`.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [TopP](/net/developer-reference/parameters/sampler/top-p/) — the common alternative.
- [TopNSigma](/net/developer-reference/parameters/sampler/top-n-sigma/) — another experimental filter.

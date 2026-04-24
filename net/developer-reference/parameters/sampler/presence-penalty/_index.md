---
weight: 110
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/presence-penalty/
feedback: LLMNET
version: 26.5.0
title: PresencePenalty
description: Additive penalty in Aspose.LLM for .NET applied once to any token seen in the penalty window — biases toward fresh vocabulary.
keywords:
- PresencePenalty
- sampler
- additive penalty
- vocabulary diversity
---

`PresencePenalty` is an additive penalty applied once to any token that appeared at least once in the penalty window. Unlike [`RepetitionPenalty`](/net/developer-reference/parameters/sampler/repetition-penalty/) (multiplicative) or [`FrequencyPenalty`](/net/developer-reference/parameters/sampler/frequency-penalty/) (scales with count), `PresencePenalty` fires uniformly on first appearance.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `0.0` (disabled) |
| **Range** | `0.0` = disabled, typical `0.0` – `1.0` |
| **Category** | Repetition penalty |
| **Field on** | `SamplerParameters.PresencePenalty` |

## What it does

For each token in the penalty window, subtract `PresencePenalty` from its logit before sampling. The penalty is applied **once per token** regardless of how often the token appeared.

- `0.0` (default) — disabled.
- `0.3` – `0.6` — moderate push toward fresh tokens.
- `0.8` – `1.0` — strong push; output lean heavily on new vocabulary.

`PresencePenalty` encourages broader vocabulary without caring about repetition frequency. It pairs well with topical content that should stay within a subject but use varied terms.

## When to change it

| Scenario | Value |
|---|---|
| Disabled | `0.0` (default) |
| Slightly broaden vocabulary | `0.3` |
| Strongly encourage new words | `0.6` |
| Push model to avoid any previously-seen vocabulary | `1.0` |

Unlike `RepetitionPenalty`, `PresencePenalty` does not care how many times a token has been used — only whether it has appeared. For volume-sensitive control, combine with `FrequencyPenalty`.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.PresencePenalty = 0.5f;
preset.SamplerParameters.FrequencyPenalty = 0.3f;
// Broader vocabulary; slight extra push against over-used tokens.

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`PenaltyContextSize`](/net/developer-reference/parameters/sampler/penalty-context-size/) — window over which this penalty applies.
- [`RepetitionPenalty`](/net/developer-reference/parameters/sampler/repetition-penalty/) — multiplicative alternative.
- [`FrequencyPenalty`](/net/developer-reference/parameters/sampler/frequency-penalty/) — scales with frequency; stacks with `PresencePenalty`.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [FrequencyPenalty](/net/developer-reference/parameters/sampler/frequency-penalty/) — companion count-based penalty.
- [RepetitionPenalty](/net/developer-reference/parameters/sampler/repetition-penalty/) — the multiplicative variant.

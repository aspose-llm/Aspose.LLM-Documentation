---
weight: 120
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/frequency-penalty/
feedback: LLMNET
version: 26.5.0
title: FrequencyPenalty
description: Count-based additive penalty in Aspose.LLM for .NET — subtracts FrequencyPenalty × count from the logit of each recently-seen token.
keywords:
- FrequencyPenalty
- sampler
- frequency-based penalty
- count-based penalty
---

`FrequencyPenalty` is an additive penalty proportional to how many times each token has appeared in the penalty window. The more often a token appears, the larger the penalty.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `0.0` (disabled) |
| **Range** | `0.0` = disabled, typical `0.0` – `1.0` |
| **Category** | Repetition penalty |
| **Field on** | `SamplerParameters.FrequencyPenalty` |

## What it does

For each token in the penalty window, compute `count × FrequencyPenalty` and subtract that from the token's logit. Tokens that appeared ten times get penalized ten times as hard as tokens that appeared once.

- `0.0` (default) — disabled.
- `0.1` – `0.3` — moderate; common words stay usable but over-used ones get suppressed.
- `0.5+` — aggressive; breaks repetition but risks under-generating common function words (articles, prepositions).

`FrequencyPenalty` is the finest-grained of the three penalties: [`RepetitionPenalty`](/net/developer-reference/parameters/sampler/repetition-penalty/) is binary-ish on the token, [`PresencePenalty`](/net/developer-reference/parameters/sampler/presence-penalty/) fires once per token, and `FrequencyPenalty` scales with count.

## When to change it

| Scenario | Value |
|---|---|
| Disabled | `0.0` (default) |
| Discourage over-used words | `0.2` |
| Strong anti-repetition when other penalties do not suffice | `0.5` |
| Stacks with others; tune one penalty at a time | |

When experimenting, enable **one** penalty at a time and raise until looping stops. Enabling all three at once with aggressive values produces awkward output.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.FrequencyPenalty = 0.3f;
// Over-used words get progressively suppressed.

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`PenaltyContextSize`](/net/developer-reference/parameters/sampler/penalty-context-size/) — window over which the count is measured.
- [`RepetitionPenalty`](/net/developer-reference/parameters/sampler/repetition-penalty/) — multiplicative companion.
- [`PresencePenalty`](/net/developer-reference/parameters/sampler/presence-penalty/) — uniform additive companion.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [PresencePenalty](/net/developer-reference/parameters/sampler/presence-penalty/) — companion uniform penalty.
- [RepetitionPenalty](/net/developer-reference/parameters/sampler/repetition-penalty/) — multiplicative penalty.

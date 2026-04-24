---
weight: 100
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/repetition-penalty/
feedback: LLMNET
version: 26.5.0
title: RepetitionPenalty
description: Multiplicative penalty in Aspose.LLM for .NET applied to tokens seen recently — values above 1.0 make repeats less likely.
keywords:
- RepetitionPenalty
- sampler
- penalty
- anti-repetition
- loops
---

`RepetitionPenalty` divides the logit of any token that appeared in the recent window (see [`PenaltyContextSize`](/net/developer-reference/parameters/sampler/penalty-context-size/)) by the penalty value. Values above `1.0` make recently-seen tokens less likely to be picked again.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `1.1` |
| **Range** | `1.0` = disabled, above `1.0` = active |
| **Category** | Repetition penalty |
| **Field on** | `SamplerParameters.RepetitionPenalty` |

## What it does

For each token that appeared at least once in the penalty window, divide its logit by `RepetitionPenalty` before sampling. The higher the penalty, the stronger the suppression.

- `1.0` — no penalty; behaves as if disabled.
- `1.05` – `1.15` — gentle. Breaks common loop patterns without starving the sampler of basic words.
- `1.2` – `1.3` — aggressive. Useful when the model loops persistently, but risks under-generating common words like "the" or "and".
- `1.5+` — very aggressive. The model will work hard to use different words; output quality usually drops.

## When to change it

| Scenario | Value |
|---|---|
| Disabled | `1.0` |
| Default (most chat) | `1.1` |
| Model loops occasionally | `1.15` |
| Model loops persistently | `1.2` |
| Do not change beyond 1.3 without reason | |

If raising `RepetitionPenalty` past `1.2` still does not solve looping, switch to [`DryMultiplier`](/net/developer-reference/parameters/sampler/dry-multiplier/) (DRY) — it catches phrase-level repeats that token-level penalty misses.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.RepetitionPenalty = 1.15f;
preset.SamplerParameters.PenaltyContextSize = 256;

using var api = AsposeLLMApi.Create(preset);
string reply = await api.SendMessageAsync("Describe spring in three sentences.");
Console.WriteLine(reply);
```

## Interactions

- [`PenaltyContextSize`](/net/developer-reference/parameters/sampler/penalty-context-size/) — window over which this penalty applies.
- [`PresencePenalty`](/net/developer-reference/parameters/sampler/presence-penalty/) — alternative additive penalty; can combine.
- [`FrequencyPenalty`](/net/developer-reference/parameters/sampler/frequency-penalty/) — scales with occurrence count; can combine.
- [`DryMultiplier`](/net/developer-reference/parameters/sampler/dry-multiplier/) — phrase-level anti-repetition; complementary to token-level.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — managing entropy adaptively may reduce the need for penalties.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [PresencePenalty](/net/developer-reference/parameters/sampler/presence-penalty/) and [FrequencyPenalty](/net/developer-reference/parameters/sampler/frequency-penalty/) — additive alternatives.
- [Garbled output troubleshooting](/net/troubleshooting/garbled-output/) — diagnosing repetition issues.

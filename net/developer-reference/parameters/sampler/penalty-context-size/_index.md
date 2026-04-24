---
weight: 90
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/penalty-context-size/
feedback: LLMNET
version: 26.5.0
title: PenaltyContextSize
description: Window size in Aspose.LLM for .NET over which repetition, presence, and frequency penalties are computed — how many recent tokens count.
keywords:
- PenaltyContextSize
- penalty window
- sampler
- repetition
- recency
---

`PenaltyContextSize` sets the number of recent tokens considered when applying repetition, presence, and frequency penalties. Only the last `PenaltyContextSize` tokens influence penalty calculations.

## Quick reference

| | |
|---|---|
| **Type** | `int` |
| **Default** | `-1` (use the model's full context size) |
| **Range** | `-1` or positive integer |
| **Category** | Penalty window |
| **Field on** | `SamplerParameters.PenaltyContextSize` |

## What it does

Before each sampling step, the three penalty knobs ([`RepetitionPenalty`](/net/developer-reference/parameters/sampler/repetition-penalty/), [`PresencePenalty`](/net/developer-reference/parameters/sampler/presence-penalty/), [`FrequencyPenalty`](/net/developer-reference/parameters/sampler/frequency-penalty/)) need to know which prior tokens to examine. `PenaltyContextSize` defines that window.

- `PenaltyContextSize = -1` — use the full context (equivalent to `ContextParameters.ContextSize`). Maximum recall; penalties apply across the entire conversation.
- `PenaltyContextSize = 256` — only the last 256 tokens contribute. Penalties are local; the model can freely reuse words that appeared earlier than that.
- `PenaltyContextSize = 64` — very local window; penalties essentially prevent immediate repetition only.

Narrow windows make penalties local (avoid recent verbatim repeats); wide windows make them global (avoid any mention of a token anywhere in history).

## When to change it

| Scenario | Value |
|---|---|
| Default — penalize repetition across full context | `-1` |
| Fresh-style writing that can revisit topics | `256` – `512` |
| Strict anti-repetition for short outputs | `128` |
| Very local penalty (only consecutive repeats) | `64` |

Longer conversations may benefit from a smaller penalty window so the model isn't punished for reusing common words across a long dialogue. For short-form answers, the default `-1` is usually fine.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.PenaltyContextSize = 256;
preset.SamplerParameters.RepetitionPenalty = 1.15f;
// Discourage repetition within the last 256 tokens; older history doesn't trigger the penalty.

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`RepetitionPenalty`](/net/developer-reference/parameters/sampler/repetition-penalty/) — applied within this window.
- [`PresencePenalty`](/net/developer-reference/parameters/sampler/presence-penalty/) — same window.
- [`FrequencyPenalty`](/net/developer-reference/parameters/sampler/frequency-penalty/) — same window.
- [`ContextParameters.ContextSize`](/net/developer-reference/parameters/context/context-size/) — upper bound; at `-1`, `PenaltyContextSize` equals this.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [RepetitionPenalty](/net/developer-reference/parameters/sampler/repetition-penalty/) — the main penalty that uses this window.
- [Garbled output — repetition loops](/net/troubleshooting/garbled-output/) — when penalty tuning helps.

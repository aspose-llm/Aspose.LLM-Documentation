---
weight: 170
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/dry-multiplier/
feedback: LLMNET
version: 26.5.0
title: DryMultiplier
description: DRY sampler strength in Aspose.LLM for .NET — enables the Don't Repeat Yourself filter that penalizes verbatim phrase repetition.
keywords:
- DryMultiplier
- DRY
- Don't Repeat Yourself
- phrase repetition
- sampler
---

`DryMultiplier` enables and scales the **DRY (Don't Repeat Yourself)** sampler. DRY detects when the model is about to produce a verbatim copy of a phrase it said earlier and applies an exponentially growing penalty to discourage the repeat.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `-1.0` (disabled) |
| **Range** | `> 0` enables; typical `0.5` – `1.5` |
| **Category** | Phrase-level repetition penalty |
| **Field on** | `SamplerParameters.DryMultiplier` |

## What it does

DRY scans the generation window and detects consecutive-token sequences that match what the model generated earlier. When a match extends beyond [`DryAllowedLength`](/net/developer-reference/parameters/sampler/dry-allowed-length/), it applies a penalty that grows as `DryMultiplier × DryBase^(match_length - DryAllowedLength)`.

- `DryMultiplier = -1` (default) — disabled.
- `DryMultiplier = 0.8` — classic strength recommended by the DRY paper.
- `DryMultiplier = 1.5+` — aggressive; risks distorting natural common phrases.

DRY is phrase-level, complementary to token-level [`RepetitionPenalty`](/net/developer-reference/parameters/sampler/repetition-penalty/). Use it when output has entire paragraphs or sentences echoing earlier text — token-level penalties cannot catch that.

## When to change it

| Scenario | Value |
|---|---|
| Default (disabled) | `-1.0` |
| Creative writing, stop phrase repeats | `0.8` |
| Persistent phrase loops | `1.2` |
| Code generation | Leave disabled (code naturally repeats syntax) |

DRY is often the right tool when [`RepetitionPenalty`](/net/developer-reference/parameters/sampler/repetition-penalty/) tuning up to `1.2` doesn't fix phrase-level loops. Enable DRY and tune `DryMultiplier` + [`DryAllowedLength`](/net/developer-reference/parameters/sampler/dry-allowed-length/).

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.DryMultiplier = 0.8f;
preset.SamplerParameters.DryBase = 1.75f;
preset.SamplerParameters.DryAllowedLength = 3;

using var api = AsposeLLMApi.Create(preset);
string reply = await api.SendMessageAsync("Write an essay about patience.");
Console.WriteLine(reply);
```

## Interactions

- [`DryBase`](/net/developer-reference/parameters/sampler/dry-base/) — exponent base for growing penalty.
- [`DryAllowedLength`](/net/developer-reference/parameters/sampler/dry-allowed-length/) — minimum match length before penalty fires.
- [`DryPenaltyLastN`](/net/developer-reference/parameters/sampler/dry-penalty-last-n/) — how far back to look.
- [`DrySequenceBreakers`](/net/developer-reference/parameters/sampler/dry-sequence-breakers/) — tokens that reset the match detector.
- [`RepetitionPenalty`](/net/developer-reference/parameters/sampler/repetition-penalty/) — token-level companion; both can be active.

## What's next

- [DryBase](/net/developer-reference/parameters/sampler/dry-base/), [DryAllowedLength](/net/developer-reference/parameters/sampler/dry-allowed-length/) — the other DRY knobs.
- [RepetitionPenalty](/net/developer-reference/parameters/sampler/repetition-penalty/) — token-level alternative.
- [Garbled output troubleshooting](/net/troubleshooting/garbled-output/) — when to reach for DRY.

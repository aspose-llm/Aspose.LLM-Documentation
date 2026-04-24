---
weight: 200
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/dry-penalty-last-n/
feedback: LLMNET
version: 26.5.0
title: DryPenaltyLastN
description: Window of recent tokens scanned by DRY in Aspose.LLM for .NET — 0 means scan everything; positive values limit the look-back distance.
keywords:
- DryPenaltyLastN
- DRY
- sampler
- look-back window
---

`DryPenaltyLastN` controls how far back DRY looks when scanning for phrase repetition. It determines the size of the generation window scanned for matches.

## Quick reference

| | |
|---|---|
| **Type** | `int` |
| **Default** | `0` (scan all available context) |
| **Range** | `0` = unlimited; `> 0` = window size |
| **Category** | Phrase-level repetition penalty |
| **Field on** | `SamplerParameters.DryPenaltyLastN` |

## What it does

DRY scans the last `DryPenaltyLastN` tokens of generation for consecutive-token sequences matching whatever the model is about to emit.

- `DryPenaltyLastN = 0` (default) — scan all available tokens; no distance limit.
- `DryPenaltyLastN = 512` — only the last 512 tokens are scanned for matches. Older content is ignored.
- `DryPenaltyLastN = 128` — very local; catches only immediate phrase repeats.

Narrowing the window speeds up DRY scanning slightly and relaxes the constraint for long conversations where older repeats are benign.

## When to change it

| Scenario | Value |
|---|---|
| Default (scan all) | `0` |
| Long conversations, focus DRY on recent output | `512` |
| Very local enforcement | `128` |

Most deployments leave `DryPenaltyLastN = 0`. Raise only if DRY is catching repeats from very old conversation history that are acceptable in your domain.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.DryMultiplier = 0.8f;
preset.SamplerParameters.DryPenaltyLastN = 512;  // only scan last 512 tokens

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`DryMultiplier`](/net/developer-reference/parameters/sampler/dry-multiplier/) — gate; `DryPenaltyLastN` only matters when DRY is active.
- Other DRY knobs operate within this window.

## What's next

- [DryMultiplier](/net/developer-reference/parameters/sampler/dry-multiplier/) — enables DRY.
- [DrySequenceBreakers](/net/developer-reference/parameters/sampler/dry-sequence-breakers/) — reset tokens.
- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs.

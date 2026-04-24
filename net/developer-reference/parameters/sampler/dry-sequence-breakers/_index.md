---
weight: 210
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/dry-sequence-breakers/
feedback: LLMNET
version: 26.5.0
title: DrySequenceBreakers
description: Tokens that reset DRY's match detector in Aspose.LLM for .NET — default list covers newlines, colons, quotes, and asterisks.
keywords:
- DrySequenceBreakers
- DRY
- sampler
- sequence breakers
- reset
---

`DrySequenceBreakers` is the list of string tokens that reset DRY's phrase-match detector. When DRY encounters one of these tokens, any partial match in progress is discarded.

## Quick reference

| | |
|---|---|
| **Type** | `List<string>` |
| **Default** | `["\n", ":", "\"", "*"]` |
| **Category** | Phrase-level repetition penalty |
| **Field on** | `SamplerParameters.DrySequenceBreakers` |

## What it does

DRY detects long matches by comparing the token stream against earlier tokens. When the detector encounters a "breaker" string — typically punctuation or structural markup — it treats that as the end of one phrase and the start of the next. This prevents DRY from penalizing tokens that naturally repeat across paragraph boundaries.

Default breakers:

- `"\n"` — newline: paragraph boundary.
- `":"` — colon: list-item or header boundary.
- `"\""` — double quote: start/end of quoted text.
- `"*"` — asterisk: markdown emphasis or list markers.

## When to change it

| Scenario | Value |
|---|---|
| Default list | `["\n", ":", "\"", "*"]` |
| Markdown-heavy output | Add `"#"` for headings, `"-"` for bullets |
| Code generation | Add `"{"`, `"}"`, `";"`, `"("`, `")"` |
| Minimal breakers — let DRY fire more aggressively across paragraphs | `["\n"]` or empty list |

Breakers shape what DRY considers a "phrase". Narrowing the list makes DRY more global; widening it localizes enforcement.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.DryMultiplier = 0.8f;
preset.SamplerParameters.DrySequenceBreakers = new List<string>
{
    "\n", ":", "\"", "*", "#", "-", // markdown-heavy content
};

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`DryMultiplier`](/net/developer-reference/parameters/sampler/dry-multiplier/) — DRY gate.
- [`DryAllowedLength`](/net/developer-reference/parameters/sampler/dry-allowed-length/) — match length threshold within a phrase boundary.

## What's next

- [DryMultiplier](/net/developer-reference/parameters/sampler/dry-multiplier/) — enables DRY.
- [DryAllowedLength](/net/developer-reference/parameters/sampler/dry-allowed-length/) — length threshold.
- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs.

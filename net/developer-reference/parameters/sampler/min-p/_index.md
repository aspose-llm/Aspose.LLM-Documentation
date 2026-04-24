---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/min-p/
feedback: LLMNET
version: 26.5.0
title: MinP
description: Minimum-probability truncation in Aspose.LLM for .NET — keeps only tokens whose probability is at least MinP times the top token's probability.
keywords:
- MinP
- sampler
- truncation
- minimum probability
---

`MinP` defines a minimum probability threshold **relative to the top token**. Any candidate whose probability is below `MinP × p(top)` is discarded.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `0.05` |
| **Range** | `0.0` – `1.0` |
| **Category** | Core sampling |
| **Field on** | `SamplerParameters.MinP` |

## What it does

After the other filters run, compute the top candidate's probability `p_max`. For every remaining token, check if its probability is at least `MinP × p_max`. If not, discard.

- `MinP = 0.0` — filter disabled.
- `MinP = 0.05` (default) — keep tokens at 5 % of the top token's probability or higher.
- `MinP = 0.1` — keep tokens at 10 % or higher (stricter).

Unlike `TopP` (which is cumulative-mass aware) and `TopK` (which is count aware), `MinP` is relative-to-top aware. It adapts to distribution shape differently: on a peaked distribution it keeps fewer tokens; on a flat distribution it keeps more.

## When to change it

| Scenario | Value |
|---|---|
| Disabled | `0.0` |
| Loose tail retention | `0.02` |
| Default balance | `0.05` |
| Conservative, drops more tail | `0.1` – `0.15` |

Some users recommend `MinP` as a replacement for `TopP` — simpler to reason about, less sensitive to vocabulary size. The default `0.05` is a conservative, broadly safe value.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.MinP = 0.1f; // stricter tail cutoff

using var api = AsposeLLMApi.Create(preset);
string reply = await api.SendMessageAsync("Describe today's weather in three words.");
Console.WriteLine(reply);
```

Creative writing with wider tail:

```csharp
preset.SamplerParameters.MinP = 0.02f;
preset.SamplerParameters.Temperature = 0.9f;
// More diverse sampling, still filtered against very-low-probability tokens.
```

## Interactions

- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/) — applied before `MinP`.
- [`TopP`](/net/developer-reference/parameters/sampler/top-p/) — can coexist; final candidate set respects both.
- [`TopK`](/net/developer-reference/parameters/sampler/top-k/) — count-based cap; stacks with `MinP`.
- [`MinKeep`](/net/developer-reference/parameters/sampler/min-keep/) — floor; `MinP` never cuts below `MinKeep`.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — bypasses `MinP` when active.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [TopP](/net/developer-reference/parameters/sampler/top-p/) — cumulative-mass cousin of `MinP`.
- [TopK](/net/developer-reference/parameters/sampler/top-k/) — count-based cap.

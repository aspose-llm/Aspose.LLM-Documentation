---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/top-k/
feedback: LLMNET
version: 26.5.0
title: TopK
description: Hard cap on candidate token count in Aspose.LLM for .NET — keeps only the top K highest-probability tokens at each step.
keywords:
- TopK
- top-K sampling
- sampler
- candidate count
- truncation
---

`TopK` caps the number of candidate tokens the sampler considers at each step. After `Temperature`, the engine sorts tokens by probability and keeps only the top `TopK` entries.

## Quick reference

| | |
|---|---|
| **Type** | `int` (documented as `float` in the SDK class, but used as integer count) |
| **Default** | `40` |
| **Range** | `0` disables; `1` = greedy; typical `10` – `100` |
| **Category** | Core sampling |
| **Field on** | `SamplerParameters.TopK` |

## What it does

Sort the distribution by probability descending. Take the first `TopK` tokens. Discard everything else. The remaining tokens are renormalized and passed downstream.

- `TopK = 0` — filter is disabled (or interpreted as unlimited by `llama.cpp`).
- `TopK = 1` — greedy. Only the single highest-probability token survives regardless of distribution shape.
- `TopK = 40` (default) — a cap that leaves room for diversity while excluding the long tail.

Unlike `TopP`, `TopK` is probability-blind — it always keeps exactly `TopK` tokens, even if the distribution is tightly peaked on one token and 39 runner-ups are all nearly zero.

## When to change it

| Scenario | Value |
|---|---|
| Disabled (rely on `TopP` + `MinP`) | `0` (or set very large, e.g., `100000`) |
| Greedy-like, single candidate | `1` |
| Very conservative | `10` – `20` |
| Balanced default | `40` |
| Creative, wide pool | `80` – `100` |

Most users leave `TopK = 40` and tune `TopP` instead. `TopK` is a safety net that prevents the sampler from ever considering extremely rare tokens.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.TopK = 20; // conservative candidate pool

using var api = AsposeLLMApi.Create(preset);
string reply = await api.SendMessageAsync("Write one short haiku about rain.");
Console.WriteLine(reply);
```

Combining with `TopP` for fine control:

```csharp
preset.SamplerParameters.TopK = 40;
preset.SamplerParameters.TopP = 0.9f;
// Final candidate set is the intersection — whichever is stricter per step.
```

## Interactions

- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/) — applied before `TopK`.
- [`TopP`](/net/developer-reference/parameters/sampler/top-p/) — both active filters stack; sampling happens over the intersection.
- [`MinP`](/net/developer-reference/parameters/sampler/min-p/) — can further thin the candidate pool.
- [`MinKeep`](/net/developer-reference/parameters/sampler/min-keep/) — floor; `TopK` never cuts below `MinKeep`.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — bypasses `TopK` when active.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [TopP](/net/developer-reference/parameters/sampler/top-p/) — the probability-aware counterpart.
- [MinP](/net/developer-reference/parameters/sampler/min-p/) — another truncation strategy.

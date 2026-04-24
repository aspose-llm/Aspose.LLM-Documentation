---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/top-p/
feedback: LLMNET
version: 26.5.0
title: TopP
description: Nucleus sampling threshold in Aspose.LLM for .NET — keeps only the smallest set of tokens whose cumulative probability exceeds TopP.
keywords:
- TopP
- nucleus sampling
- sampler
- truncation
- cumulative probability
---

`TopP` implements **nucleus sampling**. At each generation step, tokens are sorted by probability and the engine keeps only the smallest set whose cumulative probability is at least `TopP`. Tokens outside that nucleus are discarded before sampling.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `0.9` |
| **Range** | `0.0` – `1.0` (values ≥ `1.0` disable the filter) |
| **Category** | Core sampling |
| **Field on** | `SamplerParameters.TopP` |

## What it does

After `Temperature` scales the distribution, sort tokens by probability descending. Walk the sorted list accumulating probability. Stop when the running sum reaches `TopP`. Every token past that cutoff is removed; the remaining tokens are renormalized and sampled from.

- At `TopP = 1.0`, no tokens are removed — the filter is effectively off.
- At `TopP = 0.9` (default), the engine keeps ~90 % of the probability mass. On a peaked distribution this is 2-5 tokens; on a flat distribution it can be 50+.
- At `TopP = 0.5`, only the dominant half of the mass survives — tighter, more deterministic output.

`TopP` is probability-aware: on a confident step (one token at 0.95 probability) it keeps only that one token; on an uncertain step (many comparable tokens) it keeps a larger set.

## When to change it

| Scenario | Value |
|---|---|
| Disabled (rely on `TopK` + `MinP`) | `1.0` |
| Creative output with some variety | `0.95` |
| Balanced general-purpose chat (default) | `0.9` |
| Conservative, precise output | `0.7` – `0.8` |
| Very tight, near-greedy | `0.5` – `0.6` |

Lower `TopP` excludes more of the tail — output is more predictable but less varied. The default `0.9` is a broadly accepted balance.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.TopP = 0.95f; // keep a bit more of the tail for creativity

using var api = AsposeLLMApi.Create(preset);
string reply = await api.SendMessageAsync("Suggest a name for a coffee shop.");
Console.WriteLine(reply);
```

For precision work:

```csharp
preset.SamplerParameters.Temperature = 0.2f;
preset.SamplerParameters.TopP = 0.8f;
// Narrow both dimensions — precise and deterministic.
```

## Interactions

- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/) — applied before `TopP`. Very low `Temperature` + loose `TopP` still produces near-greedy output.
- [`TopK`](/net/developer-reference/parameters/sampler/top-k/) — stacks with `TopP`. The final candidate set is the intersection.
- [`MinP`](/net/developer-reference/parameters/sampler/min-p/) — complements `TopP` on the tail; both can be active.
- [`TypicalP`](/net/developer-reference/parameters/sampler/typical-p/) — alternative to `TopP` based on local typicality.
- [`MinKeep`](/net/developer-reference/parameters/sampler/min-keep/) — floor on candidate count; `TopP` never cuts below `MinKeep`.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — bypasses `TopP` when active.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [Temperature](/net/developer-reference/parameters/sampler/temperature/) — the partner knob that runs before `TopP`.
- [MinP](/net/developer-reference/parameters/sampler/min-p/) — another probability-relative cutoff.

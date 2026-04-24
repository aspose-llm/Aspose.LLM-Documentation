---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/temperature/
feedback: LLMNET
version: 26.5.0
title: Temperature
description: Sampler temperature in Aspose.LLM for .NET — scales logits before sampling, from deterministic (0.0) to high-randomness (>1.0).
keywords:
- Temperature
- sampler
- randomness
- creativity
- determinism
---

`Temperature` scales the model's logits (unnormalized probabilities) before sampling. Lower temperature sharpens the distribution toward the most likely token; higher temperature flattens it, making rare tokens more competitive.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `0.7` |
| **Range** | `0.0` and above (typical `0.0` – `1.5`) |
| **Category** | Core sampling |
| **Field on** | `SamplerParameters.Temperature` |

## What it does

Each generation step produces a vector of logits — one value per vocabulary token. The engine divides every logit by `Temperature` before applying softmax:

- At `Temperature = 1.0`, the softmax is unchanged; the model samples from its native distribution.
- Below `1.0`, differences between logits are magnified. The top tokens become more likely; rare tokens are suppressed. At the limit `Temperature = 0.0`, the engine picks the single highest-logit token every step (greedy decoding).
- Above `1.0`, logits flatten. The top token's advantage shrinks; tail tokens gain relative probability. At high temperatures, output becomes increasingly random.

`Temperature` runs before all downstream filters (`TopP`, `TopK`, `MinP`). A combination of low temperature and tight `TopP` produces very deterministic output; high temperature with wide filters produces creative, unpredictable output.

## When to change it

| Scenario | Value |
|---|---|
| Fully deterministic, reproducible output | `0.0` (greedy; `Seed` becomes irrelevant) |
| Precise tasks — code, structured data, classification | `0.1` – `0.3` |
| General-purpose chat (default balance) | `0.7` |
| Creative writing, brainstorming | `0.8` – `1.0` |
| Maximum variety, risk of incoherence | `> 1.0` |

Temperature trades accuracy for variety. On factual questions, high temperature increases hallucination rate. On creative tasks, low temperature produces dull, repetitive output.

## Example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen25Preset();
preset.SamplerParameters.Temperature = 0.2f; // precise, low-variety output

using var api = AsposeLLMApi.Create(preset);
string reply = await api.SendMessageAsync("List three primes under 20.");
Console.WriteLine(reply);
// Output: 2, 3, 5.
```

For a fully reproducible run:

```csharp
preset.SamplerParameters.Temperature = 0.0f;
preset.SamplerParameters.Seed = 42; // seed is ignored when Temperature == 0
```

## Interactions

- [`TopP`](/net/developer-reference/parameters/sampler/top-p/) — applies after temperature to trim the tail.
- [`TopK`](/net/developer-reference/parameters/sampler/top-k/) — hard cap on candidate count after temperature.
- [`MinP`](/net/developer-reference/parameters/sampler/min-p/) — minimum relative probability after temperature.
- [`Seed`](/net/developer-reference/parameters/sampler/seed/) — irrelevant when `Temperature = 0`.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — when active, adaptively adjusts entropy and bypasses `Temperature` tuning.
- [`DynatempRange`](/net/developer-reference/parameters/sampler/dynatemp-range/) — dynamically varies `Temperature` per step based on entropy.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [Tune for speed vs quality](/net/how-to/tune-for-speed-vs-quality/) — where `Temperature` sits in the trade-off.
- [Garbled output troubleshooting](/net/troubleshooting/garbled-output/) — when high temperature causes incoherence.

---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/seed/
feedback: LLMNET
version: 26.5.0
title: Seed
description: Random seed for sampling in Aspose.LLM for .NET — fix an integer for reproducible output, or use the sentinel for time-based randomness.
keywords:
- Seed
- reproducibility
- random
- determinism
- testing
---

`Seed` controls the random number generator used for stochastic sampling. A fixed integer makes generation reproducible — the same prompt with the same model and the same parameters produces the same output. The default sentinel maps to a time-based seed, giving non-deterministic output on every run.

## Quick reference

| | |
|---|---|
| **Type** | `int` |
| **Default** | `0xFFFFFFFF` (sentinel — time-based seed inside llama.cpp) |
| **Range** | Any `int` value; default sentinel is a signed-int reinterpretation of `0xFFFFFFFF` |
| **Category** | Core sampling |
| **Field on** | `SamplerParameters.Seed` |

## What it does

`Seed` initializes the sampler's pseudo-random number generator. The RNG drives every stochastic choice — which token to pick from the candidate pool when multiple tokens survive filtering.

- A specific integer (`42`, `12345`, etc.) — deterministic. Two runs with the same seed and identical parameters produce identical token sequences.
- The default sentinel `0xFFFFFFFF` — `llama.cpp` substitutes a time-based seed. Output varies per run.

`Seed` only matters when sampling is actually stochastic. At `Temperature = 0` the sampler is greedy (always picks the top token), and `Seed` has no effect.

## When to change it

| Scenario | Value |
|---|---|
| Default non-deterministic behavior | `unchecked((int)0xFFFFFFFF)` (default) |
| Reproducible tests and benchmarks | A fixed `int`, e.g., `42` |
| Per-request seeding (rotate value per request) | Pass a computed `int` from your caller |

Set `Seed` when writing tests that assert on specific output, or when building regression suites that catch sampling drift across SDK versions.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.Temperature = 0.7f;
preset.SamplerParameters.Seed = 42;

using var api = AsposeLLMApi.Create(preset);
string reply1 = await api.SendMessageAsync("Write one short sentence about rain.");
string reply2 = await api.SendMessageAsync("Write one short sentence about rain.");
// reply1 and reply2 are identical because Seed is fixed and history is the same.
```

For deterministic single-request output:

```csharp
preset.SamplerParameters.Temperature = 0.0f;
// Temperature 0 = greedy; Seed becomes irrelevant.
```

## Interactions

- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/) — at `Temperature = 0`, `Seed` has no effect.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — uses its own adaptive process; `Seed` still affects the underlying RNG but the entropy target dominates.
- Session history — a fixed `Seed` only produces identical output when the entire preceding history is identical. A fresh session and a loaded session with the same history will both honor the seed.

## Notes

Reproducibility is deterministic within the same SDK version and `BinaryManagerParameters.ReleaseTag`. Upgrading the SDK or the llama.cpp release tag may shift token sequences slightly even with the same `Seed`.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs at a glance.
- [Temperature](/net/developer-reference/parameters/sampler/temperature/) — the primary randomness knob.
- [Session persistence portability](/net/developer-reference/session-persistence/portability/) — interaction between seeding and restored sessions.

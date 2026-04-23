---
weight: 65
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/how-to/
feedback: LLMNET
version: 26.5.0
title: How-to recipes
description: Short focused recipes for Aspose.LLM for .NET — model picking, quantization primer, speed-vs-quality tuning, cancellation, first-token latency, memory estimation.
keywords:
- how-to
- recipe
- quantization
- speed
- quality
- cancellation
- latency
- memory
---

Task-oriented recipes. Each page answers one specific question with a concise, actionable solution — a lookup, a short formula, or a two-to-three step procedure. Recipes avoid full runnable projects; for that level of detail, see [Use cases](/net/use-cases/).

## How this section is organized

| You want… | Go to |
|---|---|
| A decision (which model, which quantization, what settings) | A how-to recipe |
| A running project you can build and execute | A [use case](/net/use-cases/) |
| Definition and behavior of a specific API or parameter | The [developer reference](/net/developer-reference/) |
| A compact pattern to copy into existing code | A [quick win](/net/quick-wins/) |

How-tos are deliberately short. If a recipe grows past ~200 lines, it is promoted to a full use case instead.

## Recipes by theme

### Choosing your setup

- [Select a model by task](/net/how-to/select-model-by-task/) — decision table over the nine built-in text presets and four vision presets.
- [Understand quantization](/net/how-to/understand-quantization/) — Q4 vs Q5 vs Q8 vs F16 primer; what each letter means; when to pick which.
- [Estimate memory requirements](/net/how-to/estimate-memory-requirements/) — simple formula for RAM / VRAM given model size, quantization, and context length.

### Tuning behavior

- [Tune for speed vs quality](/net/how-to/tune-for-speed-vs-quality/) — concrete sampler / context / model-size dials for the speed-quality trade-off.
- [Reduce first-token latency](/net/how-to/reduce-first-token-latency/) — why the first token is slow and the five patterns that cut the time.

### Runtime control

- [Handle cancellation](/net/how-to/handle-cancellation/) — cancel an in-flight generation cleanly; session state after cancel; common pitfalls with `CancellationToken`.

## When a recipe does not match

Start with the [symptom shortcut table in troubleshooting](/net/troubleshooting/#symptom--page-shortcut) if you are debugging a concrete failure, or the [parameter reference](/net/developer-reference/parameters/) if you need comprehensive detail on a specific knob.

For deeper questions, the [Aspose Support Forum](https://forum.aspose.com/) is the canonical place to ask.

## What's next

- [Use cases](/net/use-cases/) — full running projects organized by scenario.
- [Quick wins](/net/quick-wins/) — compact copy-paste snippets for common first-time tasks.
- [Developer's reference](/net/developer-reference/) — conceptual reference for presets, parameters, sessions, and APIs.
- [Troubleshooting](/net/troubleshooting/) — diagnose and fix known problems.

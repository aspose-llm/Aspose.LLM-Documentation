---
weight: 260
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/enable-infill/
feedback: LLMNET
version: 26.5.0
title: EnableInfill
description: Fill-in-the-middle infill sampler in Aspose.LLM for .NET — specialized mode for code completion models that support it.
keywords:
- EnableInfill
- infill
- fill-in-the-middle
- code completion
- sampler
---

`EnableInfill` switches the sampler to the INFILL variant used for fill-in-the-middle code completion. Only useful for models trained for FIM (code-completion-oriented) workflows.

## Quick reference

| | |
|---|---|
| **Type** | `bool` |
| **Default** | `false` |
| **Category** | Specialized sampling |
| **Field on** | `SamplerParameters.EnableInfill` |

## What it does

When `true`, the engine uses a specialized sampler variant tuned for completing a gap between a prefix and a suffix. This is the FIM (Fill-in-the-Middle) pattern used by code models trained with FIM tokens (for example, some DeepSeek-Coder or StarCoder derivatives).

- `false` (default) — standard chat sampler. Correct for all text presets and vision presets.
- `true` — INFILL sampler. Only enables when the model supports FIM.

## When to change it

| Scenario | Value |
|---|---|
| Standard chat, reasoning, or generation | `false` |
| Code completion with FIM-trained model | `true` |

Most users will leave this `false`. Enable only when you explicitly use a FIM-trained code model and understand the prefix/suffix prompt format that model expects.

## Example

```csharp
var preset = new DeepSeekCoder2Preset();
preset.SamplerParameters.EnableInfill = true;

using var api = AsposeLLMApi.Create(preset);
// Use a prompt that follows the FIM template for your specific model.
```

## Interactions

Independent of other sampler knobs. `EnableInfill` toggles which sampler implementation runs; all filter knobs still apply within the chosen sampler.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs.
- [Custom preset](/net/use-cases/custom-preset/) — patterns for building presets around specialized models.

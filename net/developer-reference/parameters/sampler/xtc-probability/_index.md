---
weight: 150
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/xtc-probability/
feedback: LLMNET
version: 26.5.0
title: XtcProbability
description: XTC sampler activation probability in Aspose.LLM for .NET — chance that the eXclude Top Choices filter runs at each step to inject diversity.
keywords:
- XtcProbability
- XTC
- Exclude Top Choices
- sampler
- diversity
---

`XtcProbability` enables the **XTC (eXclude Top Choices)** sampler. With probability `XtcProbability` at each step, XTC removes the top candidate tokens, forcing the engine to sample from the tail. Useful for injecting diversity without raising temperature.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `-1.0` (disabled) |
| **Range** | `0.0` – `1.0`, or `-1.0` to disable |
| **Category** | Advanced / diversity |
| **Field on** | `SamplerParameters.XtcProbability` |

## What it does

At each generation step, with probability `XtcProbability`, XTC fires: the engine excludes all tokens whose probability is above [`XtcThreshold`](/net/developer-reference/parameters/sampler/xtc-threshold/), and samples from whatever remains. This nudges the model into less-obvious paths without changing temperature.

- `-1.0` (default) — disabled.
- `0.1` – `0.3` — XTC fires on 10–30 % of steps. Mild diversity boost.
- `0.5` – `0.8` — XTC fires often. Strong variety; risk of incoherence.

## When to change it

| Scenario | Value |
|---|---|
| Default (disabled) | `-1.0` |
| Light creativity boost | `0.1` – `0.2` |
| Alternative to high temperature | `0.3` – `0.5` |
| Experimental variety push | `0.5+` |

XTC is a recent addition. Prefer conventional [`Temperature`](/net/developer-reference/parameters/sampler/temperature/) + [`TopP`](/net/developer-reference/parameters/sampler/top-p/) tuning first. Reach for XTC when you want variety specifically when the model is confident — standard sampling tightens those steps; XTC breaks them.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.XtcProbability = 0.2f;
preset.SamplerParameters.XtcThreshold = 0.1f;
// On ~20 % of steps, tokens above 0.1 probability are excluded.

using var api = AsposeLLMApi.Create(preset);
string reply = await api.SendMessageAsync("Suggest three fictional startup names.");
Console.WriteLine(reply);
```

## Interactions

- [`XtcThreshold`](/net/developer-reference/parameters/sampler/xtc-threshold/) — minimum probability a token must have to be considered for exclusion.
- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/) — orthogonal. XTC works at any temperature.
- [`TopP`](/net/developer-reference/parameters/sampler/top-p/) — still applied; XTC runs within what `TopP` keeps.

## What's next

- [XtcThreshold](/net/developer-reference/parameters/sampler/xtc-threshold/) — the probability cutoff companion.
- [Temperature](/net/developer-reference/parameters/sampler/temperature/) — conventional randomness knob.
- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs.

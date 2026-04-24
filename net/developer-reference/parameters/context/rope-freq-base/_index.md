---
weight: 80
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/rope-freq-base/
feedback: LLMNET
version: 26.5.0
title: RopeFreqBase
description: Override of RoPE base frequency in Aspose.LLM for .NET — null or 0 uses the model's trained value.
keywords:
- RopeFreqBase
- RoPE base frequency
- theta
- position encoding
---

`RopeFreqBase` is the base frequency (often denoted θ, theta) used in RoPE's positional encoding. Overriding it changes how position indices map to rotation angles. Most users leave this at the model default.

## Quick reference

| | |
|---|---|
| **Type** | `float?` |
| **Default** | `null` (use model metadata; equivalent to the value from the GGUF file) |
| **Range** | Typical `10000.0` – `10000000.0` |
| **Category** | Position encoding |
| **Field on** | `ContextParameters.RopeFreqBase` |

## What it does

In RoPE, each attention head's frequency vector is built from `freq_base^(-k/d)` for `k` across dimensions. The default `freq_base = 10000.0` works for contexts in the tens of thousands of tokens. Very long contexts sometimes use much larger bases — for example, `RopeFreqBase = 1_000_000` for 128K-trained models.

- `null` (default) — use the model's trained value from GGUF metadata.
- A positive float — override. Use only when you know the correct value for your target context.

Built-in presets leave this as `null` — the metadata is usually correct.

## When to change it

| Scenario | Value |
|---|---|
| Default (recommended) | `null` |
| Running a model in a non-standard context regime | Set per upstream model card |
| Extending a short-context model with hand-tuned RoPE | Non-trivial — refer to papers |

Changing `RopeFreqBase` without understanding the effect usually degrades quality. Prefer [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) approaches for context extension.

## Example

```csharp
var preset = new Qwen25Preset();
preset.ContextParameters.RopeFreqBase = 1_000_000f;  // only if the model documents this
```

## Interactions

- [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) — overall scaling algorithm.
- [`RopeFreqScale`](/net/developer-reference/parameters/context/rope-freq-scale/) — multiplicative scaler applied on top.

## What's next

- [RopeScalingType](/net/developer-reference/parameters/context/rope-scaling-type/) — algorithm selector.
- [RopeFreqScale](/net/developer-reference/parameters/context/rope-freq-scale/) — scale factor.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

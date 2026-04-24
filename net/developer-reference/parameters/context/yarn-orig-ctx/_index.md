---
weight: 140
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/yarn-orig-ctx/
feedback: LLMNET
version: 26.5.0
title: YarnOrigCtx
description: Original trained context length for YaRN scaling in Aspose.LLM for .NET — set this to the model's native context when extending beyond it.
keywords:
- YarnOrigCtx
- YaRN
- original context
- training window
---

`YarnOrigCtx` tells YaRN the model's **original** trained context length. The scaling factor is derived from the ratio between the target [`ContextSize`](/net/developer-reference/parameters/context/context-size/) and `YarnOrigCtx`. This is the most commonly set YaRN knob.

## Quick reference

| | |
|---|---|
| **Type** | `uint?` |
| **Default** | `null` (use model default from GGUF metadata) |
| **Range** | Positive integer; the model's trained max context |
| **Category** | YaRN position encoding |
| **Field on** | `ContextParameters.YarnOrigCtx` |

## What it does

YaRN extends context by a factor `ContextSize / YarnOrigCtx`. If the model was trained at 8K and you target 32K, the scale factor is `32768 / 8192 = 4`. YaRN's quality depends on knowing both values accurately.

- `null` (default) — YaRN reads the value from the GGUF metadata. This is what most presets use.
- Specific integer — override. Set to the model's actual training context length when the metadata is missing or wrong.

## When to change it

| Scenario | Value |
|---|---|
| Default (GGUF has correct metadata) | `null` |
| Custom GGUF without trained-context metadata | Set to the model's native context |
| Running an older model at extended context | Set to the original training window |

For a Qwen 2.5 7B model (trained at 32K) targeting 128K:

```csharp
preset.ContextParameters.ContextSize = 131072;
preset.ContextParameters.RopeScalingType = RopeScalingType.Yarn;
preset.ContextParameters.YarnOrigCtx = 32768;  // Qwen2.5 training context
```

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ContextParameters.ContextSize = 65536;
preset.ContextParameters.RopeScalingType = RopeScalingType.Yarn;
preset.ContextParameters.YarnOrigCtx = 32768;

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — target; scale = `ContextSize / YarnOrigCtx`.
- [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) — must be `Yarn`.
- Other YaRN knobs ([`YarnExtFactor`](/net/developer-reference/parameters/context/yarn-ext-factor/), [`YarnAttnFactor`](/net/developer-reference/parameters/context/yarn-attn-factor/), [`YarnBetaFast`](/net/developer-reference/parameters/context/yarn-beta-fast/), [`YarnBetaSlow`](/net/developer-reference/parameters/context/yarn-beta-slow/)) — usually null; model defaults work.

## What's next

- [RopeScalingType](/net/developer-reference/parameters/context/rope-scaling-type/) — enables YaRN.
- [ContextSize](/net/developer-reference/parameters/context/context-size/) — the extended target.
- [Long context tuning](/net/use-cases/long-context-tuning/) — full recipe.

---
weight: 110
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/yarn-attn-factor/
feedback: LLMNET
version: 26.5.0
title: YarnAttnFactor
description: YaRN magnitude scaling factor in Aspose.LLM for .NET — scales attention magnitudes at long positions; leave null to use the model default.
keywords:
- YarnAttnFactor
- YaRN
- attention scaling
- RoPE
---

`YarnAttnFactor` scales attention logit magnitudes as part of the YaRN algorithm. It compensates for the attention-softmax becoming too flat at extreme positions. Relevant only when [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) is `Yarn`.

## Quick reference

| | |
|---|---|
| **Type** | `float?` |
| **Default** | `null` (use model default) |
| **Range** | Typical `1.0` – `1.5` |
| **Category** | YaRN position encoding |
| **Field on** | `ContextParameters.YarnAttnFactor` |

## What it does

As positions grow beyond the training window, YaRN mathematically applies a scaling to attention magnitudes. `YarnAttnFactor` controls this. The YaRN paper derives a value like `0.1 × log(scale) + 1.0` as a reasonable choice; the model's metadata usually carries the correct value.

- `null` — use model default (recommended).
- Specific float — override.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Research / YaRN tuning | Per YaRN paper formula |

Rarely touched in production.

## Example

```csharp
var preset = new Llama32Preset();
preset.ContextParameters.RopeScalingType = RopeScalingType.Yarn;
// preset.ContextParameters.YarnAttnFactor = null; // default — from model
```

## Interactions

- [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) — must be `Yarn`.
- Other YaRN knobs operate together.

## What's next

- [YarnOrigCtx](/net/developer-reference/parameters/context/yarn-orig-ctx/) — the primary YaRN field you might touch.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

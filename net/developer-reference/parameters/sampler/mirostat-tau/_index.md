---
weight: 230
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/mirostat-tau/
feedback: LLMNET
version: 26.5.0
title: MirostatTau
description: Mirostat target entropy in Aspose.LLM for .NET — the entropy (in nats) Mirostat tries to maintain; lower values produce more deterministic output.
keywords:
- MirostatTau
- Mirostat
- target entropy
- sampler
---

`MirostatTau` is the target entropy Mirostat tries to maintain across the output. Mirostat adjusts its sampling parameters on the fly to keep the actual output entropy close to this value.

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `5.0` |
| **Range** | `> 0`; typical `3.0` – `8.0` |
| **Category** | Adaptive sampler |
| **Field on** | `SamplerParameters.MirostatTau` |

## What it does

Entropy is measured in nats (natural-log units) over the per-step token distribution. Lower target entropy → tighter, more deterministic output. Higher target → more variety.

- `MirostatTau = 3.0` — tight. Output is focused, often close to greedy.
- `MirostatTau = 5.0` (default) — balanced. Good match for general chat.
- `MirostatTau = 7.0+` — looser. Output has more variety, higher perplexity.

`MirostatTau` only matters when [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) is enabled (`1` or `2`). Otherwise the field is ignored.

## When to change it

| Scenario | Value |
|---|---|
| Default | `5.0` |
| Precise / code / factual output | `3.0` – `4.0` |
| Creative writing | `6.0` – `8.0` |

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.Mirostat = 2;
preset.SamplerParameters.MirostatTau = 4.0f;  // tighter than default
preset.SamplerParameters.MirostatEta = 0.1f;

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — must be `1` or `2` for `MirostatTau` to take effect.
- [`MirostatEta`](/net/developer-reference/parameters/sampler/mirostat-eta/) — how fast Mirostat adapts toward `Tau`.

## What's next

- [Mirostat](/net/developer-reference/parameters/sampler/mirostat/) — main mode selector.
- [MirostatEta](/net/developer-reference/parameters/sampler/mirostat-eta/) — learning rate.
- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs.

---
weight: 240
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/mirostat-eta/
feedback: LLMNET
version: 26.5.0
title: MirostatEta
description: Mirostat learning rate in Aspose.LLM for .NET — how fast the adaptive sampler adjusts toward the target entropy.
keywords:
- MirostatEta
- Mirostat
- learning rate
- sampler
---

`MirostatEta` is the learning rate of Mirostat's adaptive loop. It controls how quickly Mirostat reacts to divergence between observed entropy and the target entropy [`MirostatTau`](/net/developer-reference/parameters/sampler/mirostat-tau/).

## Quick reference

| | |
|---|---|
| **Type** | `float` |
| **Default** | `0.1` |
| **Range** | `> 0`; typical `0.05` – `0.3` |
| **Category** | Adaptive sampler |
| **Field on** | `SamplerParameters.MirostatEta` |

## What it does

After each token, Mirostat computes the difference between the observed entropy and `MirostatTau`. It scales that difference by `MirostatEta` and adjusts its internal threshold accordingly.

- `MirostatEta = 0.05` — slow adaptation. Smoother behavior; takes longer to settle.
- `MirostatEta = 0.1` (default) — balanced.
- `MirostatEta = 0.3` — fast adaptation. Tighter tracking of `Tau` but noisier.

`MirostatEta` only matters when [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) is enabled.

## When to change it

| Scenario | Value |
|---|---|
| Default | `0.1` |
| Smoother, slower adaptation | `0.05` |
| Fast tracking, aggressive correction | `0.2` – `0.3` |

Most users leave `MirostatEta = 0.1`. Adjust only when Mirostat's entropy wanders too far from `Tau` or oscillates too much.

## Example

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.Mirostat = 2;
preset.SamplerParameters.MirostatTau = 5.0f;
preset.SamplerParameters.MirostatEta = 0.05f;  // slow, smooth adaptation

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — must be `1` or `2`.
- [`MirostatTau`](/net/developer-reference/parameters/sampler/mirostat-tau/) — the target `MirostatEta` adapts toward.

## What's next

- [Mirostat](/net/developer-reference/parameters/sampler/mirostat/) — mode selector.
- [MirostatTau](/net/developer-reference/parameters/sampler/mirostat-tau/) — entropy target.
- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs.

---
weight: 250
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/logit-bias/
feedback: LLMNET
version: 26.5.0
title: LogitBias
description: Per-token logit adjustment in Aspose.LLM for .NET — add or subtract a value from any token's logit to favor or ban it.
keywords:
- LogitBias
- sampler
- token bias
- ban token
- force token
---

`LogitBias` lets you add a per-token offset to the model's logits before sampling. Positive values favor a token; large negative values effectively ban it.

## Quick reference

| | |
|---|---|
| **Type** | `Dictionary<int, float>` (token ID → bias) |
| **Default** | Empty dictionary (no biases) |
| **Range** | Typical `-100.0` to `+10.0` |
| **Category** | Fine-grained control |
| **Field on** | `SamplerParameters.LogitBias` |

## What it does

Before any filter runs, for each token ID present in `LogitBias`, the engine adds the associated float to that token's logit.

- `LogitBias[1234] = +5.0` — the token with ID 1234 gets a +5 boost to its logit. Strongly favors it.
- `LogitBias[5678] = -100.0` — the token at ID 5678 gets a massive negative bias. Effectively banned.
- `LogitBias[42] = -2.0` — token 42 is discouraged but still available.

Bias is applied once per step regardless of token occurrence history. It does not interact with the penalty window.

## When to change it

| Scenario | Example |
|---|---|
| Ban profanity / slurs token IDs | Set to `-100.0` |
| Force specific token (end-of-message, for instance) | Set to `+5.0` |
| Soft discouragement without ban | Set to `-2.0` |

Obtaining token IDs requires the model's tokenizer — not exposed in this parameter bag in the current SDK version. If you need logit-bias control, collect token IDs by tokenizing your target strings externally (for example, with the llama.cpp `tokenize` tool against the same model).

## Example

```csharp
var preset = new Qwen25Preset();

// Soft discourage token ID 1234; strong discourage 5678.
preset.SamplerParameters.LogitBias[1234] = -2.0f;
preset.SamplerParameters.LogitBias[5678] = -100.0f;

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`Temperature`](/net/developer-reference/parameters/sampler/temperature/), other filters — run after `LogitBias` is applied.
- [`Mirostat`](/net/developer-reference/parameters/sampler/mirostat/) — `LogitBias` still applies; Mirostat adapts around the biased distribution.

## What's next

- [Sampler parameters hub](/net/developer-reference/parameters/sampler/) — all sampler knobs.
- [EnableInfill](/net/developer-reference/parameters/sampler/enable-infill/) — specialized sampling mode.

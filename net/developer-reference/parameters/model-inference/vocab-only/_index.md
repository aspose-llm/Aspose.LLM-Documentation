---
weight: 60
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/vocab-only/
feedback: LLMNET
version: 26.5.0
title: VocabOnly
description: Load only the vocabulary of the model in Aspose.LLM for .NET — no weights; used for tokenizer-only scenarios.
keywords:
- VocabOnly
- vocabulary
- tokenizer
- model loading
---

`VocabOnly` loads just the model's vocabulary and tokenizer without loading the weights. The resulting model cannot generate output — it is a tokenizer-only configuration.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (false — load full model) |
| **Category** | Model loading |
| **Field on** | `ModelInferenceParameters.VocabOnly` |

## What it does

- `null` or `false` — load the full model (vocabulary + weights). Required for inference.
- `true` — load only vocabulary data. The model is loaded with no weights; chat methods are not meaningful.

Use `VocabOnly = true` only for tokenizer-level operations — for example, probing token IDs to populate [`LogitBias`](/net/developer-reference/parameters/sampler/logit-bias/) without paying the cost of loading weights.

## When to change it

| Scenario | Value |
|---|---|
| Normal chat inference | `null` or `false` |
| Tokenizer-only probing | `true` |

Rare in practice. Most applications load the full model.

## Example

```csharp
var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.VocabOnly = true;
// Tokenizer-only mode. Chat methods will not function; use only for
// token-ID discovery.
```

## Interactions

- Other `ModelInferenceParameters` fields (GpuLayers, TensorSplit, etc.) are largely irrelevant in `VocabOnly` mode.

## What's next

- [LogitBias](/net/developer-reference/parameters/sampler/logit-bias/) — use case for token-ID probing.
- [Model inference hub](/net/developer-reference/parameters/model-inference/) — all inference knobs.

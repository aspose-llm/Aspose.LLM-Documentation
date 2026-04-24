---
weight: 180
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/pooling-type/
feedback: LLMNET
version: 26.5.0
title: PoolingType
description: Embedding pooling strategy in Aspose.LLM for .NET — how per-token embeddings are reduced to a single vector.
keywords:
- PoolingType
- embeddings
- pooling
- mean pooling
- CLS
---

`PoolingType` selects the strategy the engine uses to reduce per-token embeddings to a single vector for the full input. Relevant only when [`Embeddings`](/net/developer-reference/parameters/context/embeddings/) is `true`.

## Quick reference

| | |
|---|---|
| **Type** | `PoolingType?` enum |
| **Default** | `null` (use model default) |
| **Values** | `Unspecified`, `None`, `Mean`, `Cls`, `Last`, `Rank` |
| **Category** | Embeddings |
| **Field on** | `ContextParameters.PoolingType` |

## What it does

| Value | Behavior |
|---|---|
| `Unspecified` (`-1`) | Use model default. |
| `None` (`0`) | Return per-token embeddings without reduction. |
| `Mean` (`1`) | Average all token embeddings. Good default for sentence-level semantic similarity. |
| `Cls` (`2`) | Use the first (CLS) token's embedding. Common for BERT-family. |
| `Last` (`3`) | Use the last token's embedding. Common for causal-LM embeddings. |
| `Rank` (`4`) | Rank-based pooling (experimental). |

Pick the pooling strategy the model was trained with. Mismatched pooling produces embeddings of degraded quality.

## When to change it

| Scenario | Value |
|---|---|
| Default chat — not used | `null` |
| Causal-LM embeddings | `Last` |
| BERT-style embedder | `Cls` |
| Sentence-transformer-style | `Mean` |

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ContextParameters.Embeddings = true;
preset.ContextParameters.PoolingType = PoolingType.Mean;
preset.ContextParameters.AttentionType = AttentionType.NonCausal;

using var api = AsposeLLMApi.Create(preset);
// Embedding-only configuration.
```

## Interactions

- [`Embeddings`](/net/developer-reference/parameters/context/embeddings/) — must be `true` for `PoolingType` to take effect.
- [`AttentionType`](/net/developer-reference/parameters/context/attention-type/) — usually `NonCausal` with embedding-specific pooling.

## What's next

- [Embeddings](/net/developer-reference/parameters/context/embeddings/) — flag that enables this pipeline.
- [AttentionType](/net/developer-reference/parameters/context/attention-type/) — companion choice.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

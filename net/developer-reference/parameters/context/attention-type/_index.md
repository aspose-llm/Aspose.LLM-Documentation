---
weight: 150
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/attention-type/
feedback: LLMNET
version: 26.5.0
title: AttentionType
description: Causal vs non-causal attention in Aspose.LLM for .NET — standard chat uses Causal; embedding workflows may use NonCausal.
keywords:
- AttentionType
- causal attention
- non-causal
- bidirectional
- embeddings
---

`AttentionType` selects between causal (autoregressive) and non-causal (bidirectional) attention. Standard chat models use causal attention; some embedding models use non-causal.

## Quick reference

| | |
|---|---|
| **Type** | `AttentionType?` enum |
| **Default** | `null` (use model default) |
| **Values** | `Unspecified`, `Causal`, `NonCausal` |
| **Category** | Attention |
| **Field on** | `ContextParameters.AttentionType` |

## What it does

| Value | Behavior |
|---|---|
| `Unspecified` (`-1`) | Use the model's metadata-declared type. |
| `Causal` (`0`) | Each token attends only to earlier tokens. Standard for chat / autoregressive generation. |
| `NonCausal` (`1`) | Each token attends to all tokens. Used for some embedding models and masked-language workflows. |

All built-in chat presets use `Causal` implicitly (via model metadata). Change to `NonCausal` only for embedding extraction with a model trained for bidirectional attention.

## When to change it

| Scenario | Value |
|---|---|
| Default — chat / text generation | `Unspecified` (model wins) |
| Bidirectional embedding extraction | `NonCausal` |

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ContextParameters.AttentionType = AttentionType.NonCausal;
preset.ContextParameters.Embeddings = true;
preset.ContextParameters.PoolingType = PoolingType.Mean;
// Embedding-only configuration. Chat generation is not meaningful here.
```

## Interactions

- [`Embeddings`](/net/developer-reference/parameters/context/embeddings/) — embedding extraction usually pairs with `NonCausal`.
- [`PoolingType`](/net/developer-reference/parameters/context/pooling-type/) — how embeddings are pooled.

## What's next

- [Embeddings](/net/developer-reference/parameters/context/embeddings/) — extraction mode flag.
- [PoolingType](/net/developer-reference/parameters/context/pooling-type/) — embedding pooling.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

---
weight: 190
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/embeddings/
feedback: LLMNET
version: 26.5.0
title: Embeddings
description: Enable embedding extraction in Aspose.LLM for .NET — switches the model into embedding-output mode alongside or instead of logits.
keywords:
- Embeddings
- embedding extraction
- vector
- semantic similarity
---

`Embeddings` is a boolean flag. When `true`, the engine extracts embedding vectors alongside (or instead of) logits. Use it with a [`PoolingType`](/net/developer-reference/parameters/context/pooling-type/) that matches the model's training regime.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (disabled) |
| **Category** | Embeddings |
| **Field on** | `ContextParameters.Embeddings` |

## What it does

- `null` or `false` — standard generation mode. Only logits are produced; no embedding extraction.
- `true` — the engine configures the pipeline to output embeddings per input.

Embeddings are typically used for semantic search, clustering, classification, or as retrieval keys in RAG systems. The SDK's current chat API (`SendMessageAsync`) focuses on text generation; embedding workflows require reaching into the `Engine` and `ChatSession` APIs directly.

## When to change it

| Scenario | Value |
|---|---|
| Default chat | `null` |
| Extract embeddings | `true`, paired with `PoolingType` and often `AttentionType = NonCausal` |

A dedicated use case for embeddings is on the documentation roadmap but not covered in this version.

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ContextParameters.Embeddings = true;
preset.ContextParameters.PoolingType = PoolingType.Mean;
preset.ContextParameters.AttentionType = AttentionType.NonCausal;

using var api = AsposeLLMApi.Create(preset);
// Direct chat methods do not surface embeddings; use Engine/ChatSession internals.
```

## Interactions

- [`PoolingType`](/net/developer-reference/parameters/context/pooling-type/) — reducer for token-level embeddings.
- [`AttentionType`](/net/developer-reference/parameters/context/attention-type/) — usually `NonCausal` for embedding-only models.

## What's next

- [PoolingType](/net/developer-reference/parameters/context/pooling-type/) — pooling strategy.
- [AttentionType](/net/developer-reference/parameters/context/attention-type/) — attention direction.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

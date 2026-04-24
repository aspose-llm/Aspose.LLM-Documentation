---
weight: 250
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/kv-unified/
feedback: LLMNET
version: 26.5.0
title: KvUnified
description: Use a unified KV buffer across sequences in Aspose.LLM for .NET — llama.cpp internal flag; leave at default unless instructed otherwise.
keywords:
- KvUnified
- KV cache
- unified buffer
- llama.cpp
---

`KvUnified` is an internal llama.cpp flag controlling whether the engine uses a single unified buffer for the KV cache across input sequences during attention. Leave at default unless specifically instructed by SDK guidance.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (use native default) |
| **Category** | KV cache (internal) |
| **Field on** | `ContextParameters.KvUnified` |

## What it does

The unified buffer layout can optimize some multi-sequence scenarios by colocating K and V for all sequences in one memory block. Whether this helps or hurts depends on backend and workload.

- `null` — native default. Usually correct.
- `true` — force unified buffer.
- `false` — force separate buffers.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Specific backend-tuning guidance from SDK docs | As instructed |

Most workloads never touch this.

## Example

```csharp
var preset = new Qwen25Preset();
// preset.ContextParameters.KvUnified = null; // default
```

## Interactions

- [`NSeqMax`](/net/developer-reference/parameters/context/n-seq-max/) — multi-sequence scenarios may interact with unified-buffer layout.

## What's next

- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

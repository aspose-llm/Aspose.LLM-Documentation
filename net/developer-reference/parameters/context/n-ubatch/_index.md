---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/n-ubatch/
feedback: LLMNET
version: 26.5.0
title: NUbatch
description: Physical maximum batch size in Aspose.LLM for .NET — the largest chunk actually processed per kernel call, typically ≤ NBatch.
keywords:
- NUbatch
- physical batch
- micro-batch
- kernel batch
---

`NUbatch` is the **physical** maximum batch size — the largest chunk actually processed in a single kernel call. Normally set equal to or smaller than [`NBatch`](/net/developer-reference/parameters/context/n-batch/).

## Quick reference

| | |
|---|---|
| **Type** | `uint?` |
| **Default** | `null` (native default, typically equal to `NBatch`) |
| **Range** | `≤ NBatch` |
| **Category** | Context size and batching |
| **Field on** | `ContextParameters.NUbatch` |

## What it does

`NBatch` defines the logical batch — the largest number of tokens submitted at once. `NUbatch` defines the largest chunk the engine processes in a single kernel invocation. When `NUbatch < NBatch`, the engine splits one logical batch into multiple kernel calls.

- `NUbatch = NBatch` (simplest case) — one logical batch = one kernel call.
- `NUbatch < NBatch` — one logical batch dispatched as several smaller kernel invocations.

The split matters mainly in specific multi-sequence scenarios where sequential processing of sub-batches is required. For single-sequence chat, `NUbatch = NBatch` is typical.

## When to change it

| Scenario | Value |
|---|---|
| Default — match `NBatch` | same as `NBatch` |
| Advanced multi-sequence workflows | Smaller than `NBatch` |

Most deployments set `NUbatch = NBatch` and never touch this field.

## Example

```csharp
var preset = new Qwen25Preset();
preset.ContextParameters.NBatch = 4096;
preset.ContextParameters.NUbatch = 4096;  // match the logical batch

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`NBatch`](/net/developer-reference/parameters/context/n-batch/) — upper bound; `NUbatch ≤ NBatch`.
- [`NSeqMax`](/net/developer-reference/parameters/context/n-seq-max/) — parallel sequence cap, related in multi-sequence scenarios.

## What's next

- [NBatch](/net/developer-reference/parameters/context/n-batch/) — logical batch cap.
- [NSeqMax](/net/developer-reference/parameters/context/n-seq-max/) — parallel sequences.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

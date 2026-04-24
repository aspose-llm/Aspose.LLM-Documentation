---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/n-seq-max/
feedback: LLMNET
version: 26.5.0
title: NSeqMax
description: Maximum parallel sequences in Aspose.LLM for .NET — matters for recurrent models; leave at default for standard transformers.
keywords:
- NSeqMax
- parallel sequences
- recurrent
- RNN
- Mamba
---

`NSeqMax` is the maximum number of distinct sequences the engine handles in parallel, each with its own state. It matters for recurrent or state-tracking models. Standard transformer chat presets do not require tuning it.

## Quick reference

| | |
|---|---|
| **Type** | `uint?` |
| **Default** | `null` (native default, typically 1 for transformer models) |
| **Range** | `1` and above; power-of-two values recommended for advanced scenarios |
| **Category** | Context size and batching |
| **Field on** | `ContextParameters.NSeqMax` |

## What it does

For recurrent or state-space models (Mamba, RWKV, hybrid architectures), each independent sequence carries its own recurrent state. `NSeqMax` caps how many such states the engine maintains simultaneously.

- `NSeqMax = 1` (default for standard transformers) — no parallel state tracking needed.
- `NSeqMax = 4+` — enables parallel recurrent-model sequences.

Transformer models (Qwen, Llama, Gemma, Phi, etc.) do not maintain per-sequence hidden state in this sense. `NSeqMax = 1` is correct for them.

## When to change it

| Scenario | Value |
|---|---|
| Standard transformer chat presets | Leave `null` or `1` |
| Recurrent / state-space model | Set to the number of parallel sequences you serve |

If you are not building against a recurrent-model-specific preset, leave `NSeqMax` at the default.

## Example

```csharp
// Standard transformer use case — no change needed.
var preset = new Qwen25Preset();
// preset.ContextParameters.NSeqMax = null; // (default)

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`NBatch`](/net/developer-reference/parameters/context/n-batch/), [`NUbatch`](/net/developer-reference/parameters/context/n-ubatch/) — batch sizes interact with sequence count in multi-sequence scenarios.

## What's next

- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.
- [NBatch](/net/developer-reference/parameters/context/n-batch/) — batch size for prompts.

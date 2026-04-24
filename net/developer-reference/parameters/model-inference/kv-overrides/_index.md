---
weight: 100
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/kv-overrides/
feedback: LLMNET
version: 26.5.0
title: KvOverrides
description: Runtime overrides for GGUF metadata keys in Aspose.LLM for .NET — patch context length, RoPE settings, architecture flags without modifying the file.
keywords:
- KvOverrides
- GGUF metadata
- override
- ModelKeyValueOverride
---

`KvOverrides` lets you patch specific keys in the GGUF metadata at load time. Each override targets one metadata key and provides a typed replacement value. Use to fix missing or incorrect metadata on a custom GGUF without rebuilding the file.

## Quick reference

| | |
|---|---|
| **Type** | `ModelKeyValueOverride[]?` |
| **Default** | `null` (no overrides) |
| **Category** | Model metadata |
| **Field on** | `ModelInferenceParameters.KvOverrides` |

## What it does

The engine reads model configuration from GGUF metadata at load. `KvOverrides` intercepts specific keys and substitutes your values. Common targets: context length, RoPE frequency base, RoPE scaling type.

Each override has:

| Field | Type |
|---|---|
| `Key` | `string` — metadata key (e.g., `llama.context_length`) |
| `Type` | `ModelKvOverrideType` — `Int`, `Float`, `Bool`, `String` |
| `IntValue`, `FloatValue`, `BoolValue`, `StringValue` | typed value slots |

Only the slot matching `Type` is read.

## When to change it

| Scenario | Value |
|---|---|
| Default — trust GGUF metadata | `null` |
| GGUF missing expected metadata | Single override for each missing key |
| Force a specific YaRN/RoPE recipe | Overrides for `llama.rope.*` keys |
| Diagnostic — test different metadata | Temporary overrides |

Wrong overrides silently break the model. Only patch metadata you have a clear reason to change.

## Example

```csharp
using Aspose.LLM.Abstractions.Parameters;

var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.KvOverrides = new[]
{
    new ModelKeyValueOverride
    {
        Key = "llama.rope.scaling.type",
        Type = ModelKvOverrideType.String,
        StringValue = "yarn",
    },
    new ModelKeyValueOverride
    {
        Key = "llama.context_length",
        Type = ModelKvOverrideType.Int,
        IntValue = 131072,
    },
};

using var api = AsposeLLMApi.Create(preset);
```

## Common override keys

| Key | Type | Notes |
|---|---|---|
| `llama.context_length` | `Int` | Declared training context length |
| `llama.embedding_length` | `Int` | Hidden size |
| `llama.rope.freq_base` | `Float` | RoPE theta |
| `llama.rope.scaling.type` | `String` | `"none"`, `"linear"`, `"yarn"`, `"longrope"` |
| `llama.rope.scaling.factor` | `Float` | Scaling multiplier |
| `general.architecture` | `String` | Model family name |

Exact key names vary by architecture. Inspect the model's metadata with a tool like `gguf-dump` from llama.cpp before overriding.

## Interactions

- [`ContextParameters.RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) — overriding `llama.rope.scaling.type` via `KvOverrides` has similar effect.
- [`ContextParameters.ContextSize`](/net/developer-reference/parameters/context/context-size/) — at load time, `KvOverrides` of `llama.context_length` defines what the runtime treats as the trained window.

## What's next

- [RopeScalingType](/net/developer-reference/parameters/context/rope-scaling-type/) — alternative way to control scaling.
- [Long context tuning](/net/use-cases/long-context-tuning/) — when `KvOverrides` helps.
- [Bring your own GGUF](/net/use-cases/bring-your-own-gguf/) — custom-model workflows.

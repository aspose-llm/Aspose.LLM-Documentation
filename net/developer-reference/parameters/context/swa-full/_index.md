---
weight: 240
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/swa-full/
feedback: LLMNET
version: 26.5.0
title: SwaFull
description: Use full-size SWA cache in Aspose.LLM for .NET — relevant for sliding-window attention models; stores uncompressed window instead of compressed.
keywords:
- SwaFull
- sliding window attention
- SWA
- KV cache
---

`SwaFull` controls whether the engine stores the full, uncompressed SWA (sliding-window attention) cache for models that use sliding-window attention. Only relevant for models with SWA layers.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (use native default) |
| **Category** | KV cache (SWA-specific) |
| **Field on** | `ContextParameters.SwaFull` |

## What it does

Sliding-window attention (used by some Mistral, Gemma, and other architectures) attends only to a bounded recent window. The engine can store this window either:

- **Compressed** (`SwaFull = false` or `null`) — smaller memory footprint, typical default.
- **Full** (`SwaFull = true`) — uncompressed, larger memory footprint, may be faster in specific workloads.

For models without SWA, this field has no effect.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Benchmarking SWA performance | `true` to test uncompressed path |
| Memory constrained on SWA model | `null` or `false` |

Few models currently on the built-in preset list use SWA extensively. If you are unsure, leave `null`.

## Example

```csharp
var preset = new Qwen25Preset();  // not SWA — SwaFull has no effect
preset.ContextParameters.SwaFull = null;  // default
```

## Interactions

- Only relevant for SWA-architected models.
- [`TypeK`](/net/developer-reference/parameters/context/type-k/), [`TypeV`](/net/developer-reference/parameters/context/type-v/) — the dtype applies regardless.

## What's next

- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.
- [Supported presets](/net/product-overview/supported-presets/) — check which presets use SWA.

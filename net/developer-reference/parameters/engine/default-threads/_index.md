---
weight: 40
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/engine/default-threads/
feedback: LLMNET
version: 26.5.0
title: DefaultThreads
description: Default thread count for Aspose.LLM for .NET operations when per-bag thread fields are unset — defaults to ProcessorCount minus 1.
keywords:
- DefaultThreads
- CPU threads
- parallelism
- default
---

`DefaultThreads` is the thread count the engine uses when the more specific [`ContextParameters.NThreads`](/net/developer-reference/parameters/context/n-threads/) and [`NThreadsBatch`](/net/developer-reference/parameters/context/n-threads-batch/) are not set.

## Quick reference

| | |
|---|---|
| **Type** | `int` |
| **Default** | `Environment.ProcessorCount - 1` |
| **Category** | Engine |
| **Field on** | `EngineParameters.DefaultThreads` |

## What it does

When [`ContextParameters.NThreads`](/net/developer-reference/parameters/context/n-threads/) is `null`, the engine falls back to `DefaultThreads`. Same for `NThreadsBatch`.

The default leaves one logical core free for the rest of your application. On an 8-core machine, the default is 7. On a 16-core machine, 15.

## When to change it

| Scenario | Value |
|---|---|
| Default | unset (uses `ProcessorCount - 1`) |
| Dedicated inference box | `Environment.ProcessorCount` (all cores) |
| CPU-quota-limited container | Fixed smaller number matching quota |
| Shared host with other heavy work | Half `ProcessorCount` |

For per-phase control (generation vs prompt processing), set [`NThreads`](/net/developer-reference/parameters/context/n-threads/) and [`NThreadsBatch`](/net/developer-reference/parameters/context/n-threads-batch/) directly — they override `DefaultThreads`.

## Example

```csharp
var preset = new Qwen25Preset();
preset.EngineParameters.DefaultThreads = 4;
// Only 4 threads used when ContextParameters.NThreads is null.

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`ContextParameters.NThreads`](/net/developer-reference/parameters/context/n-threads/) — generation threads; overrides default when set.
- [`ContextParameters.NThreadsBatch`](/net/developer-reference/parameters/context/n-threads-batch/) — prompt-processing threads; overrides default when set.

## What's next

- [NThreads](/net/developer-reference/parameters/context/n-threads/) — per-phase override.
- [CPU acceleration](/net/developer-reference/acceleration/cpu/) — threading strategy.
- [Engine parameters hub](/net/developer-reference/parameters/engine/) — all engine knobs.

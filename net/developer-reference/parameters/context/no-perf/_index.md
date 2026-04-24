---
weight: 270
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/no-perf/
feedback: LLMNET
version: 26.5.0
title: NoPerf
description: Disable performance timing collection in Aspose.LLM for .NET — micro-optimization for high-throughput production; off by default.
keywords:
- NoPerf
- performance timing
- overhead
- production
---

`NoPerf` disables collection of performance timings inside the native layer. The savings are small but non-zero; useful in very high-throughput production loops.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (timings collected) |
| **Category** | Performance |
| **Field on** | `ContextParameters.NoPerf` |

## What it does

- `null` or `false` — timings collected. Minor overhead per call but you can inspect them via native logging.
- `true` — timings disabled. Slightly faster; nothing to inspect.

This is a micro-optimization. The savings are usually below measurement noise on a single request. On a high-throughput server processing many requests per second, the savings add up.

## When to change it

| Scenario | Value |
|---|---|
| Default (keeps timings for debugging) | `null` |
| High-throughput production squeezing every last cycle | `true` |
| Debugging performance | `null` or `false` |

## Example

```csharp
var preset = new Qwen25Preset();
preset.ContextParameters.NoPerf = true;           // production
preset.EngineParameters.EnableDebugLogging = false;
```

## Interactions

- [`EnableDebugLogging`](/net/developer-reference/parameters/engine/) — turning on debug logs defeats the purpose of `NoPerf = true`.

## What's next

- [Performance issues troubleshooting](/net/troubleshooting/performance-issues/) — throughput-focused tuning.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

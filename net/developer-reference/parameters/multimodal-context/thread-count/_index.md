---
weight: 30
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/multimodal-context/thread-count/
feedback: LLMNET
version: 26.5.0
title: ThreadCount
description: Thread count for mtmd (multimodal) processing in Aspose.LLM for .NET — null defers to native default; lower for CPU-constrained hosts.
keywords:
- ThreadCount
- mtmd threads
- CPU
- multimodal
---

`ThreadCount` sets the number of threads `mtmd` uses for CPU-side multimodal work (image preprocessing, CPU portions of the projector).

## Quick reference

| | |
|---|---|
| **Type** | `int?` |
| **Default** | `null` (native default — often half the logical cores) |
| **Category** | Multimodal context |
| **Field on** | `MultimodalContextParameters.ThreadCount` |

## What it does

- `null` — use mtmd's heuristic.
- Positive integer — use that many threads.

Multimodal preprocessing is short-lived per request. On a server handling many concurrent vision requests, cap `ThreadCount` to prevent per-request work from starving other requests.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Concurrent vision requests | `2` – `4` per request |
| Single-request, all cores available | `Environment.ProcessorCount / 2` |

## Example

```csharp
var preset = new Qwen3VL2BPreset();
preset.MtmdContextParameters.ThreadCount = 4;

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`ContextParameters.NThreads`](/net/developer-reference/parameters/context/n-threads/) — base-model generation threads; independent of mtmd's thread count.

## What's next

- [Multimodal context hub](/net/developer-reference/parameters/multimodal-context/) — all mtmd knobs.
- [Context parameters — NThreads](/net/developer-reference/parameters/context/n-threads/) — base-model counterpart.

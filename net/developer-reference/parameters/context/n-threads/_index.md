---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/n-threads/
feedback: LLMNET
version: 26.5.0
title: NThreads
description: CPU thread count for per-token generation in Aspose.LLM for .NET — affects throughput during sequential decoding.
keywords:
- NThreads
- CPU threads
- decoding
- generation
- throughput
---

`NThreads` is the number of CPU threads the engine uses during **generation** — when producing each output token sequentially. Generation is bandwidth-bound and often does not benefit from all available cores.

## Quick reference

| | |
|---|---|
| **Type** | `int?` |
| **Default** | `null` (falls back to `EngineParameters.DefaultThreads`) |
| **Range** | `1` and above |
| **Category** | Threading |
| **Field on** | `ContextParameters.NThreads` |

## What it does

During the generation phase (token-by-token decode), the engine distributes matrix multiplications across `NThreads` CPU threads. When `null`, it uses [`EngineParameters.DefaultThreads`](/net/developer-reference/parameters/engine/), which defaults to `ProcessorCount - 1`.

- `NThreads = 4` — decent for 4-core machines; use most cores.
- `NThreads = 8` — common sweet spot on mainstream desktop CPUs.
- `NThreads = 16+` — diminishing returns; sometimes slower due to cache contention and memory-bandwidth saturation.

Unlike prompt processing (which scales well with more threads), generation often peaks at 8-12 threads and degrades with more. Benchmark on your hardware.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` (use `DefaultThreads`) |
| Laptop / 4-8 core | `4` – `6` |
| Mainstream desktop | `8` – `10` |
| High-core server (but avoid over-allocation) | `10` – `16` |
| Competing with other CPU workloads | Cap explicitly to half `ProcessorCount` |

Set `NThreads` and [`NThreadsBatch`](/net/developer-reference/parameters/context/n-threads-batch/) separately — generation and prompt processing have different optima.

## Example

```csharp
var preset = new Qwen25Preset();
preset.ContextParameters.NThreads = 8;
preset.ContextParameters.NThreadsBatch = 16;  // more threads for prompt processing

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`EngineParameters.DefaultThreads`](/net/developer-reference/parameters/engine/) — fallback when `NThreads` is null.
- [`NThreadsBatch`](/net/developer-reference/parameters/context/n-threads-batch/) — prompt-processing threads.
- CPU acceleration — `NThreads` has no effect when GPU offload is active for every layer.

## What's next

- [NThreadsBatch](/net/developer-reference/parameters/context/n-threads-batch/) — prompt-processing variant.
- [CPU acceleration](/net/developer-reference/acceleration/cpu/) — how threading interacts with AVX variants.
- [Performance issues](/net/troubleshooting/performance-issues/) — thread-related throughput issues.

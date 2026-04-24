---
weight: 60
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/n-threads-batch/
feedback: LLMNET
version: 26.5.0
title: NThreadsBatch
description: CPU thread count for prompt processing in Aspose.LLM for .NET — typically higher than NThreads because batch workloads parallelize better.
keywords:
- NThreadsBatch
- CPU threads
- prompt processing
- prefill
- parallelism
---

`NThreadsBatch` is the number of CPU threads the engine uses during **prompt processing** (the initial prefill phase). Prompt processing is embarrassingly parallel and benefits from using most or all available cores.

## Quick reference

| | |
|---|---|
| **Type** | `int?` |
| **Default** | `null` (falls back to `EngineParameters.DefaultThreads`) |
| **Range** | `1` and above |
| **Category** | Threading |
| **Field on** | `ContextParameters.NThreadsBatch` |

## What it does

During prompt processing, the engine runs matrix multiplications over many tokens at once. This workload parallelizes well: more threads directly translate to higher throughput, up to memory-bandwidth limits.

- `NThreadsBatch = ProcessorCount` — typical. Use all cores for fast prompt ingestion.
- `NThreadsBatch = half ProcessorCount` — leave room for other CPU workloads.
- `NThreadsBatch < NThreads` — unusual, almost always wrong for modern CPUs.

Prompt processing happens once per incoming message (on user input). Generation ([`NThreads`](/net/developer-reference/parameters/context/n-threads/)) happens per output token. For chat, prompt-processing time dominates when the prompt is long, generation dominates when the output is long.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` (use `DefaultThreads`) |
| Dedicated inference machine | `ProcessorCount` (all cores) |
| Shared machine | `ProcessorCount / 2` |
| Very long prompts, memory-bound hardware | Benchmark — adding threads may not help past 16 |

## Example

```csharp
var preset = new Qwen25Preset();
preset.ContextParameters.NThreads = 8;                             // generation
preset.ContextParameters.NThreadsBatch = Environment.ProcessorCount; // prompt prefill
```

## Interactions

- [`NThreads`](/net/developer-reference/parameters/context/n-threads/) — generation threads; typically different from `NThreadsBatch`.
- [`NBatch`](/net/developer-reference/parameters/context/n-batch/) — larger batch sizes better utilize high `NThreadsBatch`.
- [`EngineParameters.DefaultThreads`](/net/developer-reference/parameters/engine/) — fallback when null.

## What's next

- [NThreads](/net/developer-reference/parameters/context/n-threads/) — generation-phase threads.
- [NBatch](/net/developer-reference/parameters/context/n-batch/) — batch size.
- [Reduce first-token latency](/net/how-to/reduce-first-token-latency/) — prompt-processing throughput's role in TTFT.

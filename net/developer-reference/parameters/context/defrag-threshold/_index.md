---
weight: 230
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/defrag-threshold/
feedback: LLMNET
version: 26.5.0
title: DefragThreshold
description: KV cache defragmentation trigger in Aspose.LLM for .NET — fraction of holes above which the engine compacts the cache. Negative disables.
keywords:
- DefragThreshold
- defragmentation
- KV cache
- compaction
---

`DefragThreshold` is the fraction of KV cache holes above which the engine triggers defragmentation. Useful for long-running services where repeated cleanup creates fragmentation.

## Quick reference

| | |
|---|---|
| **Type** | `float?` |
| **Default** | `null` (disabled — same as negative value) |
| **Range** | `< 0` = disabled; `0.0` – `1.0` enables |
| **Category** | KV cache maintenance |
| **Field on** | `ContextParameters.DefragThreshold` |

## What it does

When messages are evicted from the KV cache (by [`CacheCleanupStrategy`](/net/developer-reference/parameters/chat/)), their slots become holes. Over many cycles, the cache may hold scattered used slots interspersed with holes, wasting capacity.

If `DefragThreshold` is set, the engine monitors the hole fraction. When it crosses the threshold, the engine compacts the cache — moves live tokens together and frees the tail.

- `null` or negative — disabled. Cache is never compacted.
- `0.1` – `0.5` — typical active values. Compact when 10-50 % of the cache is holes.

## When to change it

| Scenario | Value |
|---|---|
| Default (short-lived or bounded sessions) | `null` |
| Long-running service with many evictions | `0.3` |
| Aggressive compaction | `0.1` |

Compaction has a one-time cost when triggered. For bursty workloads where cache usage oscillates, defrag helps sustained throughput.

## Example

```csharp
var preset = new Qwen25Preset();
preset.ContextParameters.DefragThreshold = 0.3f;
// Compact when >30 % of KV slots are holes.
```

## Interactions

- [`CacheCleanupStrategy`](/net/developer-reference/parameters/chat/) — the policy that creates the holes defrag compacts.
- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — larger caches benefit more from defrag.

## What's next

- [Cache management](/net/developer-reference/cache-management/) — cleanup strategies and compaction together.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/use-memory-mapping/
feedback: LLMNET
version: 26.5.0
title: UseMemoryMapping
description: Toggle mmap-based model loading in Aspose.LLM for .NET — default true; disable only for network filesystems or specific memory layouts.
keywords:
- UseMemoryMapping
- mmap
- model loading
- memory
- file system
---

`UseMemoryMapping` controls whether the engine memory-maps the GGUF file instead of reading it into RAM. Memory mapping lets the OS stream the model on demand, which cuts startup time and peak memory.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (native default — usually `true`) |
| **Category** | Model loading |
| **Field on** | `ModelInferenceParameters.UseMemoryMapping` |

## What it does

- `true` (default) — the OS maps the GGUF file into address space. Pages are brought into memory on first access. Startup time is fast; peak memory is bounded by the working set.
- `false` — the engine reads the full file into RAM before model init. Startup is slower; peak memory doubles during load (read buffer + allocation).
- `null` — native default; behaves as `true` on most platforms.

Memory mapping is preferred unless your filesystem does not support `mmap` (some network filesystems, container volume drivers).

## When to change it

| Scenario | Value |
|---|---|
| Default (recommended) | `null` |
| Network filesystem without mmap support | `false` |
| Need full model loaded into RAM upfront | `false` |
| Debugging mmap-specific issues | `false` |

## Example

```csharp
var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.UseMemoryMapping = true;  // default, shown explicitly
```

For an NFS-mounted model:

```csharp
preset.BaseModelInferenceParameters.UseMemoryMapping = false;
// Loading takes longer, but works on filesystems where mmap fails.
```

## Interactions

- [`UseMemoryLocking`](/net/developer-reference/parameters/model-inference/use-memory-locking/) — lock working set to prevent paging.
- [`GpuLayers`](/net/developer-reference/parameters/model-inference/gpu-layers/) — offloaded layers are copied from the mapped file to GPU memory.

## What's next

- [UseMemoryLocking](/net/developer-reference/parameters/model-inference/use-memory-locking/) — prevent paging.
- [GpuLayers](/net/developer-reference/parameters/model-inference/gpu-layers/) — GPU offload.
- [Model inference hub](/net/developer-reference/parameters/model-inference/) — all inference knobs.

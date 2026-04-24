---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/use-memory-locking/
feedback: LLMNET
version: 26.5.0
title: UseMemoryLocking
description: Lock model memory to prevent OS paging in Aspose.LLM for .NET — requires mlock/VirtualLock privileges; off by default.
keywords:
- UseMemoryLocking
- mlock
- VirtualLock
- paging
- swap
---

`UseMemoryLocking` requests the OS to lock model memory pages, preventing them from being swapped out. Requires elevated privileges or raised ulimits.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (native default — usually `false`) |
| **Category** | Model loading |
| **Field on** | `ModelInferenceParameters.UseMemoryLocking` |

## What it does

- `true` — the engine calls `mlock` (Linux/macOS) or `VirtualLock` (Windows) on the model memory. The OS will not page it out.
- `false` or `null` — no locking. OS may page model memory under pressure.

Paging inference model memory is catastrophic for performance — suddenly generation stalls for seconds while the kernel pages weights back from disk. `UseMemoryLocking = true` prevents that.

**Cost**: requires appropriate privileges. On Linux, the user must have sufficient `RLIMIT_MEMLOCK` (raise via `ulimit -l` or `/etc/security/limits.conf`). On Windows, the process needs "Lock Pages in Memory" permission.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` (disabled) |
| Shared host under memory pressure | `true` (requires privilege) |
| Container without memlock capability | `null` (do not attempt) |
| Dedicated inference machine with ample RAM | `null` (unnecessary) |

## Example

```csharp
var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.UseMemoryLocking = true;
// Requires the process to have the required OS-level privilege.
```

Linux ulimit bump (at the shell, before running):

```bash
ulimit -l unlimited
dotnet run
```

## Interactions

- [`UseMemoryMapping`](/net/developer-reference/parameters/model-inference/use-memory-mapping/) — with mmap on, `mlock` locks the mapped pages as they fault in.
- System-level configuration — `mlock` availability depends on OS limits.

## What's next

- [UseMemoryMapping](/net/developer-reference/parameters/model-inference/use-memory-mapping/) — companion load-time knob.
- [Model inference hub](/net/developer-reference/parameters/model-inference/) — all inference knobs.

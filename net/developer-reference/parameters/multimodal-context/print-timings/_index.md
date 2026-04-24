---
weight: 20
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/multimodal-context/print-timings/
feedback: LLMNET
version: 26.5.0
title: PrintTimings
description: Enable mtmd per-step timing diagnostics in Aspose.LLM for .NET — useful for diagnosing slow vision first-token latency; off in production.
keywords:
- PrintTimings
- mtmd timings
- performance diagnostics
- vision
---

`PrintTimings` turns on `mtmd`'s built-in per-step timing output — time spent tokenizing, running the projector, and evaluating chunks. Useful when a vision query is slow and you need to localize the bottleneck.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (off) |
| **Category** | Multimodal context diagnostics |
| **Field on** | `MultimodalContextParameters.PrintTimings` |

## What it does

- `null` or `false` — no timing output.
- `true` — `mtmd` emits per-stage timings through the native logger. Requires [`EnableDebugLogging`](/net/developer-reference/parameters/engine/enable-debug-logging/) or equivalent output routing to see them.

Timings are noisy. Disable in production; keep on only while debugging.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Investigating vision first-token latency | `true` |
| Production | `null` / `false` |

## Example

```csharp
var preset = new Qwen3VL2BPreset();
preset.EngineParameters.EnableDebugLogging = true;
preset.MtmdContextParameters.PrintTimings = true;

using var api = AsposeLLMApi.Create(preset, logger);
// Per-stage mtmd timings now appear in the logs.
```

## Interactions

- [`EnableDebugLogging`](/net/developer-reference/parameters/engine/enable-debug-logging/) — required for timing lines to reach the logger.
- [`Verbosity`](/net/developer-reference/parameters/multimodal-context/verbosity/) — complementary verbosity knob.

## What's next

- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — full diagnostic playbook.
- [Verbosity](/net/developer-reference/parameters/multimodal-context/verbosity/) — mtmd log-level knob.
- [Multimodal context hub](/net/developer-reference/parameters/multimodal-context/) — all mtmd knobs.

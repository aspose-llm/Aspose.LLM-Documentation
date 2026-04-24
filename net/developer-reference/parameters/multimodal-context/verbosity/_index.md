---
weight: 40
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/multimodal-context/verbosity/
feedback: LLMNET
version: 26.5.0
title: Verbosity
description: mtmd log verbosity in Aspose.LLM for .NET — integer 0-3 (Error, Warn, Info, Debug); off by default in production.
keywords:
- Verbosity
- mtmd logs
- log level
- diagnostics
---

`Verbosity` controls how much the `mtmd` (multimodal) layer logs. Higher values produce more detail, useful for diagnosing vision issues but noisy in production.

## Quick reference

| | |
|---|---|
| **Type** | `int?` |
| **Default** | `null` (use native default) |
| **Range** | `0` = Error, `1` = Warn, `2` = Info, `3` = Debug |
| **Category** | Multimodal context diagnostics |
| **Field on** | `MultimodalContextParameters.Verbosity` |

## What it does

Maps to the native mtmd log-level setting. Higher values include all lower-level logs plus their own.

- `0` — errors only.
- `1` — warnings and errors.
- `2` — info, warnings, errors.
- `3` — debug and everything lower.

Logs are visible when either [`EnableDebugLogging`](/net/developer-reference/parameters/engine/enable-debug-logging/) is on, or the logger passed to `AsposeLLMApi.Create` is configured to accept them.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Production | `0` or `1` |
| Diagnosing vision template / alignment | `3` |

## Example

```csharp
var preset = new Qwen3VL2BPreset();
preset.EngineParameters.EnableDebugLogging = true;
preset.MtmdContextParameters.Verbosity = 3;
preset.MtmdContextParameters.PrintTimings = true;

using var api = AsposeLLMApi.Create(preset, logger);
```

## Interactions

- [`EnableDebugLogging`](/net/developer-reference/parameters/engine/enable-debug-logging/) — required for high-verbosity lines to reach the logger.
- [`PrintTimings`](/net/developer-reference/parameters/multimodal-context/print-timings/) — per-stage timings as a complementary diagnostic.

## What's next

- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — tag taxonomy and diagnostic flow.
- [Multimodal context hub](/net/developer-reference/parameters/multimodal-context/) — all mtmd knobs.

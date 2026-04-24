---
weight: 20
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/engine/enable-debug-logging/
feedback: LLMNET
version: 26.5.0
title: EnableDebugLogging
description: Turn on verbose native-level logs in Aspose.LLM for .NET — useful for diagnosis; off by default to keep production throughput high.
keywords:
- EnableDebugLogging
- debug logs
- ILogger
- diagnostics
- verbose
---

`EnableDebugLogging` toggles verbose diagnostic logs from the native `llama.cpp` layer. Off by default to minimize production overhead; flip on when investigating a specific problem.

## Quick reference

| | |
|---|---|
| **Type** | `bool` |
| **Default** | `false` |
| **Category** | Engine |
| **Field on** | `EngineParameters.EnableDebugLogging` |

## What it does

- `false` (default) — the native layer produces no diagnostic output. Inference runs at full throughput.
- `true` — the native layer emits tagged lines (`[MM]`, `[CTX]`, `[KV]`, etc.) via `NativeLoggerAdapter` into the `ILogger` you pass to `AsposeLLMApi.Create`. Lines are also written to the file at [`LogDirectoryPath`](/net/developer-reference/parameters/engine/log-directory-path/).

Debug logging adds measurable overhead — typically 5-15 % throughput loss. Not intended for production.

## When to change it

| Scenario | Value |
|---|---|
| Default production | `false` |
| Diagnosing template mismatch, KV eviction, vision alignment | `true` |
| Benchmarking a specific backend | `false` (avoid logging overhead) |
| CI / staging runs | Either — depends on debugging requirements |

## Example

```csharp
using Microsoft.Extensions.Logging;

using var loggerFactory = LoggerFactory.Create(b =>
    b.AddConsole().SetMinimumLevel(LogLevel.Debug));
var logger = loggerFactory.CreateLogger<Program>();

var preset = new Qwen25Preset();
preset.EngineParameters.EnableDebugLogging = true;
preset.EngineParameters.LogDirectoryPath = "logs/debug-run.log";

using var api = AsposeLLMApi.Create(preset, logger);
// Tagged [MM], [CTX], [KV] lines flow to both the logger and the file.
```

## Interactions

- [`LogDirectoryPath`](/net/developer-reference/parameters/engine/log-directory-path/) — file destination for debug output.
- [`MultimodalContextParameters.Verbosity`](/net/developer-reference/parameters/multimodal-context/verbosity/) — mtmd-layer verbosity, independent of this flag.
- `ILogger` passed to `AsposeLLMApi.Create` — debug lines route through it.

## What's next

- [Logging and diagnostics](/net/developer-reference/logging-and-diagnostics/) — tagged log taxonomy.
- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — multimodal-specific diagnosis.
- [Engine parameters hub](/net/developer-reference/parameters/engine/) — all engine knobs.

---
weight: 70
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/logging-and-diagnostics/
feedback: LLMNET
version: 26.5.0
title: Logging and diagnostics
description: Integrate ILogger with Aspose.LLM for .NET, enable native debug logs, and interpret tagged log output for multimodal, context, and KV cache diagnostics.
keywords:
- ILogger
- logging
- EnableDebugLogging
- NativeLoggerAdapter
- parse_mm_logs
- diagnostics
---

Aspose.LLM for .NET routes diagnostic output through standard `ILogger` abstractions and a native-log bridge. Enable debug logging when you need to diagnose inference issues, vision misalignments, or KV cache behavior.

## Connect an ILogger

Pass an `ILogger` to `AsposeLLMApi.Create`:

```csharp
using Microsoft.Extensions.Logging;

using var loggerFactory = LoggerFactory.Create(builder =>
    builder.AddConsole().SetMinimumLevel(LogLevel.Information));
var logger = loggerFactory.CreateLogger<Program>();

using var api = AsposeLLMApi.Create(new Qwen25Preset(), logger);
```

Logs from the engine, model manager, binary manager, and chat sessions flow into this logger. Without a logger, the engine produces no managed-side log output.

When you use the [DI path](/net/developer-reference/dependency-injection/) (`AddLlamaServices`), logging is configured automatically — a console provider plus a file provider pointed at `EngineParameters.LogDirectoryPath`.

## Enable native debug logs

Set `EngineParameters.EnableDebugLogging` before creating the API:

```csharp
var preset = new Qwen25Preset();
preset.EngineParameters.EnableDebugLogging = true;
preset.EngineParameters.LogDirectoryPath = "logs/debug.log";

using var api = AsposeLLMApi.Create(preset, logger);
```

When true:

- The native `llama.cpp` layer emits verbose diagnostic lines via `NativeLoggerAdapter`.
- The logger minimum level for the engine becomes `Debug`.
- Native logs are written to both the `ILogger` and the file at `LogDirectoryPath`.

Debug logging adds measurable overhead — disable in production.

## Log-line tags

Native logs use prefix tags so you can filter by concern. Common tags:

| Tag | Meaning |
|---|---|
| `[MM]` | Multimodal — projector load, template selection, chunk tokenization, alignment. |
| `[CTX]` | Context manager — batch dispatch, sequence IDs, context state. |
| `[KV]` | KV cache — reservations, evictions, cleanup strategy application. |
| `DBG tok:` | Tokenizer — raw tokens for prompts and generation steps. |
| `DBG mtmd-tokenize` | `mtmd_tokenize` details for image chunks. |

Example lines:

```
[MM] selected template: Qwen3VL
[MM] projector loaded: mmproj-Qwen3VL-2B-Instruct-Q8_0.gguf (524 MB)
[CTX] batch dispatched: tokens=1536, seq_id=0
[KV] cleanup: strategy=RemoveOldestMessages, freed=4096 tokens
```

Grep by tag to isolate a specific subsystem:

```bash
grep '^\[MM\]' run.log
grep '^\[KV\]' run.log
```

## `parse_mm_logs.zsh` helper

For multimodal debugging, the SDK repository ships `parse_mm_logs.zsh` — a zsh script that sections raw log output by concern (projector load, template choice, chunks, alignment, KV state, token stream, final answer). Useful when vision output is garbled and you need to localize the failure to a specific stage.

```bash
./parse_mm_logs.zsh < raw.log > sectioned.txt
```

See [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) for typical failure patterns and how the sectioned output helps.

## Log levels at a glance

| What you want to see | Set level | `EnableDebugLogging` |
|---|---|---|
| Errors only in production | `Warning` or `Error` | `false` |
| Default observability | `Information` | `false` |
| Diagnosing a specific issue | `Debug` | `true` |
| Full firehose during model porting / migration | `Debug` | `true` |

In the DI path, `EnableDebugLogging` sets the engine's effective level automatically. In the facade path, you control the level via the `ILogger` you pass in — `EnableDebugLogging` still controls the native-side verbosity independently.

## Sample: log to console and file

```csharp
using Microsoft.Extensions.Logging;

using var loggerFactory = LoggerFactory.Create(builder =>
{
    builder
        .AddConsole()
        .AddFile("logs/app-{Date}.log") // third-party file sink, e.g., Serilog.Extensions.Logging.File
        .SetMinimumLevel(LogLevel.Debug);
});
var logger = loggerFactory.CreateLogger<Program>();

var preset = new Qwen25Preset();
preset.EngineParameters.EnableDebugLogging = true;

using var api = AsposeLLMApi.Create(preset, logger);
```

The SDK itself does not bundle a file sink — add the provider of your choice (Serilog, NLog, built-in `AddFile` extensions) from your application.

## What's next

- [Engine parameters](/net/developer-reference/parameters/engine/) — `EnableDebugLogging` and `LogDirectoryPath`.
- [Dependency injection](/net/developer-reference/dependency-injection/) — automatic logging wiring in the DI path.
- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — multimodal-specific diagnostics.
- [Multimodal context parameters](/net/developer-reference/parameters/multimodal-context/) — `Verbosity` for `mtmd` layer logs.

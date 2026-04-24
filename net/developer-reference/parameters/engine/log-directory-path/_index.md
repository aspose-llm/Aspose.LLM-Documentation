---
weight: 30
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/engine/log-directory-path/
feedback: LLMNET
version: 26.5.0
title: LogDirectoryPath
description: File path for native log output in Aspose.LLM for .NET — defaults to logs/log.txt; the name says "directory" but the value is a full file path.
keywords:
- LogDirectoryPath
- log file
- native logging
- diagnostics
---

`LogDirectoryPath` is the file path where the native logger writes output. Despite the name, the value is a full file path — not a directory.

## Quick reference

| | |
|---|---|
| **Type** | `string` |
| **Default** | `"logs/log.txt"` |
| **Category** | Engine |
| **Field on** | `EngineParameters.LogDirectoryPath` |

## What it does

When the native logger writes (mostly when [`EnableDebugLogging`](/net/developer-reference/parameters/engine/enable-debug-logging/) is `true`), output goes to this file. If the file does not exist, it is created; if it does, output is appended.

- Default `logs/log.txt` — a file `log.txt` inside a `logs` folder, resolved relative to the current working directory.
- Absolute path — write to a specific location regardless of working directory.

## When to change it

| Scenario | Value |
|---|---|
| Default | `"logs/log.txt"` |
| Per-environment log file | `"logs/staging.log"`, `"logs/prod.log"` |
| Centralized logging location | `"/var/log/aspose-llm/app.log"` |
| Working directory is not writable | Absolute path to a writable folder |

## Example

```csharp
var preset = new Qwen25Preset();
preset.EngineParameters.EnableDebugLogging = true;
preset.EngineParameters.LogDirectoryPath = "/var/log/aspose-llm/app.log";

using var api = AsposeLLMApi.Create(preset, logger);
```

Ensure the parent directory exists and the process has write permission:

```csharp
var logFile = "/var/log/aspose-llm/app.log";
Directory.CreateDirectory(Path.GetDirectoryName(logFile)!);
preset.EngineParameters.LogDirectoryPath = logFile;
```

## Interactions

- [`EnableDebugLogging`](/net/developer-reference/parameters/engine/enable-debug-logging/) — the main switch for output that this file receives.
- `ILogger` passed to `AsposeLLMApi.Create` — complementary managed-side logging.

## What's next

- [Logging and diagnostics](/net/developer-reference/logging-and-diagnostics/) — full logging reference.
- [Engine parameters hub](/net/developer-reference/parameters/engine/) — all engine knobs.

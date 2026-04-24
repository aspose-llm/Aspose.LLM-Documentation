---
weight: 60
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/engine/
feedback: LLMNET
version: 26.5.0
title: Engine parameters
description: Configure engine-wide defaults in Aspose.LLM for .NET — model cache path, debug logging, log file location, and default thread count.
keywords:
- EngineParameters
- ModelCachePath
- EnableDebugLogging
- DefaultThreads
- LogDirectoryPath
- settings
---

`EngineParameters` holds engine-wide settings that apply across the entire `AsposeLLMApi` lifecycle. Unlike the per-session parameter bags, these values are read once during engine construction and not re-evaluated per request.

## Class reference

```csharp
namespace Aspose.LLM.Core.DependencyInjection;

public class EngineParameters
{
    public string? ModelCachePath { get; set; }    // default: <LocalAppData>/Aspose.LLM/models
    public bool EnableDebugLogging { get; set; }   // default: false
    public string LogDirectoryPath { get; set; }   // default: "logs/log.txt"
    public int DefaultThreads { get; set; }        // default: Environment.ProcessorCount - 1
}
```

All four properties have working defaults. Override only when the default does not match your deployment.

## Detailed field reference

Each field has a dedicated page with full defaults, scenario tables, code examples, and interactions.

- [ModelCachePath](/net/developer-reference/parameters/engine/model-cache-path/)
- [EnableDebugLogging](/net/developer-reference/parameters/engine/enable-debug-logging/)
- [LogDirectoryPath](/net/developer-reference/parameters/engine/log-directory-path/)
- [DefaultThreads](/net/developer-reference/parameters/engine/default-threads/)

## Fields

| Field | Type | Default | Purpose |
|---|---|---|---|
| `ModelCachePath` | `string?` | `<LocalAppData>/Aspose.LLM/models` | Folder where downloaded model files are stored. |
| `EnableDebugLogging` | `bool` | `false` | Enables native-level debug logs from `llama.cpp`. Verbose — use for diagnosis, not in production. |
| `LogDirectoryPath` | `string` | `"logs/log.txt"` | File path for native log output. Despite the name, this is a full file path, not a directory. |
| `DefaultThreads` | `int` | `ProcessorCount - 1` | Default thread count used when a parameter bag does not specify its own threading. |

### `ModelCachePath`

Points to the folder where downloaded GGUF files are cached. On first run, the engine:

1. Checks whether the model file already exists under this folder.
2. If not, downloads it from the source defined in [`ModelSourceParameters`](/net/developer-reference/parameters/model-source/).
3. Loads the cached file on subsequent runs.

Typical reasons to override:

- **Shared model cache** across multiple applications or users.
- **Faster disk** — point to an SSD when your `LocalApplicationData` is on an HDD.
- **Constrained disk layout** — put models on a data drive, code on the system drive.
- **Docker / container** scenarios — mount the cache as a volume so models survive restarts.

Example — use a shared cache under `D:\models`:

```csharp
var preset = new Qwen25Preset();
preset.EngineParameters.ModelCachePath = @"D:\models";

using var api = AsposeLLMApi.Create(preset);
```

### `EnableDebugLogging`

When `true`, the native `llama.cpp` layer emits verbose logs — useful when diagnosing inference errors, template mismatches, or KV cache issues. Combine with an `ILogger` passed to `AsposeLLMApi.Create(preset, logger)` to capture the output:

```csharp
using Microsoft.Extensions.Logging;

using var loggerFactory = LoggerFactory.Create(builder =>
    builder.AddConsole().SetMinimumLevel(LogLevel.Debug));
var logger = loggerFactory.CreateLogger<Program>();

var preset = new Qwen25Preset();
preset.EngineParameters.EnableDebugLogging = true;

using var api = AsposeLLMApi.Create(preset, logger);
```

Leave this `false` in production; debug logs materially affect throughput.

### `LogDirectoryPath`

The path used by the native log writer. Defaults to `logs/log.txt` relative to the working directory. The name suggests a directory, but the value is a full file path.

Change this when:

- Your app has a specific logging convention (for example, centralized log folder).
- You need per-environment log files (`logs/dev.log`, `logs/prod.log`).
- The working directory is not writable in your deployment.

```csharp
preset.EngineParameters.LogDirectoryPath = @"C:\logs\aspose-llm\app.log";
```

### `DefaultThreads`

Thread count used when `ContextParameters.NThreads` is not set explicitly. The default is `Environment.ProcessorCount - 1` — one fewer than the total logical cores — to leave one core for the rest of your application.

Override in two situations:

- **Dedicated inference machine** — use `Environment.ProcessorCount` for maximum throughput.
- **Tight envelope (containers, laaS)** — use a fixed smaller number to stay inside a CPU quota.

```csharp
preset.EngineParameters.DefaultThreads = 4;
```

For finer control over threading during generation, set [`ContextParameters.NThreads`](/net/developer-reference/parameters/context/) and `NThreadsBatch` directly — those override `DefaultThreads` when set.

## Typical recipes

### Development with debug logs

```csharp
var preset = new Qwen25Preset();
preset.EngineParameters.EnableDebugLogging = true;
preset.EngineParameters.LogDirectoryPath = "logs/debug.log";
```

### Production on a dedicated server

```csharp
var preset = new Qwen25Preset();
preset.EngineParameters.ModelCachePath = @"/var/lib/aspose-llm/models";
preset.EngineParameters.EnableDebugLogging = false;
preset.EngineParameters.LogDirectoryPath = @"/var/log/aspose-llm/app.log";
preset.EngineParameters.DefaultThreads = Environment.ProcessorCount;
```

### Container with mounted model volume

```csharp
var preset = new Qwen25Preset();
preset.EngineParameters.ModelCachePath = "/models";           // volume mount
preset.EngineParameters.LogDirectoryPath = "/var/log/app.log"; // volume mount
preset.EngineParameters.DefaultThreads = 4;                   // CPU quota
```

## What's next

- [Context parameters](/net/developer-reference/parameters/context/) — threads per inference call, batch sizes.
- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — where native `llama.cpp` binaries are cached.
- [Logging and diagnostics](/net/developer-reference/) — `ILogger` integration and native log tags (planned reference page in a future release).

---
weight: 40
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/binary-manager/binary-path/
feedback: LLMNET
version: 26.5.0
title: BinaryPath
description: Cache folder for native llama.cpp binaries in Aspose.LLM for .NET — defaults to a per-user cache; override for shared or pre-populated deployments.
keywords:
- BinaryPath
- native binary cache
- runtimes
- offline
---

`BinaryPath` is the folder where native `llama.cpp` binaries are cached. First-run downloads land here; subsequent runs for the same `ReleaseTag` and acceleration load from this cache.

## Quick reference

| | |
|---|---|
| **Type** | `string` |
| **Default** | `<LocalAppData>/Aspose.LLM/runtimes` |
| **Category** | Native binary source |
| **Field on** | `BinaryManagerParameters.BinaryPath` |

## What it does

- On Windows: `%LOCALAPPDATA%\Aspose.LLM\runtimes`.
- On Linux / macOS: equivalent `Environment.SpecialFolder.LocalApplicationData` path joined with `Aspose.LLM/runtimes`.
- Override to control where binaries are cached.

The binary manager subdivides this folder per `ReleaseTag` × acceleration variant, so multiple configurations can coexist.

## When to change it

| Scenario | Value |
|---|---|
| Default | unset |
| Shared cache across services | `/srv/aspose-llm/runtimes` (or equivalent) |
| Read-only root filesystem | Writable volume path |
| Pre-populated deployment (offline) | Path to bundled binaries |
| Docker / container | Volume mount path |

## Example

```csharp
var preset = new Qwen25Preset();
preset.BinaryManagerParameters.BinaryPath = @"/opt/aspose-llm/runtimes";

using var api = AsposeLLMApi.Create(preset);
```

For air-gapped deployments:

```csharp
// Copy pre-downloaded binaries to a location accessible on the target host,
// then point BinaryPath there.
preset.BinaryManagerParameters.BinaryPath = "/opt/aspose-llm/runtimes";
preset.EngineParameters.ModelCachePath = "/opt/aspose-llm/models";
```

## Interactions

- [`ReleaseTag`](/net/developer-reference/parameters/binary-manager/release-tag/) — cache is subdivided per tag.
- [`PreferredAcceleration`](/net/developer-reference/parameters/binary-manager/preferred-acceleration/) — cache also subdivides per acceleration.
- [`EngineParameters.ModelCachePath`](/net/developer-reference/parameters/engine/model-cache-path/) — separate cache for model files.

## What's next

- [Offline deployment](/net/use-cases/offline-deployment/) — pre-populating `BinaryPath`.
- [Binary manager hub](/net/developer-reference/parameters/binary-manager/) — all binary-manager knobs.

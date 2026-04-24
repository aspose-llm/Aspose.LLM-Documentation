---
weight: 10
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/engine/model-cache-path/
feedback: LLMNET
version: 26.5.0
title: ModelCachePath
description: Folder where Aspose.LLM for .NET stores downloaded model files — defaults to a per-user cache; override for shared storage or non-default drives.
keywords:
- ModelCachePath
- model cache
- Hugging Face
- storage
- disk
---

`ModelCachePath` is the folder where the engine stores downloaded model files. First-run downloads from Hugging Face (or the Aspose catalog) land here; subsequent runs load from this cache.

## Quick reference

| | |
|---|---|
| **Type** | `string?` |
| **Default** | `<LocalAppData>/Aspose.LLM/models` |
| **Category** | Engine |
| **Field on** | `EngineParameters.ModelCachePath` |

## What it does

- On Windows: `%LOCALAPPDATA%\Aspose.LLM\models`.
- On Linux / macOS: the equivalent `Environment.SpecialFolder.LocalApplicationData` path, joined with `Aspose.LLM/models`.
- Overriding sets a custom directory.

At each `AsposeLLMApi.Create`, the engine checks this folder for the preset's model file. If present, load. If absent, download from the resolved source ([`BaseModelSourceParameters`](/net/developer-reference/parameters/model-source/)), save into this folder, and load.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` or unset |
| Shared cache across multiple applications | `/srv/aspose-llm/models` (or equivalent) |
| Faster storage (SSD when `LocalAppData` is on HDD) | Path to SSD folder |
| Docker / container with mounted volume | `/models` (volume mount) |
| Pre-populated offline deployment | Path to the pre-populated cache |

## Example

```csharp
var preset = new Qwen25Preset();
preset.EngineParameters.ModelCachePath = @"D:\models";
// Large models will land on drive D instead of C.

using var api = AsposeLLMApi.Create(preset);
```

Container pattern:

```csharp
preset.EngineParameters.ModelCachePath = "/models";  // Docker volume mount
```

## Interactions

- [`BaseModelSourceParameters`](/net/developer-reference/parameters/model-source/) — resolution order; the cached file must match the requested source.
- [`BinaryManagerParameters.BinaryPath`](/net/developer-reference/parameters/binary-manager/binary-path/) — separate cache for native binaries; different folder.

## What's next

- [Offline deployment use case](/net/use-cases/offline-deployment/) — pre-populating the cache.
- [Model source parameters](/net/developer-reference/parameters/model-source/) — what gets cached.
- [Engine parameters hub](/net/developer-reference/parameters/engine/) — all engine knobs.

---
weight: 10
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-source/model-file-path/
feedback: LLMNET
version: 26.5.0
title: ModelFilePath
description: Highest-priority model source in Aspose.LLM for .NET — a local file path; bypasses all download logic.
keywords:
- ModelFilePath
- local model
- file path
- priority
---

`ModelFilePath` is the highest-priority source for the model file. When set to a non-empty path, the engine loads directly from disk and skips all download logic.

## Quick reference

| | |
|---|---|
| **Type** | `string?` |
| **Default** | `null` |
| **Priority** | 1 (highest; wins over `AsposeModelId` and Hugging Face fields) |
| **Category** | Model source |
| **Field on** | `ModelSourceParameters.ModelFilePath` |

## What it does

- `null` or empty — skip this priority level; fall through to [`AsposeModelId`](/net/developer-reference/parameters/model-source/aspose-model-id/), then Hugging Face fields.
- A path — load the GGUF from that file. No download.

The file must be accessible and readable by the process.

## When to change it

| Scenario | Value |
|---|---|
| Default (use Hugging Face or Aspose catalog) | `null` |
| Air-gapped deployment with pre-downloaded model | Absolute path |
| Custom local GGUF | Path to the file |
| Testing a model before mirroring to Hugging Face | Local path |

## Example

```csharp
var preset = new Qwen25Preset();
preset.BaseModelSourceParameters.ModelFilePath =
    @"C:\models\Qwen2.5-7B-Instruct-Q4_K_M.gguf";
// Bypass the Hugging Face values on the preset.

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`AsposeModelId`](/net/developer-reference/parameters/model-source/aspose-model-id/) — priority 2; ignored when `ModelFilePath` is set.
- [`HuggingFaceRepoId`](/net/developer-reference/parameters/model-source/hugging-face-repo-id/) + [`HuggingFaceFileName`](/net/developer-reference/parameters/model-source/hugging-face-file-name/) — priority 3; ignored when `ModelFilePath` is set.
- [`EngineParameters.ModelCachePath`](/net/developer-reference/parameters/engine/model-cache-path/) — cache is not used when loading from an explicit `ModelFilePath`.

## What's next

- [Model source hub](/net/developer-reference/parameters/model-source/) — resolution order overview.
- [Bring your own GGUF](/net/use-cases/bring-your-own-gguf/) — full custom-model workflow.
- [Offline deployment](/net/use-cases/offline-deployment/) — air-gapped setup.

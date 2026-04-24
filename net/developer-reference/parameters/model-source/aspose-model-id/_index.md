---
weight: 20
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-source/aspose-model-id/
feedback: LLMNET
version: 26.5.0
title: AsposeModelId
description: Second-priority model source in Aspose.LLM for .NET — identifier into Aspose's internal model catalog; used when ModelFilePath is unset.
keywords:
- AsposeModelId
- Aspose catalog
- model ID
- priority
---

`AsposeModelId` is the second-priority source — an identifier into Aspose's internal model catalog hosted at `releases.aspose.com`. The engine downloads by ID when no [`ModelFilePath`](/net/developer-reference/parameters/model-source/model-file-path/) is set.

## Quick reference

| | |
|---|---|
| **Type** | `string?` |
| **Default** | `null` |
| **Priority** | 2 (loses to `ModelFilePath`; wins over Hugging Face fields) |
| **Category** | Model source |
| **Field on** | `ModelSourceParameters.AsposeModelId` |

## What it does

When set, the engine resolves the ID against the Aspose model catalog and downloads the named file into [`EngineParameters.ModelCachePath`](/net/developer-reference/parameters/engine/model-cache-path/). Subsequent runs use the cache.

- `null` — skip this priority; fall through to Hugging Face fields.
- An ID — resolve against the Aspose catalog.

Built-in presets do not set `AsposeModelId` — they use Hugging Face. Use this field when Aspose ships a specific model optimized or licensed for SDK consumers.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Load from Aspose catalog by ID | Specific ID |

## Example

```csharp
var preset = new Qwen25Preset();
preset.BaseModelSourceParameters.AsposeModelId = "aspose-llm-qwen-25-7b-q4km";
preset.BaseModelSourceParameters.HuggingFaceRepoId = null; // clear the default so priority-3 fall-through isn't used
```

## Interactions

- [`ModelFilePath`](/net/developer-reference/parameters/model-source/model-file-path/) — if set, overrides `AsposeModelId`.
- Hugging Face fields — only used when both `ModelFilePath` and `AsposeModelId` are null.

## What's next

- [Model source hub](/net/developer-reference/parameters/model-source/) — resolution order.
- [HuggingFaceRepoId](/net/developer-reference/parameters/model-source/hugging-face-repo-id/) — priority-3 source.

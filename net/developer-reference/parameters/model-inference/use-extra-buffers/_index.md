---
weight: 80
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/use-extra-buffers/
feedback: LLMNET
version: 26.5.0
title: UseExtraBuffers
description: Advanced llama.cpp flag in Aspose.LLM for .NET — enables additional buffer types for weight repacking; leave at default unless instructed.
keywords:
- UseExtraBuffers
- weight repacking
- llama.cpp
- advanced
---

`UseExtraBuffers` is an advanced llama.cpp flag that enables extra buffer types used by the weight-repacking path. Rarely tuned in practice; leave at default unless specifically instructed.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (use native default) |
| **Category** | Advanced |
| **Field on** | `ModelInferenceParameters.UseExtraBuffers` |

## What it does

Internal to llama.cpp. Controls whether the engine uses additional buffer types during weight repacking for specific hardware paths. The exact behavior depends on the backend and release tag.

- `null` — native default. Correct for almost all users.
- `true` / `false` — override. Not useful without specific backend expertise.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Backend-specific advice from SDK docs | As instructed |

Do not speculate. If you are not sure whether you need this flag, you do not need it.

## Example

```csharp
var preset = new Qwen25Preset();
// preset.BaseModelInferenceParameters.UseExtraBuffers = null; // default
```

## Interactions

- Backend-specific. Effects vary by acceleration variant.

## What's next

- [Model inference hub](/net/developer-reference/parameters/model-inference/) — all inference knobs.

---
weight: 30
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-source/hugging-face-repo-id/
feedback: LLMNET
version: 26.5.0
title: HuggingFaceRepoId
description: Third-priority model source in Aspose.LLM for .NET — Hugging Face repository identifier; paired with HuggingFaceFileName.
keywords:
- HuggingFaceRepoId
- Hugging Face
- repo ID
- priority
---

`HuggingFaceRepoId` identifies a Hugging Face repository to download the model from. Used when neither [`ModelFilePath`](/net/developer-reference/parameters/model-source/model-file-path/) nor [`AsposeModelId`](/net/developer-reference/parameters/model-source/aspose-model-id/) is set.

## Quick reference

| | |
|---|---|
| **Type** | `string?` |
| **Default** | `null` (set by built-in presets) |
| **Priority** | 3 (lowest) |
| **Category** | Model source |
| **Field on** | `ModelSourceParameters.HuggingFaceRepoId` |

## What it does

The engine downloads `HuggingFaceFileName` from this repository when higher-priority fields are unset. Built-in presets populate this field — for example, `Qwen25Preset` sets it to `"bartowski/Qwen2.5-7B-Instruct-GGUF"`.

- `null` — no Hugging Face download.
- An Id like `"owner/repo"` — download from that repo.

## When to change it

| Scenario | Value |
|---|---|
| Default (use preset's value) | whatever the preset sets |
| Switch to a different quantization publisher | Different owner/repo |
| Use a custom GGUF in a private Hugging Face repo | Private-repo ID (requires HF token configuration) |

## Example

```csharp
var preset = new Qwen25Preset();
preset.BaseModelSourceParameters.HuggingFaceRepoId = "unsloth/Qwen2.5-7B-Instruct-GGUF";
preset.BaseModelSourceParameters.HuggingFaceFileName = "Qwen2.5-7B-Instruct-Q4_K_M.gguf";
```

## Interactions

- [`HuggingFaceFileName`](/net/developer-reference/parameters/model-source/hugging-face-file-name/) — the specific file within the repo.
- [`ModelFilePath`](/net/developer-reference/parameters/model-source/model-file-path/), [`AsposeModelId`](/net/developer-reference/parameters/model-source/aspose-model-id/) — higher-priority sources; override this.

## What's next

- [HuggingFaceFileName](/net/developer-reference/parameters/model-source/hugging-face-file-name/) — companion file selector.
- [Bring your own GGUF](/net/use-cases/bring-your-own-gguf/) — custom-model workflow.
- [Model source hub](/net/developer-reference/parameters/model-source/) — resolution order.

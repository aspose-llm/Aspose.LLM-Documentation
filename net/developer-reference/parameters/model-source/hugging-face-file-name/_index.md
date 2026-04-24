---
weight: 40
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-source/hugging-face-file-name/
feedback: LLMNET
version: 26.5.0
title: HuggingFaceFileName
description: Specific file name within a Hugging Face repository in Aspose.LLM for .NET — selects which quantization to download.
keywords:
- HuggingFaceFileName
- Hugging Face
- file name
- quantization selection
---

`HuggingFaceFileName` names the specific file within [`HuggingFaceRepoId`](/net/developer-reference/parameters/model-source/hugging-face-repo-id/) to download. Many Hugging Face repos host several GGUF variants (different quantizations) — this field picks one.

## Quick reference

| | |
|---|---|
| **Type** | `string?` |
| **Default** | `null` (set by built-in presets) |
| **Category** | Model source |
| **Field on** | `ModelSourceParameters.HuggingFaceFileName` |

## What it does

Used alongside `HuggingFaceRepoId`. Built-in presets pick a sensible default (usually `Q4_K_M`); override to switch quantization.

For `Qwen25Preset`, the default is `"Qwen2.5-7B-Instruct-Q4_K_M.gguf"`. Common alternatives in the same repo: `Q4_0`, `Q5_K_M`, `Q6_K`, `Q8_0`.

## When to change it

| Scenario | Value |
|---|---|
| Default (preset's value) | unchanged |
| Higher quality, more memory | `Q5_K_M`, `Q6_K`, `Q8_0` |
| Lower memory, lower quality | `Q4_0`, `IQ3_M`, `IQ2_XXS` |
| Testing a specific quant | Full file name from Hugging Face listing |

See [Understand quantization](/net/how-to/understand-quantization/) for a primer on which variant to pick.

## Example

```csharp
var preset = new Qwen25Preset();
// Switch to Q8 for higher quality — about 2x the file size
preset.BaseModelSourceParameters.HuggingFaceFileName = "Qwen2.5-7B-Instruct-Q8_0.gguf";
```

Or smaller for memory-tight hardware:

```csharp
preset.BaseModelSourceParameters.HuggingFaceFileName = "Qwen2.5-7B-Instruct-IQ3_M.gguf";
```

## Interactions

- [`HuggingFaceRepoId`](/net/developer-reference/parameters/model-source/hugging-face-repo-id/) — the repo containing the file.
- [`ModelFilePath`](/net/developer-reference/parameters/model-source/model-file-path/) — higher priority; overrides Hugging Face download.

## What's next

- [Understand quantization](/net/how-to/understand-quantization/) — Q4 vs Q5 vs Q8 primer.
- [HuggingFaceRepoId](/net/developer-reference/parameters/model-source/hugging-face-repo-id/) — companion repo selector.
- [Model source hub](/net/developer-reference/parameters/model-source/) — resolution order.

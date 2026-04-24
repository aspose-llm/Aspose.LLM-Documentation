---
weight: 70
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/check-tensors/
feedback: LLMNET
version: 26.5.0
title: CheckTensors
description: Validate model tensor data on load in Aspose.LLM for .NET — adds startup time; use when suspecting corrupt GGUF.
keywords:
- CheckTensors
- tensor validation
- corrupt file
- model loading
---

`CheckTensors` validates every tensor in the GGUF file during load. Adds noticeable startup time but catches corrupted or truncated models early — cleaner than a mysterious runtime error later.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (false — no validation) |
| **Category** | Model loading |
| **Field on** | `ModelInferenceParameters.CheckTensors` |

## What it does

- `null` or `false` — default. Skip validation; trust the GGUF file.
- `true` — walk every tensor, verify shape and data consistency. Fails early with a clear error if the file is corrupted.

Validation can add 10-30 seconds to startup on a 7B model; longer on larger models. Not intended for production. Use when you have a suspicion about file integrity.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| First-run validation after download from an untrusted source | `true` |
| Debugging a corrupted GGUF suspicion | `true` |
| Production (after validation passed once) | `null` |

## Example

```csharp
var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.CheckTensors = true;
// Startup takes longer; validates every tensor against shape + data consistency.

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`UseMemoryMapping`](/net/developer-reference/parameters/model-inference/use-memory-mapping/) — validation walks mapped or read-in data; either mode works.
- [`GpuLayers`](/net/developer-reference/parameters/model-inference/gpu-layers/) — validation happens on host memory before offload.

## What's next

- [Model not loading troubleshooting](/net/troubleshooting/model-not-loading/) — when `CheckTensors` helps.
- [Model inference hub](/net/developer-reference/parameters/model-inference/) — all inference knobs.

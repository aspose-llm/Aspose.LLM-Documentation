---
weight: 10
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/developer-reference/presets/
feedback: LLMNET
title: Presets
description: Using presets to configure Aspose.LLM for .NET.
keywords:
- preset
- PresetCoreBase
- model
---

A **preset** defines the model source, context size, chat options, and sampler parameters used by the API. You pass a preset when creating the API instance.

## Base type

All presets derive from `PresetCoreBase` (namespace `Aspose.LLM.Abstractions.Parameters.Presets`). This type exposes:

- **BaseModelSourceParameters** — Where to load the main model (e.g. Hugging Face repo and file name).
- **MmprojSourceParameters** — Optional vision projector (for vision presets).
- **BinaryManagerParameters** — Release tag and binary management.
- **EngineParameters** — Engine and cache settings.
- **ChatParameters** — Chat template and role names.
- **ContextParameters** — Context size, batch sizes, threading, GPU layers.
- **SamplerParameters** — Sampling (temperature, top-p, etc.).
- **BaseModelInferenceParameters** — Inference options (GPU layers, memory mapping, etc.).

## Creating the API with a preset

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(preset);
```

You can use a built-in preset (e.g. `Qwen25Preset`, `Gemma3Preset`, `Llama32Preset`) or a custom class that extends `PresetCoreBase` and sets the properties you need.

## Default preset

`api.GetDefaultPreset()` returns a preset filled with the engine’s default parameters. You can use it to create another API instance or inspect defaults. For raw parameter tuples, use `await api.GetDefaultParametersAsync()`.

## Supported presets

See [Supported presets](/net/product-overview/supported-presets/) in the product overview for a list of built-in text and vision presets.

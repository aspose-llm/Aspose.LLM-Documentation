---
weight: 15
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/
feedback: LLMNET
version: 26.5.0
title: Parameters
description: Reference for the eight parameter bags exposed by PresetCoreBase in Aspose.LLM for .NET — model source, inference, context, chat, sampler, engine, binary manager, multimodal context.
keywords:
- parameters
- PresetCoreBase
- configuration
- settings
- reference
---

Every preset in Aspose.LLM for .NET exposes nine independent parameter bags — small POCO classes that group related settings. Tune any bag before calling `AsposeLLMApi.Create(preset)` to change how the engine loads and runs the model.

This section covers each bag in detail: the public fields, their types and defaults, and the typical reasons to change them.

## At a glance

| Bag | What it controls | Frequently changed for |
|---|---|---|
| [Model source](/net/developer-reference/parameters/model-source/) | Where the model file comes from | Local files, Hugging Face repos, custom caches |
| [Model inference](/net/developer-reference/parameters/model-inference/) | How the model loads into memory | GPU offload, tensor split, memory mapping |
| [Context](/net/developer-reference/parameters/context/) | Runtime context and batch sizes | Context length, flash attention, KV cache quantization |
| [Chat](/net/developer-reference/parameters/chat/) | Session-level conversation settings | System prompt, max tokens, cache cleanup |
| [Sampler](/net/developer-reference/parameters/sampler/) | Token sampling behavior | Temperature, penalties, DRY, Mirostat |
| [Engine](/net/developer-reference/parameters/engine/) | Engine-wide defaults | Cache path, debug logging, default threads |
| [Binary manager](/net/developer-reference/parameters/binary-manager/) | Native binary deployment | Release tag, preferred acceleration backend |
| [Multimodal context](/net/developer-reference/parameters/multimodal-context/) | mtmd (vision) context | GPU offload for projector, verbosity |

`PresetCoreBase` exposes **nine** properties, but only **eight** parameter class types — the `MmprojSourceParameters` property for the vision projector uses the same `ModelSourceParameters` type as `BaseModelSourceParameters`. Both are covered on the [Model source](/net/developer-reference/parameters/model-source/) page.

## General guidance

- **Start with a built-in preset.** Presets ship with carefully chosen defaults for each model family. Override only fields you have a concrete reason to change.
- **Every field is a public get/set property.** Mutations on the preset before `Create` take effect; after `Create`, the engine has already read the preset and further changes are ignored.
- **Every bag is lazy-initialized** on `PresetCoreBase`. Accessing any property gives you an instance even if you never set it — safe to write `preset.ContextParameters.ContextSize = 8192` without a null check.
- **Null means "use native default"** on nullable fields. Set an explicit value only when you want to override.

## Sections

- [Model source](/net/developer-reference/parameters/model-source/) — model file resolution order and cache layout.
- [Model inference](/net/developer-reference/parameters/model-inference/) — model-load knobs (GPU, memory, tensor split).
- [Context](/net/developer-reference/parameters/context/) — `llama.cpp` context parameters.
- [Chat](/net/developer-reference/parameters/chat/) — session-level conversation settings.
- [Sampler](/net/developer-reference/parameters/sampler/) — sampler knobs.
- [Engine](/net/developer-reference/parameters/engine/) — engine-wide defaults.
- [Binary manager](/net/developer-reference/parameters/binary-manager/) — native binary deployment.
- [Multimodal context](/net/developer-reference/parameters/multimodal-context/) — `mtmd` context for vision.

## What's next

- [Presets](/net/developer-reference/presets/) — how `PresetCoreBase` composes the eight bags.
- [Custom preset](/net/use-cases/custom-preset/) — override patterns for the most common scenarios.
- [Supported presets](/net/product-overview/supported-presets/) — built-in presets with their default parameter values.

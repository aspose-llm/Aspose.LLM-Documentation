---
weight: 80
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/multimodal-context/
feedback: LLMNET
version: 26.5.0
title: Multimodal context parameters
description: Configure the mtmd (multimodal) context in Aspose.LLM for .NET — GPU offload for the vision projector, timing diagnostics, thread count, verbosity, and media marker.
keywords:
- MultimodalContextParameters
- MtmdContextParameters
- mtmd
- vision
- projector
- mmproj
- media marker
---

`MultimodalContextParameters` — exposed on the preset as `MtmdContextParameters` — configures the `mtmd` context used by vision presets to evaluate image tokens. The base text model is configured by [`ContextParameters`](/net/developer-reference/parameters/context/); this bag covers only the multimodal layer.

Only vision presets use these settings. On text-only presets the bag is instantiated but has no effect.

## Class reference

```csharp
namespace Aspose.LLM.Abstractions.Parameters;

public class MultimodalContextParameters
{
    public bool? UseGpu { get; set; }
    public bool? PrintTimings { get; set; }
    public int? ThreadCount { get; set; }
    public int? Verbosity { get; set; }
    public string? MediaMarker { get; set; }
}
```

Every field is nullable. A `null` value means "use the native `mtmd` default" — override only when you have a specific reason.

## Detailed field reference

Each field has a dedicated page with full defaults, scenario tables, code examples, and interactions.

- [UseGpu](/net/developer-reference/parameters/multimodal-context/use-gpu/)
- [PrintTimings](/net/developer-reference/parameters/multimodal-context/print-timings/)
- [ThreadCount](/net/developer-reference/parameters/multimodal-context/thread-count/)
- [Verbosity](/net/developer-reference/parameters/multimodal-context/verbosity/)
- [MediaMarker](/net/developer-reference/parameters/multimodal-context/media-marker/)

## Fields

| Field | Type | Default | Purpose |
|---|---|---|---|
| `UseGpu` | `bool?` | native default | Whether to offload the vision projector to GPU. |
| `PrintTimings` | `bool?` | native default | Emit per-step timing diagnostics from the `mtmd` layer. |
| `ThreadCount` | `int?` | native default | Threads used by `mtmd` processing. |
| `Verbosity` | `int?` | native default | Log level for the `mtmd` layer. |
| `MediaMarker` | `string?` | native default | Placeholder token text that marks image positions in the prompt. |

### `UseGpu`

Controls whether the vision projector (`mmproj`) runs on the GPU alongside the base model. The `mmproj` is typically small (200 MB - 2 GB), so GPU offload is fast even on modest hardware.

- `null` — delegate to `mtmd`'s auto-detection (currently: GPU if available).
- `true` — force GPU.
- `false` — force CPU. Use when you have limited GPU memory and want to spend it entirely on the base model.

```csharp
preset.MtmdContextParameters.UseGpu = false; // keep GPU memory for the base model
```

### `PrintTimings`

Enables `mtmd`'s built-in per-step timing logs — the time spent tokenizing images, running the projector, and evaluating chunks. Useful for diagnosing slow first-response latency on vision queries.

```csharp
preset.MtmdContextParameters.PrintTimings = true;
```

Leave this `null` (off) in production. Timing logs add overhead and flood the output.

### `ThreadCount`

Threads used for CPU-side `mtmd` work (image preprocessing, CPU portions of the projector). When `null`, `mtmd` follows its own heuristic — usually half the logical cores.

Override when:

- The rest of your application needs more cores and `mtmd` is single-shot work.
- You run multiple vision requests concurrently and want to cap each one's CPU footprint.

```csharp
preset.MtmdContextParameters.ThreadCount = 2;
```

### `Verbosity`

Log verbosity for the `mtmd` layer. The native layer accepts an integer; the typical mapping is:

| Value | Level |
|---|---|
| `0` | Error |
| `1` | Warn |
| `2` | Info |
| `3` | Debug |

```csharp
preset.MtmdContextParameters.Verbosity = 3; // debug — useful when images are tokenized unexpectedly
```

Keep verbosity low in production (`0` or `1`). Higher levels emit tagged lines that need post-processing to be useful — see the `parse_mm_logs.zsh` helper script in the Aspose.LLM SDK repository.

### `MediaMarker`

Placeholder text used in the chat template to mark where images are inserted. The default is the chat-template-specific marker (different per model family — LLaVA, Qwen-VL, Gemma-Vision, and others have different tokens). Override only if you understand the model's prompt format and need a non-standard marker.

```csharp
preset.MtmdContextParameters.MediaMarker = "<|image|>";
```

In nearly all cases, leave this `null`. The correct marker is selected automatically from the model's metadata.

## Typical recipes

### Default vision configuration

```csharp
var preset = new Qwen3VL2BPreset();
// MtmdContextParameters stays at defaults — all fields null.

using var api = AsposeLLMApi.Create(preset);
```

### Debug slow image processing

```csharp
var preset = new Qwen3VL2BPreset();
preset.MtmdContextParameters.PrintTimings = true;
preset.MtmdContextParameters.Verbosity = 3;

using var api = AsposeLLMApi.Create(preset, logger);
// Inspect logs for per-stage mtmd timings.
```

### CPU-only projector to save GPU memory

```csharp
var preset = new Qwen3VL2BPreset();
preset.MtmdContextParameters.UseGpu = false;                  // projector on CPU
preset.BaseModelInferenceParameters.GpuLayers = 999;          // base model fully on GPU
```

On a GPU tight for memory, keeping the projector on CPU trades some first-token latency for more headroom for the base model and KV cache.

## What's next

- [Supported presets — vision](/net/product-overview/supported-presets/#vision-presets) — built-in vision presets and their `mmproj` sources.
- [Model source parameters](/net/developer-reference/parameters/model-source/) — configure the vision projector's download source.
- [Attaching images](/net/use-cases/) — vision use case (planned in a future release).

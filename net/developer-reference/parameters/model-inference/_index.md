---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-inference/
feedback: LLMNET
version: 26.5.0
title: Model inference parameters
description: Configure how a model loads into memory in Aspose.LLM for .NET — GPU layer offload, tensor split across GPUs, memory mapping, memory locking, tensor validation, and metadata overrides.
keywords:
- ModelInferenceParameters
- GpuLayers
- TensorSplit
- SplitMode
- MainGpu
- memory mapping
- mmap
- KvOverrides
---

`ModelInferenceParameters` controls how the engine loads a model into memory: how many layers to offload to GPU, how to split across multiple GPUs, whether to use memory mapping, and how to override GGUF metadata at runtime.

Most fields are nullable — a `null` value means "use the native default". Set an explicit value only when you need to override.

## Class reference

```csharp
namespace Aspose.LLM.Abstractions.Parameters;

public class ModelInferenceParameters
{
    public int? GpuLayers { get; set; }
    public bool? UseMemoryMapping { get; set; }
    public bool? UseMemoryLocking { get; set; }
    public int? MainGpu { get; set; }
    public LlamaSplitMode? SplitMode { get; set; }
    public bool? VocabOnly { get; set; }
    public bool? CheckTensors { get; set; }
    public bool? UseExtraBuffers { get; set; }
    public float[]? TensorSplit { get; set; }
    public ModelKeyValueOverride[]? KvOverrides { get; set; }
}
```

## Fields

| Field | Type | Default | Purpose |
|---|---|---|---|
| `GpuLayers` | `int?` | native default | Number of model layers to offload to GPU. `0` = CPU only, `999` = full offload. |
| `UseMemoryMapping` | `bool?` | native default (`true`) | Map the GGUF file instead of reading it in. Reduces startup time and memory copying. |
| `UseMemoryLocking` | `bool?` | native default (`false`) | Lock model memory to prevent OS paging. Needs `mlock` / `VirtualLock` privileges. |
| `MainGpu` | `int?` | `0` | Index of the GPU used when `SplitMode` is `None`. |
| `SplitMode` | `LlamaSplitMode?` | native default | How to split the model across multiple GPUs. |
| `VocabOnly` | `bool?` | native default (`false`) | Load only the vocabulary without weights. Used for tokenizer-only scenarios. |
| `CheckTensors` | `bool?` | native default (`false`) | Validate tensor data on load. Adds startup time; helpful for diagnosing corrupted GGUF files. |
| `UseExtraBuffers` | `bool?` | native default | Use extra buffer types for weight repacking. Advanced. |
| `TensorSplit` | `float[]?` | equal split | Proportions per GPU when `SplitMode` splits across devices. |
| `KvOverrides` | `ModelKeyValueOverride[]?` | none | Runtime overrides for GGUF metadata keys. |

### `GpuLayers`

Controls GPU offload. Each transformer layer lives either in system RAM (CPU inference) or GPU VRAM (GPU inference). Partial offload is supported: you can put the first N layers on the GPU and keep the rest on the CPU.

| Value | Behavior |
|---|---|
| `0` | CPU only. No GPU memory allocated for model weights. |
| A layer count | Offload the first N layers to GPU, keep the remaining on CPU. |
| `999` (or any value ≥ the model's layer count) | Full GPU offload. Idiomatic way to request "put everything on the GPU". |
| `null` | Use the native default. |

Pair with a GPU-capable `BinaryManagerParameters.PreferredAcceleration` — setting `GpuLayers = 999` on a CPU-only binary silently keeps the model on the CPU.

```csharp
preset.BaseModelInferenceParameters.GpuLayers = 999;
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
```

### `UseMemoryMapping`

When `true` (default on most platforms), the engine memory-maps the GGUF file so the OS streams it in on demand. This reduces startup time and avoids copying the full model into RAM before inference.

Set to `false` only for rare scenarios — network file systems that do not support `mmap`, or environments where you want the file fully loaded before the first token. Disabling `mmap` doubles peak memory during load (the read buffer plus the mapped copy).

### `UseMemoryLocking`

When `true`, the engine calls `mlock` (Linux/macOS) or `VirtualLock` (Windows) on the model's memory so the OS does not page it out. Requires elevated privileges or raised ulimits.

Leave `null` or `false` in most deployments. Enable only when the model is swapped out under memory pressure and you have the system tuning to support locking.

### `MainGpu` and `SplitMode`

On multi-GPU hosts, `SplitMode` decides how to place the model and `MainGpu` selects the primary device when the mode is not splitting.

| `SplitMode` | Behavior |
|---|---|
| `LLAMA_SPLIT_MODE_NONE` | Single GPU. Whole model on device `MainGpu`. |
| `LLAMA_SPLIT_MODE_LAYER` | Split layers across GPUs. KV cache follows layers. |
| `LLAMA_SPLIT_MODE_ROW` | Split layers and rows across GPUs. Uses tensor parallelism where supported. |

```csharp
using Aspose.LLM.Abstractions.Parameters;

// Single GPU, use device 1:
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_NONE;
preset.BaseModelInferenceParameters.MainGpu = 1;

// Split across all GPUs, layer mode:
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_LAYER;
preset.BaseModelInferenceParameters.TensorSplit = null; // equal distribution
```

### `TensorSplit`

Proportion of the model placed on each GPU. The array length should match the number of available GPUs.

- `null` — equal distribution.
- Explicit array — values are normalized to sum to 1. For example, `[2, 1]` on two GPUs places 67 % on GPU 0 and 33 % on GPU 1.

Useful when GPUs have different memory sizes (for example, a 24 GB card paired with a 12 GB card):

```csharp
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_LAYER;
preset.BaseModelInferenceParameters.TensorSplit = new float[] { 2.0f, 1.0f };
```

### `VocabOnly`

When `true`, the engine loads only the model's vocabulary, skipping the weights. The model is not usable for generation in this state — it is a tokenizer-only configuration for rare tooling scenarios.

Leave `null` (or `false`) for any normal inference use.

### `CheckTensors`

When `true`, the engine validates every tensor's data during load. Adds significant startup time but catches corrupted GGUF files early. Useful when testing a new download or a model from an untrusted source; leave `null` in production.

### `UseExtraBuffers`

Enables additional buffer types used by weight repacking paths in `llama.cpp`. Advanced; most users should leave it `null`.

### `KvOverrides`

Overrides specific keys in the GGUF metadata at load time. Each override targets a single metadata key and provides the new typed value.

```csharp
preset.BaseModelInferenceParameters.KvOverrides = new[]
{
    new ModelKeyValueOverride
    {
        Key = "llama.rope.scaling.type",
        Type = ModelKvOverrideType.String,
        StringValue = "yarn",
    },
    new ModelKeyValueOverride
    {
        Key = "llama.context_length",
        Type = ModelKvOverrideType.Int,
        IntValue = 131072,
    },
};
```

The `ModelKeyValueOverride` class carries one of four typed values depending on `Type`:

| `Type` | Value field | Example keys |
|---|---|---|
| `Int` | `IntValue` | `llama.context_length`, `llama.embedding_length` |
| `Float` | `FloatValue` | `llama.rope.freq_base`, `llama.rope.scaling.factor` |
| `Bool` | `BoolValue` | Model-specific boolean flags |
| `String` | `StringValue` | `general.architecture`, `llama.rope.scaling.type` |

Use overrides with care — wrong metadata makes the model load incorrectly or silently produce garbage.

## Typical recipes

### CPU-only inference

```csharp
var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.GpuLayers = 0;
```

### Full GPU offload

```csharp
var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.GpuLayers = 999;
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
```

### Partial offload on a memory-constrained GPU

A 12 GB GPU cannot fit a full 8B Q4_K_M model plus KV cache. Offload the first 28 layers and keep the rest on the CPU:

```csharp
var preset = new Qwen3Preset(); // 8B, 32 layers
preset.BaseModelInferenceParameters.GpuLayers = 28;
preset.BaseModelInferenceParameters.UseMemoryMapping = true;
```

Benchmark to find the right split for your hardware — "offload until VRAM is ~1-2 GB short of full".

### Two unequal GPUs

A 24 GB GPU paired with a 12 GB GPU:

```csharp
preset.BaseModelInferenceParameters.SplitMode = LlamaSplitMode.LLAMA_SPLIT_MODE_LAYER;
preset.BaseModelInferenceParameters.TensorSplit = new float[] { 2.0f, 1.0f };
preset.BaseModelInferenceParameters.GpuLayers = 999;
```

### Validate a suspect GGUF

```csharp
preset.BaseModelInferenceParameters.CheckTensors = true;
// After a clean load, switch back to the default for production.
```

## What's next

- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — pair with `PreferredAcceleration` to select the right native binary.
- [Context parameters](/net/developer-reference/parameters/context/) — KV cache configuration and batch sizes that interact with GPU offload.
- [System requirements](/net/system-requirements/) — GPU backends and their driver / runtime requirements.

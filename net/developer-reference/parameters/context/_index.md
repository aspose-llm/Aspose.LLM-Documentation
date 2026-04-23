---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/
feedback: LLMNET
version: 26.5.0
title: Context parameters
description: Configure the runtime context in Aspose.LLM for .NET — context size, batch sizes, threading, RoPE and YaRN scaling, flash attention, KV cache quantization, pooling, and embeddings.
keywords:
- ContextParameters
- ContextSize
- NBatch
- NThreads
- RoPE
- YaRN
- FlashAttention
- KV cache
- TypeK
- TypeV
- embeddings
- pooling
---

`ContextParameters` mirrors `llama_context_params` in `llama.cpp`. It controls the shape of the runtime context — how many tokens the model can attend to, how batching is sized, how threads are split, how RoPE scaling stretches the context window, how flash attention is used, and how the KV cache is stored.

This is the largest parameter bag. Most fields are nullable; `null` means "use the native default" or "derive from the model". Touch these values only when you know what you are changing.

## Class reference

```csharp
namespace Aspose.LLM.Abstractions.Models;

public partial class ContextParameters
{
    // Context size and batching
    public int? ContextSize { get; set; }
    public uint? NBatch { get; set; }
    public uint? NUbatch { get; set; }
    public uint? NSeqMax { get; set; }

    // Threading
    public int? NThreads { get; set; }
    public int? NThreadsBatch { get; set; }

    // RoPE scaling
    public RopeScalingType? RopeScalingType { get; set; }
    public float? RopeFreqBase { get; set; }
    public float? RopeFreqScale { get; set; }

    // YaRN
    public float? YarnExtFactor { get; set; }
    public float? YarnAttnFactor { get; set; }
    public float? YarnBetaFast { get; set; }
    public float? YarnBetaSlow { get; set; }
    public uint? YarnOrigCtx { get; set; }

    // Attention
    public AttentionType? AttentionType { get; set; }
    public FlashAttentionType? FlashAttentionMode { get; set; }
    public bool? FlashAttention { get; set; } // legacy

    // Pooling and embeddings
    public PoolingType? PoolingType { get; set; }
    public bool? Embeddings { get; set; }

    // KV cache
    public GgmlType? TypeK { get; set; }
    public GgmlType? TypeV { get; set; }
    public bool? OffloadKqv { get; set; }
    public float? DefragThreshold { get; set; }
    public bool? SwaFull { get; set; }
    public bool? KvUnified { get; set; }

    // Other
    public bool? OpOffload { get; set; }
    public bool? NoPerf { get; set; }
}
```

## Context size and batching

### `ContextSize`

Length of the context window in tokens — the maximum number of tokens the model sees at once. Set to `null` (or `0`) to use the model's maximum from its GGUF metadata.

Built-in presets pre-set this: `Qwen25Preset` uses 32 768, `Llama32Preset` uses 131 072, `Oss20Preset` uses 131 072. See [Supported presets](/net/product-overview/supported-presets/) for each default.

Trade-off: larger context allows longer conversations and documents, but the KV cache size scales with `ContextSize × model-depth`. Going from 32K to 131K quadruples KV memory.

```csharp
preset.ContextParameters.ContextSize = 8192; // save memory for short conversations
```

### `NBatch`

Logical maximum batch size — the largest number of tokens submitted in one `llama_decode` call. Affects prompt-processing throughput.

Typical values: 512 - 4096. Larger `NBatch` speeds up prompt processing but needs more temporary memory.

Built-in presets use `NBatch` between 2 048 and 4 096 depending on model and context size.

### `NUbatch`

Physical maximum batch size — the largest chunk actually processed per kernel call. Normally `NUbatch ≤ NBatch`. `NUbatch` ≈ `NBatch` for simplicity on most deployments; different values apply only to specific multi-sequence scenarios.

### `NSeqMax`

Maximum number of distinct sequences (for recurrent models) handled in parallel. For standard transformer models, leave at `null` or `1`.

## Threading

### `NThreads`

CPU threads used during generation (token-by-token decode). When `null`, the engine falls back to `EngineParameters.DefaultThreads`.

Override when:

- Running multiple concurrent inferences per process and you want to cap each.
- Benchmarking to find the sweet spot for your model and CPU.

```csharp
preset.ContextParameters.NThreads = 8;
```

### `NThreadsBatch`

Threads used for batch (prompt) processing. Often different from `NThreads` because prompt processing parallelizes better. Typical production setting: `NThreadsBatch` ≈ `ProcessorCount` and `NThreads` ≈ half.

## RoPE scaling

RoPE (rotary position embedding) is how transformer models encode token positions. Scaling lets you use a larger context than the model was trained on, at some accuracy cost.

### `RopeScalingType`

Scaling algorithm.

| Value | Behavior |
|---|---|
| `Unspecified` (`-1`) | Use the model default (usually what you want). |
| `None` | Disable RoPE scaling. |
| `Linear` | Linear interpolation scaling. |
| `Yarn` | YaRN scaling — better quality at long contexts. |
| `LongRope` | LongRoPE scaling for very extended contexts. |

### `RopeFreqBase` and `RopeFreqScale`

Override the RoPE frequency base and scaling factor. `null` or `0` means "from model metadata". Most users leave these alone; advanced users override when extending context beyond the trained window.

## YaRN

YaRN is a specific RoPE-scaling recipe. All five fields activate only when `RopeScalingType = Yarn`.

| Field | Role | Default behavior |
|---|---|---|
| `YarnExtFactor` | Extrapolation mix factor | Negative or null = from model |
| `YarnAttnFactor` | Magnitude scaling factor | Null = from model |
| `YarnBetaFast` | Low correction dim | Null = from model |
| `YarnBetaSlow` | High correction dim | Null = from model |
| `YarnOrigCtx` | Original training context length | Null = from model |

Typical usage: the preset author (or you, for a custom preset) picks `Yarn` and sets `YarnOrigCtx` to the model's training context; other YaRN fields default from the model's metadata.

## Attention

### `AttentionType`

Causal (autoregressive) versus non-causal (bidirectional) attention.

| Value | Behavior |
|---|---|
| `Unspecified` | Model default. |
| `Causal` | Standard chat/inference attention. |
| `NonCausal` | Bidirectional; used for embedding models. |

Leave `Unspecified` unless you know why you are changing it.

### `FlashAttentionMode`

Flash attention is a fused-kernel optimization that reduces memory and speeds up long-context inference.

| Value | Behavior |
|---|---|
| `Auto` (`-1`) | Runtime picks based on support. Recommended. |
| `Disabled` | Never use flash attention. |
| `Enabled` | Force flash attention (fails on backends that do not support it). |

```csharp
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
```

Flash attention is especially effective on long contexts (32K+). For short contexts, the speed-up is small.

### `FlashAttention`

Legacy boolean flag. Prefer `FlashAttentionMode`. Kept for backwards compatibility with earlier SDK code.

## Pooling and embeddings

### `PoolingType`

Only relevant when generating embeddings (not normal chat).

| Value | Behavior |
|---|---|
| `Unspecified` / `None` | No pooling. |
| `Mean` | Average the token embeddings. |
| `Cls` | Use the CLS token's embedding. |
| `Last` | Use the last token's embedding. |
| `Rank` | Rank-based pooling (experimental). |

### `Embeddings`

When `true`, the engine extracts embeddings alongside logits. Combined with a suitable `PoolingType`, this turns the model into an embedding generator.

Embedding workflows are not covered by the chat API (`SendMessageAsync`). A dedicated use case will be added in a future release.

## KV cache

The KV cache stores the keys and values of every token the model has seen. Its size grows with context, and its precision affects both accuracy and memory.

### `TypeK` and `TypeV`

Data type for the K and V tensors in the cache. Lower precision saves memory but can degrade output quality.

Common values (full enum has 30+ entries; see the [API reference](/net/developer-reference/api-reference/) for the complete list):

| Type | Bits per value | Relative size | Notes |
|---|---:|---:|---|
| `F32` | 32 | 1.0× | Full precision. Rarely worth the memory. |
| `F16` | 16 | 0.5× | Default on many builds. Good balance. |
| `BF16` | 16 | 0.5× | Alternative to F16; slightly different range. |
| `Q8_0` | 8 | 0.25× | Very minor quality loss for substantial memory savings. |
| `Q5_1` | 5 | ~0.16× | More compact; quality drop noticeable on long contexts. |
| `Q4_0` | 4 | ~0.125× | Aggressive quantization; only for memory-tight deployments. |

Rule of thumb: **quantize V more aggressively than K** — V is less sensitive to precision.

```csharp
preset.ContextParameters.TypeK = GgmlType.F16;
preset.ContextParameters.TypeV = GgmlType.Q8_0;
```

### `OffloadKqv`

When `true`, the KQV computation (attention) and the KV cache itself live on the GPU. When `false`, they stay on CPU even if layers are offloaded. Default varies by build; usually `true` on GPU-enabled builds.

Disable only when you are fighting for a sliver of VRAM and willing to trade throughput.

### `DefragThreshold`

Threshold (fraction of holes) above which the engine defragments the KV cache. Negative = disabled (default).

Set to `0.1` - `0.5` for long-running services that churn sessions — keeps KV memory compact over time.

### `SwaFull`

Relevant for models using sliding-window attention (SWA). When `true`, stores the full SWA cache instead of the compressed form. Costs more memory, can be faster for certain workloads.

### `KvUnified`

When `true`, uses a unified buffer across input sequences for attention. Implementation detail; leave at the default.

## Other

### `OpOffload`

Offload host tensor operations to the device. Supplementary to `GpuLayers` in [`ModelInferenceParameters`](/net/developer-reference/parameters/model-inference/). Leave `null` unless you know why.

### `NoPerf`

When `true`, the engine stops collecting performance timings. A micro-optimization for high-throughput production — shaves a small amount of overhead.

## Typical recipes

### Minimize memory on a tight GPU

Short context and aggressive KV quantization:

```csharp
preset.ContextParameters.ContextSize = 4096;
preset.ContextParameters.TypeK = GgmlType.Q8_0;
preset.ContextParameters.TypeV = GgmlType.Q4_0;
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
```

### Extended context with YaRN scaling

```csharp
preset.ContextParameters.ContextSize = 131072;
preset.ContextParameters.RopeScalingType = RopeScalingType.Yarn;
preset.ContextParameters.YarnOrigCtx = 32768;   // the model's original training context
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
```

### High-throughput production

Lean on flash attention and optimized KV:

```csharp
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
preset.ContextParameters.TypeK = GgmlType.F16;
preset.ContextParameters.TypeV = GgmlType.Q8_0;
preset.ContextParameters.DefragThreshold = 0.3f;
preset.ContextParameters.NoPerf = true;
preset.ContextParameters.NThreads = Environment.ProcessorCount;
preset.ContextParameters.NThreadsBatch = Environment.ProcessorCount;
```

### Benchmark-focused (reproducible)

```csharp
preset.ContextParameters.ContextSize = 8192;
preset.ContextParameters.NBatch = 2048;
preset.ContextParameters.NUbatch = 2048;
preset.ContextParameters.NThreads = 8;
preset.ContextParameters.NThreadsBatch = 16;
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
preset.ContextParameters.NoPerf = false; // collect timings
```

### Embedding extraction

```csharp
preset.ContextParameters.Embeddings = true;
preset.ContextParameters.PoolingType = PoolingType.Mean;
preset.ContextParameters.AttentionType = AttentionType.NonCausal;
```

Chat APIs are not the right surface for embedding workflows — this recipe is preparation for a dedicated embeddings API that will be covered in a future release.

## What's next

- [Model inference parameters](/net/developer-reference/parameters/model-inference/) — GPU layers and tensor split that complement context KV settings.
- [Chat parameters](/net/developer-reference/parameters/chat/) — per-session max tokens and cache cleanup strategy.
- [Sampler parameters](/net/developer-reference/parameters/sampler/) — how the engine picks tokens within the context.

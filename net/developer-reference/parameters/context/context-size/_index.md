---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/context-size/
feedback: LLMNET
version: 26.5.0
title: ContextSize
description: Maximum number of tokens the model attends to in Aspose.LLM for .NET — governs conversation length and KV cache size.
keywords:
- ContextSize
- context window
- tokens
- KV cache
- memory
---

`ContextSize` is the maximum number of tokens the loaded model can attend to at once. It caps how long a conversation — system prompt plus all turns plus the response being generated — can be before the engine must trim via [`CacheCleanupStrategy`](/net/developer-reference/parameters/chat/).

## Quick reference

| | |
|---|---|
| **Type** | `int?` |
| **Default** | `null` (use the model's training maximum from GGUF metadata) |
| **Range** | `1` to the model's maximum; typical `4096` – `262144` |
| **Category** | Context size and batching |
| **Field on** | `ContextParameters.ContextSize` |

## What it does

The engine reserves KV cache for exactly `ContextSize` tokens at model load time. Every token of the system prompt, chat history, current user message, and generated output fits in this window. When a new token would overflow, the active [`CacheCleanupStrategy`](/net/developer-reference/parameters/chat/) trims older content.

- `null` or `0` — use the model's trained maximum as recorded in GGUF metadata.
- A positive integer up to the trained maximum — reserve exactly that many tokens of KV space.

Built-in presets set `ContextSize` to a model-appropriate default. For example:

| Preset | Default `ContextSize` |
|---:|---:|
| `Gemma3Preset` | 8 192 |
| `Phi4Preset` | 16 384 |
| `Qwen25Preset` | 32 768 |
| `Qwen3Preset` | 32 768 |
| `Llama32Preset` | 131 072 |
| `Oss20Preset` | 131 072 |
| `DeepSeekCoder2Preset` | 163 840 |

## Memory cost

KV cache scales linearly with context size. At the default KV dtype (`F16`), a 7B-parameter model typically claims ~2 MB of KV per 1 024 tokens per attention layer. For a 32-layer model at 32K context, that's ~2 GB of KV cache in addition to the model weights.

To save memory, reduce `ContextSize` or quantize the KV cache via [`TypeK`](/net/developer-reference/parameters/context/type-k/) and [`TypeV`](/net/developer-reference/parameters/context/type-v/).

## When to change it

| Scenario | Value |
|---|---|
| Short-form Q&A | `4096` – `8192` |
| Moderate chat | `16384` – `32768` |
| Long documents, extended conversations | `65536` – `131072` |
| Very long context (needs YaRN scaling) | `262144+` |
| Memory-constrained deployment | Smallest that fits your use case |

Larger `ContextSize` always costs memory; raise only when you actually use the extra window.

## Example

```csharp
var preset = new Qwen25Preset();
preset.ContextParameters.ContextSize = 8192;  // reduce from default 32768
preset.ContextParameters.TypeV = GgmlType.Q8_0; // further cut V-cache memory

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`CacheCleanupStrategy`](/net/developer-reference/parameters/chat/) — when `ContextSize` is exhausted, this strategy trims.
- [`TypeK`](/net/developer-reference/parameters/context/type-k/), [`TypeV`](/net/developer-reference/parameters/context/type-v/) — KV cache dtype multiplies context cost.
- [`FlashAttentionMode`](/net/developer-reference/parameters/context/flash-attention-mode/) — reduces memory at long contexts.
- [`RopeScalingType`](/net/developer-reference/parameters/context/rope-scaling-type/) — needed to push beyond the model's trained maximum.
- [`NBatch`](/net/developer-reference/parameters/context/n-batch/) — batch size for prompt processing.

## What's next

- [TypeK](/net/developer-reference/parameters/context/type-k/), [TypeV](/net/developer-reference/parameters/context/type-v/) — KV quantization.
- [Low memory tuning](/net/use-cases/low-memory-tuning/) — shrinking context and KV together.
- [Long context tuning](/net/use-cases/long-context-tuning/) — pushing toward 128K+.
- [Estimate memory requirements](/net/how-to/estimate-memory-requirements/) — predict KV cost.

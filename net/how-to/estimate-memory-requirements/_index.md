---
weight: 60
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/how-to/estimate-memory-requirements/
feedback: LLMNET
version: 26.5.0
title: Estimate memory requirements
description: Predict RAM and VRAM use for Aspose.LLM for .NET — model weights, KV cache, projector, intermediate buffers. Worked examples for common presets.
keywords:
- memory
- RAM
- VRAM
- sizing
- KV cache
- memory estimate
---

Four things claim memory when the SDK runs:

- Model weights.
- KV cache.
- Vision projector (vision presets only).
- Intermediate buffers and sampler state.

This how-to helps you predict the total before deployment.

## Rule-of-thumb sizes

| Component | Approximate size |
|---|---|
| Model weights | `parameters × bytes_per_parameter`. For a 7B Q4_K_M model, ~3.5 GB. |
| KV cache | `layers × heads × head_dim × context × 2 × bytes_per_kv`. Actual numbers below. |
| Vision projector | 200 MB – 2 GB. |
| Intermediate buffers | 50 MB – 500 MB. |

## Step 1. Weights from quantization

See [Understand quantization](/net/how-to/understand-quantization/) for the per-parameter bytes table.

Rough: `weights_bytes ≈ parameters × bytes_per_param`.

| Parameters | Q4_K_M | Q8_0 | F16 |
|---:|---:|---:|---:|
| 3B | ~1.8 GB | ~3.2 GB | ~6 GB |
| 7B | ~3.5 GB | ~7 GB | ~14 GB |
| 8B | ~4 GB | ~8 GB | ~16 GB |
| 20B | ~11 GB | ~21 GB | ~40 GB |
| 70B | ~35 GB | ~70 GB | ~140 GB |

## Step 2. KV cache

Depends on model architecture (number of layers, heads, head dimension), context size, and KV dtype. The underlying formula is complex; below are empirical numbers for common presets at their default contexts:

| Preset | KV at default context (F16) | KV at default context (Q8_0 V) |
|---|---|---|
| `Llama32Preset` (131K) | ~8 GB | ~5 GB |
| `Qwen25Preset` (32K) | ~2 GB | ~1.3 GB |
| `Qwen3Preset` (32K) | ~2 GB | ~1.3 GB |
| `DeepseekR1Qwen3Preset` (131K) | ~6 GB | ~4 GB |
| `Oss20Preset` (131K) | ~10 GB | ~6 GB |
| `Phi4Preset` (16K) | ~0.8 GB | ~0.5 GB |
| `Qwen3VL2BPreset` (262K) | ~12 GB | ~7 GB |

Scales roughly linearly with actual session length. A 32K-capable preset at only 4K of actual context uses ~1/8th of the listed KV.

## Step 3. Vision projector (if applicable)

| Projector quantization | Typical size |
|---|---|
| F16 | 800 MB – 2 GB |
| Q8_0 | 500 MB – 1 GB |
| Q4_K_M | 250 MB – 500 MB |

Each vision preset declares its `mmproj` file in `MmprojSourceParameters` — see [Supported presets](/net/product-overview/supported-presets/#vision-presets).

## Step 4. Add overhead

Sampler state, tokenizer, scratch buffers: 50 MB – 500 MB. Depends on batch size and context length.

For a conservative budget, add **500 MB** on top of weights + KV + projector.

## Worked examples

### `Qwen25Preset` on a 12 GB GPU

- Weights (7B Q4_K_M): 3.5 GB
- KV at 32K F16: 2 GB
- Overhead: 0.5 GB
- **Total: ~6 GB**

Comfortable fit; headroom for longer sessions or higher KV dtype.

### `Qwen3Preset` at full 32K on a 16 GB GPU

- Weights (8B Q4_K_M): 4 GB
- KV at 32K F16: 2 GB
- Overhead: 0.5 GB
- **Total: ~6.5 GB**

Fits with room for growth.

### `Oss20Preset` at full 131K on a 24 GB GPU

- Weights (20B Q4_K_M): 11 GB
- KV at 131K F16: 10 GB
- Overhead: 0.5 GB
- **Total: ~21.5 GB**

Fits a 24 GB card tightly. To leave more headroom:

- Quantize V cache: KV drops to ~6 GB. Total ~17.5 GB. Comfortable.
- Shorten context to 32K: KV drops to ~2.5 GB. Total ~14 GB.

### `Ministral3VisionPreset` on a 16 GB GPU

- Base weights (8B Q4_K_M): 4 GB
- Projector (BF16): ~2 GB
- KV at 32K F16 (shortened from 262K default): ~2 GB
- Overhead: 0.5 GB
- **Total: ~8.5 GB**

Shortening context is the easy win here.

## Shrinking memory

In order of quality impact (least to most):

1. **Shorten `ContextSize`** to what you actually use.
2. **Quantize V cache** (`TypeV = Q8_0`).
3. **Enable flash attention** — reduces KV memory at long contexts.
4. **Quantize K cache** (`TypeK = Q8_0`) — larger quality impact than V.
5. **Use a smaller preset** — last resort when the model itself is too large.

See [Low-memory tuning](/net/use-cases/low-memory-tuning/) for the full recipe.

## Measuring actual usage

After `Create` and a few messages, check real memory:

```bash
# Linux
nvidia-smi      # VRAM per GPU
top / htop      # system RAM

# Windows
# Task Manager → Performance → GPU / Memory
```

The number you read includes OS page cache of memory-mapped files — some of it is reclaimable under pressure. Still, treat the reading as a ceiling estimate.

## What's next

- [Understand quantization](/net/how-to/understand-quantization/) — precision impact on weights.
- [System requirements](/net/system-requirements/) — per-preset memory ranges.
- [Low-memory tuning](/net/use-cases/low-memory-tuning/) — when the numbers do not fit your budget.

---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/how-to/understand-quantization/
feedback: LLMNET
version: 26.5.0
title: Understand quantization
description: Primer on GGUF quantization levels — Q4, Q5, Q8, F16, BF16, IQ variants. What to pick for your memory budget and quality target.
keywords:
- quantization
- GGUF
- Q4_K_M
- Q8_0
- F16
- BF16
- IQ
- quality
- size
---

Quantization reduces the precision of model weights from the full-precision training format (usually F16 or BF16) to fewer bits per value. Smaller weights mean smaller files, less memory, and faster inference — at some cost in output quality.

## The basic trade-off

| Precision | File size (relative) | Quality loss |
|---|---:|---|
| F32 (32-bit float) | 2.0× | None (reference) |
| F16 (16-bit float) | 1.0× | Essentially none |
| BF16 (brain float 16) | 1.0× | Essentially none |
| Q8_0 (8-bit) | ~0.5× | Very small |
| Q6_K (6-bit) | ~0.38× | Small |
| Q5_K_M (5-bit medium) | ~0.33× | Small-to-moderate |
| Q4_K_M (4-bit medium) | ~0.27× | Moderate |
| Q4_0 (4-bit classic) | ~0.25× | Moderate |
| Q3_K (3-bit) | ~0.22× | Noticeable |
| Q2_K (2-bit) | ~0.18× | Large |
| IQ4_XS / IQ3_S (importance quant.) | ~0.23-0.30× | Smaller than Q equivalents at similar size |
| IQ2_XXS (very aggressive) | ~0.15× | Large |

Values are approximate; actual size depends on model architecture and specific quantizer.

## Popular picks

- **Q4_K_M** — the default for most community-uploaded GGUFs. Good balance for 7B+ models.
- **Q5_K_M** — slightly bigger, slightly better quality. Worth it when you have memory headroom.
- **Q8_0** — near-lossless. Use when you want the best quality that is not F16 and have 2× the memory.
- **IQ4_XS** — aggressive importance quantization; often better quality-per-byte than Q4_0 for the same size.
- **F16** — full precision. Useful for reproducibility or benchmarks; rarely worth the memory otherwise.

## How to pick for your preset

Built-in presets already choose a quantization. To change, override `BaseModelSourceParameters.HuggingFaceFileName` to a different file from the same repo:

```csharp
var preset = new Qwen25Preset();
// Default is Qwen2.5-7B-Instruct-Q4_K_M.gguf; switch to Q8_0:
preset.BaseModelSourceParameters.HuggingFaceFileName = "Qwen2.5-7B-Instruct-Q8_0.gguf";
```

Confirm the file exists in the repository (check the Hugging Face page).

## Memory rough estimate

For a model with N parameters:

| Quantization | Bytes per parameter | 7B model | 70B model |
|---|---:|---:|---:|
| F16 | 2.0 | ~14 GB | ~140 GB |
| Q8_0 | 1.0 | ~7 GB | ~70 GB |
| Q5_K_M | 0.625 | ~4.4 GB | ~44 GB |
| Q4_K_M | 0.5 | ~3.5 GB | ~35 GB |
| Q4_0 | 0.5 | ~3.5 GB | ~35 GB |
| IQ4_XS | 0.45 | ~3.2 GB | ~32 GB |
| Q3_K | 0.375 | ~2.6 GB | ~26 GB |

Add KV cache and intermediate buffers on top — see [Estimate memory requirements](/net/how-to/estimate-memory-requirements/).

## When to pick a smaller quantization

- Memory is the binding constraint. The model does not fit at higher quantization, but fits at lower.
- You are running many models and want to pack several into one machine.
- You accept some quality loss for speed — smaller quantizations run slightly faster due to reduced memory bandwidth pressure.

## When to avoid aggressive quantization

- Tasks sensitive to precise output (code generation, math, legal/medical reasoning).
- Long reasoning chains where errors compound.
- When you have the memory — use Q5_K_M or Q8_0 when you can.

## KV cache quantization

Separate from model weight quantization, you can quantize the KV cache at runtime via `ContextParameters.TypeK` and `TypeV`:

```csharp
preset.ContextParameters.TypeK = GgmlType.F16;
preset.ContextParameters.TypeV = GgmlType.Q8_0;
```

Saves memory on long contexts with minor quality impact. See [Context parameters](/net/developer-reference/parameters/context/) for the full enum.

## What's next

- [Model source parameters](/net/developer-reference/parameters/model-source/) — how to select a specific GGUF file.
- [Estimate memory requirements](/net/how-to/estimate-memory-requirements/) — factor quantization into your sizing.
- [Supported presets](/net/product-overview/supported-presets/) — built-in preset quantizations.

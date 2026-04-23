---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/how-to/tune-for-speed-vs-quality/
feedback: LLMNET
version: 26.5.0
title: Tune for speed vs quality
description: Move a preset along the speed-quality curve in Aspose.LLM for .NET — model size, quantization, sampler, context, GPU offload.
keywords:
- speed
- quality
- tuning
- throughput
- latency
- sampler
---

Several knobs move a preset along the speed-quality curve. This how-to summarizes them with concrete recommendations.

## Speed-biased configuration

When throughput matters most — bulk processing, real-time chat, short answers.

```csharp
var preset = new Llama32Preset(); // 3B model
preset.ContextParameters.ContextSize = 4096;
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
preset.ContextParameters.TypeV = GgmlType.Q8_0;

preset.SamplerParameters.Temperature = 0.3f;
preset.SamplerParameters.TopP = 0.9f;
preset.SamplerParameters.TopK = 20;

preset.ChatParameters.MaxTokens = 256;

preset.BaseModelInferenceParameters.GpuLayers = 999;
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
```

Expected throughput: 50-100 tokens/sec on a mid-range GPU, 15-30 on modern CPU.

## Quality-biased configuration

When the best possible output matters — deep analysis, complex reasoning, long-form writing.

```csharp
var preset = new Oss20Preset(); // 20B model
preset.ContextParameters.ContextSize = 32768;
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
preset.ContextParameters.TypeK = GgmlType.F16;
preset.ContextParameters.TypeV = GgmlType.F16;

preset.SamplerParameters.Temperature = 0.7f;
preset.SamplerParameters.TopP = 0.95f;
preset.SamplerParameters.MinP = 0.05f;
preset.SamplerParameters.RepetitionPenalty = 1.05f;

preset.ChatParameters.MaxTokens = 2048;

preset.BaseModelInferenceParameters.GpuLayers = 999;
```

Expected throughput: 10-30 tokens/sec on a high-end GPU. CPU not recommended.

## Balanced configuration

The default for most scenarios.

```csharp
var preset = new Qwen25Preset(); // 7B model
// All other settings at preset defaults.
```

Expected throughput: 30-60 tokens/sec on a mid-range GPU.

## Knobs cheat sheet

| Knob | Faster | Better quality |
|---|---|---|
| Model size | Smaller (`Phi4Preset`, `Llama32Preset`) | Larger (`Oss20Preset`, `Qwen3Preset`) |
| Quantization | Q4_K_M, Q4_0 | Q8_0, F16 |
| `ContextSize` | Shorter | Longer |
| `FlashAttentionMode` | `Enabled` | `Enabled` (both) |
| `TypeV` (KV cache) | `Q8_0`, `Q4_0` | `F16` |
| `TypeK` | `F16` | `F16` |
| `Temperature` | `0.0-0.3` (more deterministic) | `0.7-0.9` (more creative, nuanced) |
| `TopP` | `0.8-0.9` | `0.9-0.95` |
| `TopK` | `20` | `40` |
| `RepetitionPenalty` | `1.05` | `1.1` |
| `MaxTokens` | Lower | Higher |
| `GpuLayers` | `999` (full offload) | `999` (same) |

## Measure, do not guess

Throughput is hardware-specific. Benchmark on your actual target machine with realistic prompts before committing to a configuration.

```csharp
var sw = System.Diagnostics.Stopwatch.StartNew();
string reply = await api.SendMessageAsync(prompt);
sw.Stop();

Console.WriteLine($"Tokens: ~{reply.Split(' ').Length}");
Console.WriteLine($"Time: {sw.Elapsed.TotalSeconds:F2}s");
Console.WriteLine($"Rate: ~{reply.Split(' ').Length / sw.Elapsed.TotalSeconds:F1} tok/s");
```

Word count is an approximation — real tokens are usually 1.3-1.5× the word count for English.

## What's next

- [Sampler parameters](/net/developer-reference/parameters/sampler/) — fine-grained sampler control.
- [Context parameters](/net/developer-reference/parameters/context/) — context, flash attention, KV cache.
- [Understand quantization](/net/how-to/understand-quantization/) — how quantization affects throughput.

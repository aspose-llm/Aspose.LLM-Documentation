---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/n-batch/
feedback: LLMNET
version: 26.5.0
title: NBatch
description: Logical maximum batch size in Aspose.LLM for .NET — the largest number of tokens submitted in one llama_decode call; affects prompt throughput.
keywords:
- NBatch
- batch size
- prompt processing
- throughput
---

`NBatch` is the logical maximum batch size — the upper bound on the number of tokens submitted in one call to the native `llama_decode` function. Larger batch sizes speed up prompt processing at the cost of more temporary memory.

## Quick reference

| | |
|---|---|
| **Type** | `uint?` |
| **Default** | `null` (native default, typically 2048) |
| **Range** | `512` – `8192` typical; power-of-two values recommended |
| **Category** | Context size and batching |
| **Field on** | `ContextParameters.NBatch` |

## What it does

When the engine processes a prompt (system message + conversation history + new user turn), it feeds tokens to the model in batches. `NBatch` caps the largest batch sent in one call.

- Smaller `NBatch` (512) — lower memory footprint, slower prompt processing.
- Larger `NBatch` (4096, 8192) — faster prompt processing, more temporary memory.

`NBatch` affects prompt processing time, not generation throughput. Once the first output token is produced, subsequent tokens come one at a time regardless of batch size.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` (use native default) |
| Fast prompt processing, ample memory | `4096` |
| Memory-constrained | `512` or `1024` |
| Very long prompts (summarization, long context) | `4096` – `8192` |

Built-in presets set `NBatch` based on the model's needs — `Qwen25Preset` uses 3072, `Llama32Preset` uses 2048, vision presets often use 4096.

## Example

```csharp
var preset = new Qwen25Preset();
preset.ContextParameters.NBatch = 4096;  // faster prompt processing
preset.ContextParameters.NUbatch = 4096;

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`NUbatch`](/net/developer-reference/parameters/context/n-ubatch/) — physical batch size; typically set equal to or less than `NBatch`.
- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — `NBatch` should not exceed `ContextSize`.
- [`NThreadsBatch`](/net/developer-reference/parameters/context/n-threads-batch/) — threads that process the batch.

## What's next

- [NUbatch](/net/developer-reference/parameters/context/n-ubatch/) — physical batch size.
- [NThreadsBatch](/net/developer-reference/parameters/context/n-threads-batch/) — prompt-processing threads.
- [Reduce first-token latency](/net/how-to/reduce-first-token-latency/) — batch size's role in TTFT.

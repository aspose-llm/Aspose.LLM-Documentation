---
weight: 30
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/chat/max-tokens/
feedback: LLMNET
version: 26.5.0
title: MaxTokens
description: Hard cap on generated tokens per assistant response in Aspose.LLM for .NET — raise for reasoning models that emit think blocks.
keywords:
- MaxTokens
- response length
- token limit
- reasoning model
- chain of thought
---

`MaxTokens` is the upper bound on tokens the engine generates for a single assistant response. The default `2048` fits most general-purpose tasks; raise it for reasoning models (Qwen3, DeepSeek-R1) that emit hidden `<think>` blocks before the answer.

## Quick reference

| | |
|---|---|
| **Type** | `int` |
| **Default** | `2048` |
| **Range** | `> 0` |
| **Category** | Chat session |
| **Field on** | `ChatParameters.MaxTokens` |

## What it does

Once the assistant turn begins generation, the engine counts produced tokens. When the counter reaches `MaxTokens`, generation stops — even mid-sentence. The rest of the response is never produced.

- `256` — short answers; classifications; yes/no responses.
- `512` – `1024` — conversational replies, brief explanations.
- `2048` (default) — general-purpose.
- `2048` – `4096` — reasoning-model output (Qwen3, DeepSeek-R1). Leaves room for `<think>` block plus the final answer.
- `4096+` — long-form writing, essays, code generation.

{{% alert color="primary" %}}
**Reasoning model budget.** Qwen3, DeepSeek-R1, and similar chain-of-thought models emit hidden reasoning tokens (`<think>…</think>`) that consume 300-500 tokens before the actual answer. Set `MaxTokens` to **at least 1024** — ideally 2048-4096 — when using these models, or the response truncates mid-reasoning and produces no visible answer.
{{% /alert %}}

`MaxTokens` is a cap, not an allocation. Raising it does not cost memory or compute upfront; the engine generates only as many tokens as the model actually produces up to the limit.

## When to change it

| Scenario | Value |
|---|---|
| Classifications, yes/no | `128` – `256` |
| Conversational chat | `512` – `1024` |
| General-purpose (default) | `2048` |
| Reasoning models | `1024` – `4096` |
| Essays, code, long-form | `4096` – `8192` |

## Example

```csharp
var preset = new Qwen25Preset();
preset.ChatParameters.MaxTokens = 1024;
// Generous cap for conversational output.

using var api = AsposeLLMApi.Create(preset);
```

For DeepSeek-R1:

```csharp
var preset = new DeepseekR1Qwen3Preset();
preset.ChatParameters.MaxTokens = 2048; // room for <think> + answer
```

## Interactions

- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — input plus output plus history must fit; a high `MaxTokens` leaves less room for input.
- [`CacheCleanupStrategy`](/net/developer-reference/parameters/chat/cache-cleanup-strategy/) — trims history as output approaches the context cap.

## What's next

- [Chat parameters hub](/net/developer-reference/parameters/chat/) — all chat knobs.
- [Garbled output troubleshooting](/net/troubleshooting/garbled-output/) — truncation symptoms.
- [Tune for speed vs quality](/net/how-to/tune-for-speed-vs-quality/) — response length trade-offs.

---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/troubleshooting/garbled-output/
feedback: LLMNET
version: 26.5.0
title: Garbled output
description: Fix nonsense, repetitive, or truncated replies from Aspose.LLM for .NET — chat template mismatches, KV alignment, repetition loops, max-tokens cutoffs.
keywords:
- garbled output
- nonsense
- repetition
- template mismatch
- truncated
- hallucination
---

The model loaded successfully, but its replies are nonsensical, repetitive, or obviously broken. This is almost always a configuration issue rather than a model issue.

## Symptom

- Replies contain literal marker tokens like `<|im_start|>`, `<image>`, `<|endoftext|>`.
- The model produces the same phrase or token repeatedly.
- Output is truncated mid-sentence after exactly N tokens.
- Output is coherent at first, then devolves into nonsense after a few hundred tokens.
- Vision replies describe something different from the actual image.

## Cause

- **Chat template mismatch** — the engine picked the wrong template for the model.
- **Repetition penalty too low** (or zero) — the model loops.
- **`MaxTokens` too low** for a reasoning model — truncation mid-reasoning.
- **KV cache cleanup dropped important context** — middle of a long session.
- **Wrong preset for the model** — a custom GGUF paired with a preset that does not match its architecture.
- **Aggressive KV quantization** — on long contexts, Q4 K/V can degrade quality.

## Resolution

### 1. Literal marker tokens in the output

**Diagnosis**: template mismatch. The engine fell back to a generic template; the model's actual markers appear as text.

Enable debug logging and look for `[MM] selected template: ...`:

```csharp
preset.EngineParameters.EnableDebugLogging = true;
```

If the template is `fallback` or does not match your model family, the model needs a supported template. Options:

- Verify you are using the correct built-in preset (see [Supported presets](/net/product-overview/supported-presets/)).
- For a custom GGUF, try a different export from the same model with richer metadata.
- For vision: see [Chat templates](/net/developer-reference/multimodal/chat-templates/).

### 2. Repetition loops

**Diagnosis**: the model generates the same phrase in a loop.

Raise repetition penalty:

```csharp
preset.SamplerParameters.RepetitionPenalty = 1.15f; // default 1.1
```

If that still loops, try DRY (Don't Repeat Yourself):

```csharp
preset.SamplerParameters.DryMultiplier = 0.8f;
preset.SamplerParameters.DryBase = 1.75f;
preset.SamplerParameters.DryAllowedLength = 3;
```

See [Sampler parameters](/net/developer-reference/parameters/sampler/) for the full repetition knob set.

### 3. Truncated output (cuts off mid-sentence)

**Diagnosis**: `MaxTokens` limit hit before the model finished.

Raise the budget:

```csharp
preset.ChatParameters.MaxTokens = 2048; // from default 2048; raise further for long answers
```

For reasoning models (Qwen3, DeepSeek-R1) that emit `<think>` blocks, budget 1024-2048 for the thinking plus the answer.

### 4. Coherent start, garbled after a while

**Diagnosis**: KV cache cleanup evicted critical context mid-session, or KV quantization is too aggressive at long contexts.

Change the cleanup policy:

```csharp
preset.ChatParameters.CacheCleanupStrategy =
    CacheCleanupStrategy.KeepSystemPromptAndFirstUserMessage;
```

Or upgrade KV precision:

```csharp
preset.ContextParameters.TypeK = GgmlType.F16;
preset.ContextParameters.TypeV = GgmlType.F16;
```

F16 is the safe default; drop to Q8_0 only with memory pressure.

### 5. Model produces the wrong answer for clear questions

**Diagnosis**: custom GGUF paired with a wrong preset, or the preset's sampler settings do not match the model's training distribution.

- Use a built-in preset for the model family, or set up a [custom preset](/net/use-cases/custom-preset/) from scratch rather than reusing an unrelated built-in.
- Start with conservative sampler settings: `Temperature = 0.3`, `TopP = 0.9`.
- Test with the reference prompt from the model's Hugging Face page to rule out sampling issues.

### 6. Vision: reply describes the wrong thing

**Diagnosis**: the image was not delivered correctly to the model.

- Enable `MtmdContextParameters.PrintTimings = true` to verify the image was processed.
- Enable debug logging and look for `[MM]` lines — confirm image chunks are tokenized.
- See [Debugging vision](/net/developer-reference/multimodal/debugging-vision/).

## Prevention

- Stick to built-in presets when possible.
- Test new models with simple prompts first; check the output matches the model's reference outputs.
- Run with debug logging in staging to catch template fallbacks before production.

## What's next

- [Sampler parameters](/net/developer-reference/parameters/sampler/) — repetition penalties, DRY.
- [Chat parameters](/net/developer-reference/parameters/chat/) — `MaxTokens`, `CacheCleanupStrategy`.
- [Chat templates](/net/developer-reference/multimodal/chat-templates/) — vision template selection.
- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — multimodal-specific diagnosis.

---
weight: 170
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/context/flash-attention/
feedback: LLMNET
version: 26.5.0
title: FlashAttention (legacy)
description: Legacy boolean toggle for flash attention in Aspose.LLM for .NET — prefer FlashAttentionMode enum for new code.
keywords:
- FlashAttention
- legacy
- boolean toggle
- flash attention
---

`FlashAttention` is the legacy boolean toggle for flash attention. It predates the more granular [`FlashAttentionMode`](/net/developer-reference/parameters/context/flash-attention-mode/) enum. Prefer `FlashAttentionMode` for new code.

## Quick reference

| | |
|---|---|
| **Type** | `bool?` |
| **Default** | `null` (use model default) |
| **Category** | Attention (legacy) |
| **Field on** | `ContextParameters.FlashAttention` |

## What it does

- `null` — no explicit override; runtime / model default applies.
- `true` — request flash attention (equivalent to `FlashAttentionMode = Enabled`).
- `false` — disable flash attention (equivalent to `FlashAttentionMode = Disabled`).

`FlashAttentionMode` supersedes this field. When both are set, consult SDK behavior — to avoid ambiguity, set only one.

## When to change it

| Scenario | Value |
|---|---|
| Default — prefer `FlashAttentionMode` instead | `null` |
| Legacy code using this field | Keep for backwards compatibility |

For new code, use [`FlashAttentionMode`](/net/developer-reference/parameters/context/flash-attention-mode/) which offers the three-way `Auto` / `Disabled` / `Enabled` choice.

## Example

```csharp
// Legacy style (kept for compatibility):
preset.ContextParameters.FlashAttention = true;

// Preferred (modern):
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
```

## Interactions

- [`FlashAttentionMode`](/net/developer-reference/parameters/context/flash-attention-mode/) — newer enum replacement.

## What's next

- [FlashAttentionMode](/net/developer-reference/parameters/context/flash-attention-mode/) — recommended replacement.
- [Context parameters hub](/net/developer-reference/parameters/context/) — all context knobs.

---
weight: 50
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/multimodal-context/media-marker/
feedback: LLMNET
version: 26.5.0
title: MediaMarker
description: Custom media marker token in Aspose.LLM for .NET vision chat templates — null uses the model-specific default; override only with full knowledge of the format.
keywords:
- MediaMarker
- chat template
- marker token
- image marker
- vision
---

`MediaMarker` overrides the placeholder token used in the vision chat template to mark where images are inserted. Default is model-specific and selected automatically — override only when you understand the exact format your model expects.

## Quick reference

| | |
|---|---|
| **Type** | `string?` |
| **Default** | `null` (use the chat template's default marker) |
| **Category** | Multimodal context |
| **Field on** | `MultimodalContextParameters.MediaMarker` |

## What it does

Vision chat templates use a specific token or placeholder in the prompt to mark where the image embedding is inserted. Different model families use different markers — LLaVA uses one, Qwen-VL uses another, Gemma3 yet another. The SDK picks the correct one from the model's metadata.

- `null` (default) — template picks the model's default.
- Explicit string — override with your own marker.

Overriding without matching the model's trained format produces garbled output — the image is inserted at the wrong position or with the wrong surrounding tokens.

## When to change it

| Scenario | Value |
|---|---|
| Default (recommended) | `null` |
| Custom GGUF with a non-standard marker | Exact marker the model was trained with |

Very rarely needed. Most use cases leave this `null`.

## Example

```csharp
var preset = new Qwen3VL2BPreset();
// preset.MtmdContextParameters.MediaMarker = null; // default — correct choice
```

Experimental custom marker:

```csharp
preset.MtmdContextParameters.MediaMarker = "<|image|>";
// Only if you know the model's training format expects this exact marker.
```

## Interactions

- [Chat templates](/net/developer-reference/multimodal/chat-templates/) — which template is selected automatically.
- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — diagnose marker-related misalignment.

## What's next

- [Chat templates](/net/developer-reference/multimodal/chat-templates/) — supported template list.
- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — marker-related troubleshooting.
- [Multimodal context hub](/net/developer-reference/parameters/multimodal-context/) — all mtmd knobs.

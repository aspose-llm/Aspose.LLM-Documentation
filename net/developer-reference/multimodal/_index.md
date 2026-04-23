---
weight: 45
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/multimodal/
feedback: LLMNET
version: 26.5.0
title: Multimodal
description: Multimodal (vision) support in Aspose.LLM for .NET — attach images to chat messages, pick a vision preset, understand chat templates, and debug vision flows.
keywords:
- multimodal
- vision
- image
- mmproj
- MediaAttachment
- VLM
- vision-language model
---

Aspose.LLM for .NET supports image input alongside text through `mtmd` — the `llama.cpp` multimodal layer — and a small set of built-in vision presets. This section covers everything you need to work with images: picking a preset, attaching images to messages, understanding chat templates, and diagnosing common problems.

The SDK does **not** support audio input in the current release, even though the underlying `mtmd` layer can handle audio chunks.

## Sections

- [Vision presets](/net/developer-reference/multimodal/vision-presets/) — built-in presets with their model sources, projector sources, and picker guidance.
- [Attaching images](/net/developer-reference/multimodal/attaching-images/) — `MediaAttachment`, supported formats (JPEG, PNG, BMP, GIF, WebP), the 50 MB limit, and how to pass images to `SendMessageAsync`.
- [Chat templates](/net/developer-reference/multimodal/chat-templates/) — the eight vision chat templates the SDK recognizes and how auto-selection works.
- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — tagged logs, `parse_mm_logs.zsh`, and common misalignments.

## At a glance

Minimum vision flow:

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25VL3BPreset();   // or any vision preset
using var api = AsposeLLMApi.Create(preset);

byte[] imageBytes = File.ReadAllBytes("photo.jpg");

string reply = await api.SendMessageAsync(
    "Describe this image in one short sentence.",
    media: new[] { imageBytes });

Console.WriteLine(reply);
```

`media` is `IEnumerable<byte[]>` — you can pass one or several images per message.

## What's next

- [Vision presets](/net/developer-reference/multimodal/vision-presets/) — pick the right built-in preset.
- [Attaching images](/net/developer-reference/multimodal/attaching-images/) — formats and limits.
- [Supported presets](/net/product-overview/supported-presets/#vision-presets) — quick catalog.

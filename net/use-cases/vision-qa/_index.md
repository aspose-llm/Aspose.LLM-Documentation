---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/vision-qa/
feedback: LLMNET
version: 26.5.0
title: Vision question answering
description: Ask questions about images in Aspose.LLM for .NET — single-image Q&A, multi-image comparison, and document transcription with vision presets.
keywords:
- vision
- Q&A
- image
- VLM
- Qwen VL
- Gemma Vision
- OCR
---

Vision presets let you ask questions about images. Pass image bytes alongside the text prompt, and the model's multimodal projector feeds the image through the same generation loop as text.

## When to use this pattern

- A user uploads a photo and asks a question about it.
- You extract text from a scanned document or screenshot.
- You compare two images side by side.
- You analyze diagrams, charts, or UI mockups.

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).
- A [vision preset](/net/developer-reference/multimodal/vision-presets/) — `Qwen3VL2BPreset`, `Qwen25VL3BPreset`, `Gemma3VisionPreset`, or `Ministral3VisionPreset`.

## Minimal example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen3VL2BPreset();
using var api = AsposeLLMApi.Create(preset);

byte[] imageBytes = File.ReadAllBytes("cat.jpg");

string reply = await api.SendMessageAsync(
    "Describe what you see in one short sentence.",
    media: new[] { imageBytes });

Console.WriteLine(reply);
// Output: A gray tabby cat sits on a wooden floor near a window.
```

Supported image formats: JPEG, PNG, BMP, GIF, WebP. Per-attachment size limit: 50 MB. See [Attaching images](/net/developer-reference/multimodal/attaching-images/).

## Multi-image comparison

Most vision presets handle one or two images per message. Pass multiple byte arrays in the `media` parameter:

```csharp
byte[] before = File.ReadAllBytes("ui-before.png");
byte[] after = File.ReadAllBytes("ui-after.png");

string reply = await api.SendMessageAsync(
    "Compare these two UI screenshots. List the visible changes.",
    media: new[] { before, after });

Console.WriteLine(reply);
```

The engine preserves the image order in the prompt; each image sits at its marker position in the chat template.

## Document transcription (OCR-style)

Text-heavy images — scanned pages, PDFs rendered to images, screenshots of documents — work best with vision presets that have strong OCR fine-tunes.

```csharp
var preset = new Gemma3VisionPreset(); // fine-tuned for Latex and structured text
using var api = AsposeLLMApi.Create(preset);

byte[] scan = File.ReadAllBytes("invoice-scan.png");

string transcript = await api.SendMessageAsync(
    "Transcribe all visible text in this image verbatim. Preserve line breaks.",
    media: new[] { scan });

Console.WriteLine(transcript);
```

For complex documents (tables, multi-column layouts), ask the model to describe structure explicitly: "Transcribe the table, separating columns with `|`."

## Multi-turn vision

Sessions work with vision, too. The first message can include the image; follow-ups refer back to it.

```csharp
string sessionId = await api.StartNewChatAsync();

await api.SendMessageToSessionAsync(sessionId,
    "Describe this diagram.",
    media: new[] { File.ReadAllBytes("architecture.png") });

string q1 = await api.SendMessageToSessionAsync(sessionId,
    "Which component handles authentication?");

string q2 = await api.SendMessageToSessionAsync(sessionId,
    "How does data flow from the API Gateway to the database?");

Console.WriteLine(q1);
Console.WriteLine(q2);
```

## Full runnable example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

internal class VisionQaDemo
{
    public static async Task Main(string[] args)
    {
        var license = new Aspose.LLM.License();
        license.SetLicense("Aspose.LLM.lic");

        var preset = new Qwen3VL2BPreset();
        preset.ChatParameters.SystemPrompt =
            "You are a concise visual analyst. Answer in one or two sentences.";

        using var api = AsposeLLMApi.Create(preset);

        if (args.Length < 2)
        {
            Console.WriteLine("Usage: vision-qa <image-path> <question>");
            return;
        }

        string imagePath = args[0];
        string question = args[1];

        byte[] imageBytes;
        try
        {
            imageBytes = File.ReadAllBytes(imagePath);
        }
        catch (FileNotFoundException)
        {
            Console.WriteLine($"Image not found: {imagePath}");
            return;
        }

        try
        {
            string reply = await api.SendMessageAsync(question, media: new[] { imageBytes });
            Console.WriteLine($"Q: {question}");
            Console.WriteLine($"A: {reply}");
        }
        catch (InvalidOperationException ex) when (ex.Message.Contains("Image"))
        {
            Console.WriteLine($"Image error: {ex.Message}");
        }
    }
}
```

Run:

```bash
dotnet run -- cat.jpg "What breed of cat is this?"
```

## Common errors

- **`Unknown or unsupported image format`** — convert the image to JPEG/PNG/BMP/GIF/WebP.
- **`Image size exceeds maximum allowed (50MB)`** — downscale; the projector resizes to ~336-448 pixels anyway.
- **Literal `<image>` markers in the reply** — chat-template mismatch. See [Debugging vision](/net/developer-reference/multimodal/debugging-vision/).
- **Model ignores the image** — make sure you are using a vision preset and the prompt explicitly refers to "the image".

## What's next

- [Vision presets](/net/developer-reference/multimodal/vision-presets/) — picking the right preset.
- [Attaching images](/net/developer-reference/multimodal/attaching-images/) — format details and `MediaAttachment`.
- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — diagnose vision-specific issues.

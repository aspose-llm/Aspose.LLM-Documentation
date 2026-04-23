---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/multimodal/attaching-images/
feedback: LLMNET
version: 26.5.0
title: Attaching images
description: Pass images to Aspose.LLM for .NET chat methods — MediaAttachment class, supported formats (JPEG, PNG, BMP, GIF, WebP), 50 MB limit, magic-byte detection, and multiple images per message.
keywords:
- image input
- MediaAttachment
- MediaFormat
- JPEG
- PNG
- WebP
- vision
- multimodal
---

The `media` parameter of `SendMessageAsync` and `SendMessageToSessionAsync` accepts image byte arrays. The engine detects each image's format from its magic bytes, validates size, and passes it through the vision projector alongside the text prompt.

## Quick example

```csharp
byte[] imageBytes = File.ReadAllBytes("cat.jpg");

string reply = await api.SendMessageAsync(
    "Describe the animal in this image.",
    media: new[] { imageBytes });
```

The parameter is `IEnumerable<byte[]>?`. Pass `null` or omit it entirely for a text-only message.

## Supported formats

| Format | Magic bytes (hex) |
|---|---|
| JPEG | `FF D8` |
| PNG | `89 50 4E 47` |
| BMP | `42 4D` |
| GIF | `47 49 46` |
| WebP | `52 49 46 46 .. .. .. .. 57 45 42 50` |

Formats outside this list are rejected. If your source is TIFF, HEIC, SVG, or another format, convert to one of the supported formats before passing to the API.

## 50 MB per-attachment limit

Each image must be 50 MB or smaller. Larger images throw:

```
System.InvalidOperationException: Image size exceeds maximum allowed (50MB).
```

In practice, downsize images to the projector's native resolution (typically 336 or 448 pixels on the short side) before attaching — large images are resized internally and eat wall time.

## Multiple images per message

Pass several byte arrays in one call:

```csharp
byte[] img1 = File.ReadAllBytes("before.png");
byte[] img2 = File.ReadAllBytes("after.png");

string reply = await api.SendMessageAsync(
    "Compare these two screenshots. What changed?",
    media: new[] { img1, img2 });
```

The engine processes images in the order they appear in the array. The chat template places them at marker positions in the prompt.

Not every vision model handles arbitrary numbers of images equally well — most are tuned for one or two. Refer to the model's own documentation for recommended limits.

## `MediaAttachment` class

When `SendMessageAsync` receives `byte[]`, it wraps each array in a `MediaAttachment` internally. The class is also public if you need to handle attachments explicitly — for example, when adding them to `ChatMessage` instances manually.

```csharp
namespace Aspose.LLM.Abstractions.Models;

public class MediaAttachment
{
    public byte[] Data { get; set; }
    public MediaFormat Format { get; set; }
    public string MimeType { get; set; }
    public int Width { get; set; }
    public int Height { get; set; }
    public string? Description { get; set; }
    public string? FileName { get; set; }

    public MediaAttachment();
    public MediaAttachment(byte[] data, MediaFormat format);

    public static MediaAttachment FromBytes(
        byte[] data,
        string? fileName = null,
        string? description = null);

    public void Validate();
}
```

### `FromBytes` — auto-detecting format

The idiomatic constructor:

```csharp
using Aspose.LLM.Abstractions.Models;

var attachment = MediaAttachment.FromBytes(
    File.ReadAllBytes("scan.png"),
    fileName: "scan.png",
    description: "Invoice scanned at 300 DPI");
```

The factory:

1. Reads the first 12 bytes.
2. Matches against the magic-byte table above.
3. Sets `Format`, `MimeType`, `FileName`, and `Description` accordingly.
4. Returns the attachment.

If the magic bytes do not match, `Format` is `MediaFormat.Unknown` and `Validate` throws when called.

### `Validate`

Enforces three checks:

| Check | Failure |
|---|---|
| Data is non-null and non-empty | `InvalidOperationException: Image data cannot be null or empty.` |
| Data size ≤ 50 MB | `InvalidOperationException: Image size exceeds maximum allowed (50MB).` |
| Format != Unknown | `InvalidOperationException: Unknown or unsupported image format.` |

The SDK calls `Validate` automatically when you send a message. You can call it earlier to fail-fast in application code.

### `MediaFormat` enum

```csharp
public enum MediaFormat
{
    Unknown = 0,
    JPEG,
    PNG,
    BMP,
    GIF,
    WebP,
}
```

## `ChatMessage` with media

For scenarios that construct the chat history manually (for example, seeding `ChatParameters.History` with a multimodal turn), build `ChatMessage` objects directly:

```csharp
using Aspose.LLM.Abstractions.Models;

var attachment = MediaAttachment.FromBytes(File.ReadAllBytes("label.png"));
var msg = ChatMessage.CreateUserMessage("Read this label.", attachment);

// Attach to preset history:
preset.ChatParameters.History = new List<ChatMessage> { msg };
```

`ChatMessage.HasMedia` and `ChatMessage.TotalMediaSize` are convenience properties for inspection.

## Common errors

| Symptom | Cause | Fix |
|---|---|---|
| `Unknown or unsupported image format` | File is TIFF, HEIC, SVG, or corrupt. | Convert to JPEG/PNG/BMP/GIF/WebP. |
| `Image size exceeds maximum allowed (50MB)` | Very high-resolution image. | Downscale or recompress before attaching. |
| Garbled reply, no image detail | Chat template mismatch or wrong preset. | Verify you are using a vision preset; see [Chat templates](/net/developer-reference/multimodal/chat-templates/). |
| Reply ignores the image | Prompt is too generic. | Reference "the image", "this diagram", etc. explicitly. |

## What's next

- [Vision presets](/net/developer-reference/multimodal/vision-presets/) — pick the right preset for your images.
- [Chat templates](/net/developer-reference/multimodal/chat-templates/) — how the engine inserts image markers.
- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — diagnose misalignments and garbled output.

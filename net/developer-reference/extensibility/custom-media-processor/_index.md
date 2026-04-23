---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/extensibility/custom-media-processor/
feedback: LLMNET
version: 26.5.0
title: Custom media processor
description: Replace IMediaProcessor in Aspose.LLM for .NET to customize image preprocessing — format handling, validation, color conversion, and ProcessedMedia output.
keywords:
- IMediaProcessor
- media processing
- image preprocessing
- vision
- extensibility
- DICOM
- color conversion
---

`IMediaProcessor` controls how raw image bytes become a `ProcessedMedia` structure the vision pipeline can evaluate. Implementing a custom processor lets you handle formats the default processor does not support, apply specific preprocessing (color spaces, DICOM extraction, sensor calibration), or add validation beyond size and format.

For everyday vision use, the default `MediaManager` implementation is enough — it handles JPEG, PNG, BMP, GIF, WebP with magic-byte detection and 50 MB size limits. Consider custom only when you have specific needs the default does not cover.

## Interface reference

```csharp
namespace Aspose.LLM.Core.Services;

public interface IMediaProcessor
{
    Task<ProcessedMedia> ProcessMediaAsync(
        byte[] mediaData,
        MediaProcessingOptions? options = null);

    bool IsSupportedFormat(byte[] mediaData);

    MediaFormat DetectFormat(byte[] mediaData);

    bool ValidateMedia(byte[] mediaData, out string? errorMessage);

    ProcessedMedia CreateProcessedMedia(
        byte[] mediaData,
        MediaFormat? knownFormat = null);
}
```

| Method | Purpose |
|---|---|
| `ProcessMediaAsync` | Main entry point. Runs format detection, validation, and any preprocessing. Returns `ProcessedMedia`. |
| `IsSupportedFormat` | Lightweight check without full processing. |
| `DetectFormat` | Magic-byte detection; returns a `MediaFormat` enum value. |
| `ValidateMedia` | Returns `true` when input is acceptable; populates `errorMessage` on failure. |
| `CreateProcessedMedia` | Synchronous variant for cases where format is already known. |

`IMediaProcessor` lives in `Aspose.LLM.Core.Services`. It is public, but in the Core assembly rather than Abstractions — usable after ILRepack merges, but not part of the minimal `Aspose.LLM.Abstractions` API surface.

## `ProcessedMedia` structure

```csharp
public class ProcessedMedia
{
    public byte[] ProcessedData { get; set; } = Array.Empty<byte>();
    public int Width { get; set; }
    public int Height { get; set; }
    // ... additional fields for embeddings, metadata, etc.
}
```

Full field list varies by SDK version — reference the API documentation at `reference.aspose.com/llm/net/` for the current shape.

## Example — support a custom format

The default processor rejects anything outside JPEG/PNG/BMP/GIF/WebP. If your source is TIFF, DICOM, or raw pixel buffers, you convert early in a custom processor:

```csharp
using Aspose.LLM.Abstractions.Models;
using Aspose.LLM.Core.Services;

public class TiffAwareProcessor : IMediaProcessor
{
    private readonly IMediaProcessor _inner;

    public TiffAwareProcessor(IMediaProcessor inner) => _inner = inner;

    public async Task<ProcessedMedia> ProcessMediaAsync(
        byte[] mediaData,
        MediaProcessingOptions? options = null)
    {
        // Intercept TIFF (magic bytes 0x49 0x49 0x2A 0x00 for little-endian,
        // 0x4D 0x4D 0x00 0x2A for big-endian).
        if (IsTiff(mediaData))
        {
            byte[] pngBytes = ConvertTiffToPng(mediaData); // use System.Drawing or ImageSharp
            return await _inner.ProcessMediaAsync(pngBytes, options);
        }

        return await _inner.ProcessMediaAsync(mediaData, options);
    }

    public bool IsSupportedFormat(byte[] mediaData)
        => IsTiff(mediaData) || _inner.IsSupportedFormat(mediaData);

    public MediaFormat DetectFormat(byte[] mediaData)
        => _inner.DetectFormat(mediaData); // TIFF still shows as Unknown until conversion

    public bool ValidateMedia(byte[] mediaData, out string? errorMessage)
    {
        if (IsTiff(mediaData))
        {
            errorMessage = null;
            return true;
        }
        return _inner.ValidateMedia(mediaData, out errorMessage);
    }

    public ProcessedMedia CreateProcessedMedia(byte[] mediaData, MediaFormat? knownFormat = null)
        => _inner.CreateProcessedMedia(mediaData, knownFormat);

    private static bool IsTiff(byte[] data) =>
        data.Length >= 4 &&
        ((data[0] == 0x49 && data[1] == 0x49 && data[2] == 0x2A && data[3] == 0x00)
         || (data[0] == 0x4D && data[1] == 0x4D && data[2] == 0x00 && data[3] == 0x2A));

    private static byte[] ConvertTiffToPng(byte[] tiff) { /* ... */ throw new NotImplementedException(); }
}
```

## Example — stricter validation

Add size or content-based validation on top of the defaults:

```csharp
public class StricterMediaProcessor : IMediaProcessor
{
    private readonly IMediaProcessor _inner;
    private const int MaxDimension = 4096;

    public StricterMediaProcessor(IMediaProcessor inner) => _inner = inner;

    public Task<ProcessedMedia> ProcessMediaAsync(byte[] mediaData, MediaProcessingOptions? options = null)
        => _inner.ProcessMediaAsync(mediaData, options);

    public bool ValidateMedia(byte[] mediaData, out string? errorMessage)
    {
        if (!_inner.ValidateMedia(mediaData, out errorMessage))
            return false;

        // Extract dimensions via a fast image-header reader (ImageSharp, custom code).
        var (w, h) = ReadDimensions(mediaData);
        if (w > MaxDimension || h > MaxDimension)
        {
            errorMessage = $"Image dimensions exceed {MaxDimension}x{MaxDimension}.";
            return false;
        }

        return true;
    }

    public bool IsSupportedFormat(byte[] mediaData) => _inner.IsSupportedFormat(mediaData);
    public MediaFormat DetectFormat(byte[] mediaData) => _inner.DetectFormat(mediaData);
    public ProcessedMedia CreateProcessedMedia(byte[] mediaData, MediaFormat? knownFormat = null)
        => _inner.CreateProcessedMedia(mediaData, knownFormat);

    private static (int Width, int Height) ReadDimensions(byte[] data) { /* ... */ throw new NotImplementedException(); }
}
```

## Registration

```csharp
using Microsoft.Extensions.DependencyInjection;
using Aspose.LLM.Core.Services;
using Aspose.LLM.Core.DependencyInjection;

services.AddLlamaServices(new Qwen3VL2BPreset());
services.AddSingleton<IMediaProcessor, StricterMediaProcessor>();
```

The default `MediaManager` is wired into `Engine` as the `IMediaProcessor` implementation. Substituting via DI should replace it, but verify on your specific SDK version — the wiring can evolve between releases.

## What's next

- [Attaching images](/net/developer-reference/multimodal/attaching-images/) — the built-in format support `MediaProcessor` applies.
- [Multimodal context parameters](/net/developer-reference/parameters/multimodal-context/) — projector-side knobs.
- [Extensibility overview](/net/developer-reference/extensibility/) — other substitution points.

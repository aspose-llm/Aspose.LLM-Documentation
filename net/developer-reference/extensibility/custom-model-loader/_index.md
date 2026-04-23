---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/extensibility/custom-model-loader/
feedback: LLMNET
version: 26.5.0
title: Custom model loader
description: Replace the default IModelLoader implementation in Aspose.LLM for .NET to control the full model-loading pipeline.
keywords:
- IModelLoader
- custom loader
- extensibility
- model loading
---

`IModelLoader` is the contract the engine uses to turn `ModelSourceParameters` into an `ILlamaModel`. Substituting this interface gives you full control over the model-loading pipeline — file resolution, native `llama.cpp` model creation, inference parameter application, and any diagnostics you want to inject around it.

This is the broadest extensibility point. Prefer [`IModelFileProvider`](/net/developer-reference/extensibility/custom-file-provider/) if you only need to change **where** the model file comes from; `IModelLoader` is for changing **how** it is loaded.

## Interface reference

```csharp
namespace Aspose.LLM.Abstractions.Interfaces;

public interface IModelLoader
{
    Task<ILlamaModel> LoadModelAsync(
        ModelSourceParameters modelParameters,
        ModelInferenceParameters inferenceParameters,
        IProgress<double>? progress = null,
        CancellationToken cancellationToken = default);
}
```

The method:

- Takes `ModelSourceParameters` (where to find the model) and `ModelInferenceParameters` (how to load it — `GpuLayers`, `SplitMode`, etc.).
- Reports progress via `IProgress<double>` (0.0 to 1.0) during long operations.
- Returns an `ILlamaModel` — the loaded model ready for inference.
- Throws `ArgumentNullException` on null `modelParameters`.
- Throws `InvalidOperationException` when the model cannot be loaded.

## Implementation skeleton

```csharp
using Aspose.LLM.Abstractions.Interfaces;
using Aspose.LLM.Abstractions.Parameters;

public class MyModelLoader : IModelLoader
{
    public async Task<ILlamaModel> LoadModelAsync(
        ModelSourceParameters modelParameters,
        ModelInferenceParameters inferenceParameters,
        IProgress<double>? progress = null,
        CancellationToken cancellationToken = default)
    {
        if (modelParameters is null)
            throw new ArgumentNullException(nameof(modelParameters));

        // 1. Resolve the model file path (use IModelFileProvider or your own logic).
        progress?.Report(0.1);
        string modelFilePath = await ResolveModelFileAsync(modelParameters, cancellationToken);

        // 2. Instrument or validate as needed.
        progress?.Report(0.5);
        ValidateModelFile(modelFilePath);

        // 3. Delegate to the SDK's native loading path, or call llama.cpp yourself.
        progress?.Report(0.9);
        ILlamaModel model = await LoadIntoNativeMemoryAsync(
            modelFilePath,
            inferenceParameters,
            cancellationToken);

        progress?.Report(1.0);
        return model;
    }

    // ... helper methods
}
```

Implementing `LoadIntoNativeMemoryAsync` from scratch means wrapping the SDK's P/Invoke layer (`Aspose.LLM.Interop`). That is advanced work — you are essentially recreating `ModelManager` with custom behavior. In most cases, the simpler path is:

1. Use the default model loading via `ModelManager`.
2. Wrap it in a decorator that adds instrumentation, caching, or retries.

## Decorator pattern

```csharp
using Microsoft.Extensions.Logging;
using Aspose.LLM.Abstractions.Interfaces;
using Aspose.LLM.Abstractions.Parameters;

public class LoggingModelLoader : IModelLoader
{
    private readonly IModelLoader _inner;
    private readonly ILogger<LoggingModelLoader> _logger;

    public LoggingModelLoader(IModelLoader inner, ILogger<LoggingModelLoader> logger)
    {
        _inner = inner;
        _logger = logger;
    }

    public async Task<ILlamaModel> LoadModelAsync(
        ModelSourceParameters modelParameters,
        ModelInferenceParameters inferenceParameters,
        IProgress<double>? progress = null,
        CancellationToken cancellationToken = default)
    {
        _logger.LogInformation("Loading model from {Source}",
            modelParameters.ModelFilePath ?? modelParameters.HuggingFaceRepoId);

        var sw = System.Diagnostics.Stopwatch.StartNew();
        try
        {
            var model = await _inner.LoadModelAsync(
                modelParameters, inferenceParameters, progress, cancellationToken);
            _logger.LogInformation("Model loaded in {Elapsed}", sw.Elapsed);
            return model;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Model load failed after {Elapsed}", sw.Elapsed);
            throw;
        }
    }
}
```

## Registration

Register your implementation after `AddLlamaServices`:

```csharp
services.AddLlamaServices(new Qwen25Preset());
services.AddSingleton<IModelLoader, MyModelLoader>();
```

If you want to decorate the default loader, keep the default registration and wrap it — but the default `ModelManager` is registered as a concrete type (`ModelManager`), not as `IModelLoader`. You may need to adapt your decorator accordingly, or contact [Aspose support](https://forum.aspose.com/) for guidance on the current wiring.

## What's next

- [Custom file provider](/net/developer-reference/extensibility/custom-file-provider/) — narrower surface, often what you actually need.
- [Extensibility overview](/net/developer-reference/extensibility/) — when to use which interface.
- [Dependency injection](/net/developer-reference/dependency-injection/) — how `AddLlamaServices` wires services.

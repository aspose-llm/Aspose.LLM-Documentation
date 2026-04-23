---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/extensibility/custom-file-provider/
feedback: LLMNET
version: 26.5.0
title: Custom file provider
description: Replace IModelFileProvider in Aspose.LLM for .NET to source model files from custom locations — S3, internal registries, encrypted storage, or air-gapped networks.
keywords:
- IModelFileProvider
- file provider
- S3
- custom registry
- offline
- air-gapped
- extensibility
---

`IModelFileProvider` abstracts **file resolution** — the step that turns a `ModelSourceParameters` into a local file path. The default SDK ships two providers: `LocalFilesystemProvider` (for `ModelFilePath`) and `HuggingFaceProvider` (for `HuggingFaceRepoId` + `HuggingFaceFileName`).

Implement a custom provider when you need to source models from a location the defaults do not cover — private S3 buckets, artifact repositories (Artifactory, Nexus), encrypted storage, or signed internal model catalogs.

This is a narrower surface than [`IModelLoader`](/net/developer-reference/extensibility/custom-model-loader/) and usually what you actually need.

## Interface reference

```csharp
namespace Aspose.LLM.Abstractions.Interfaces;

public interface IModelFileProvider
{
    Task<string> GetModelFileAsync(
        ModelSourceParameters modelParameters,
        IProgress<double>? progress = null,
        CancellationToken cancellationToken = default);
}
```

The method:

- Takes `ModelSourceParameters` — resolve whichever fields are set, by priority.
- Reports progress via `IProgress<double>` (0.0 to 1.0) for downloads.
- Returns the path to a local file that downstream loaders can read.
- Throws `ArgumentNullException` on null `modelParameters`.
- Throws `InvalidOperationException` when the file cannot be retrieved.
- Throws `OperationCanceledException` when the operation is canceled.

## Example — S3 provider

```csharp
using Amazon.S3;
using Amazon.S3.Model;
using Aspose.LLM.Abstractions.Interfaces;
using Aspose.LLM.Abstractions.Parameters;

public class S3ModelFileProvider : IModelFileProvider
{
    private readonly IAmazonS3 _s3;
    private readonly string _bucketName;
    private readonly string _localCacheFolder;

    public S3ModelFileProvider(IAmazonS3 s3, string bucketName, string cacheFolder)
    {
        _s3 = s3;
        _bucketName = bucketName;
        _localCacheFolder = cacheFolder;
        Directory.CreateDirectory(_localCacheFolder);
    }

    public async Task<string> GetModelFileAsync(
        ModelSourceParameters modelParameters,
        IProgress<double>? progress = null,
        CancellationToken cancellationToken = default)
    {
        if (modelParameters is null)
            throw new ArgumentNullException(nameof(modelParameters));

        // Map the AsposeModelId field to an S3 key.
        string s3Key = modelParameters.AsposeModelId
            ?? throw new InvalidOperationException("AsposeModelId is required for S3 resolution.");

        string localPath = Path.Combine(_localCacheFolder, Path.GetFileName(s3Key));

        // Cache hit.
        if (File.Exists(localPath))
        {
            progress?.Report(1.0);
            return localPath;
        }

        // Download.
        var response = await _s3.GetObjectAsync(
            new GetObjectRequest { BucketName = _bucketName, Key = s3Key },
            cancellationToken);

        using var outStream = File.Create(localPath);
        long total = response.ContentLength;
        long read = 0;
        byte[] buffer = new byte[81920];
        int n;
        while ((n = await response.ResponseStream.ReadAsync(buffer, cancellationToken)) > 0)
        {
            await outStream.WriteAsync(buffer.AsMemory(0, n), cancellationToken);
            read += n;
            progress?.Report((double)read / total);
        }

        return localPath;
    }
}
```

Usage in a preset: set `AsposeModelId` to the S3 key, leave `HuggingFaceRepoId` unset.

## Example — air-gapped internal registry

For air-gapped deployments where models are prepopulated in a shared directory:

```csharp
public class InternalRegistryProvider : IModelFileProvider
{
    private readonly string _registryRoot;

    public InternalRegistryProvider(string registryRoot) => _registryRoot = registryRoot;

    public Task<string> GetModelFileAsync(
        ModelSourceParameters modelParameters,
        IProgress<double>? progress = null,
        CancellationToken cancellationToken = default)
    {
        string relative = modelParameters.AsposeModelId
            ?? throw new InvalidOperationException("AsposeModelId required.");

        string path = Path.Combine(_registryRoot, relative);
        if (!File.Exists(path))
            throw new InvalidOperationException($"Model not found: {path}");

        progress?.Report(1.0);
        return Task.FromResult(path);
    }
}
```

## Priority handling

`ModelSourceParameters` has three priority-ordered fields: `ModelFilePath`, `AsposeModelId`, `HuggingFaceRepoId` + `HuggingFaceFileName`. Your provider decides which field(s) to respect. Common patterns:

- **Single-source provider**: respect one field only (e.g., `AsposeModelId` for an internal registry). Let callers set that field explicitly.
- **Multi-source provider**: honor the full priority chain — check `ModelFilePath` first, then your custom source, then fall back to Hugging Face.

## Registration

```csharp
using Microsoft.Extensions.DependencyInjection;
using Aspose.LLM.Abstractions.Interfaces;
using Aspose.LLM.Core.DependencyInjection;

services.AddLlamaServices(new Qwen25Preset());
services.AddSingleton<IModelFileProvider>(new InternalRegistryProvider(@"/opt/models"));
```

The default `ModelManager` is registered by `AddLlamaServices`. Adding your own `IModelFileProvider` registration overrides it only if the downstream consumer (`ModelManager`) is wired to consume `IModelFileProvider` directly — in the current SDK, `ModelManager` uses concrete providers (`LocalFilesystemProvider`, `HuggingFaceProvider`). You may need a custom `IModelLoader` that uses your `IModelFileProvider` instead of `ModelManager`.

Confirm the current wiring before committing to a custom provider — contact [Aspose support](https://forum.aspose.com/) if the injection point does not behave as you expect.

## What's next

- [Custom model loader](/net/developer-reference/extensibility/custom-model-loader/) — broader surface when you need full pipeline control.
- [Model source parameters](/net/developer-reference/parameters/model-source/) — what `ModelSourceParameters` carries.
- [Dependency injection](/net/developer-reference/dependency-injection/) — standard service wiring.

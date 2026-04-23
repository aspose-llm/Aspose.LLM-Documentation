---
weight: 65
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/extensibility/
feedback: LLMNET
version: 26.5.0
title: Extensibility
description: Replace core services in Aspose.LLM for .NET via dependency injection — IModelLoader, IModelFileProvider, IPromptFormatter, IMediaProcessor. Advanced customization for niche scenarios.
keywords:
- extensibility
- IModelLoader
- IModelFileProvider
- IPromptFormatter
- IMediaProcessor
- dependency injection
- custom
---

Most users never need to touch extensibility — a preset plus a few parameter overrides covers everything. But for niche scenarios — custom model registries, alternative prompt formats, bespoke media pre-processing — the SDK exposes four public interfaces you can substitute via a DI container.

## When to use extensibility

Consider extending only when:

- Your organization maintains a private model registry (S3, internal GitHub Enterprise, etc.) and you want to avoid Hugging Face / local-file paths.
- You need a custom chat template that does not map to any of the SDK's built-in formatters.
- You have special image preprocessing requirements (specific color spaces, DICOM, sensor calibration).
- You need to inject instrumentation, caching, or auditing at the service boundary.

For everyday customization — different model, sampler settings, GPU layers — use a [preset](/net/developer-reference/presets/) and the [parameter bags](/net/developer-reference/parameters/). That covers the vast majority of cases.

## Available interfaces

| Interface | Replaces | Page |
|---|---|---|
| `IModelLoader` | Full model loading pipeline | [Custom model loader](/net/developer-reference/extensibility/custom-model-loader/) |
| `IModelFileProvider` | File resolution (download / local path) | [Custom file provider](/net/developer-reference/extensibility/custom-file-provider/) |
| `IPromptFormatter` | Chat template formatting | [Custom prompt formatter](/net/developer-reference/extensibility/custom-prompt-formatter/) |
| `IMediaProcessor` | Image preprocessing for vision presets | [Custom media processor](/net/developer-reference/extensibility/custom-media-processor/) |

All four are public types. `IModelLoader`, `IModelFileProvider`, and `IPromptFormatter` live in `Aspose.LLM.Abstractions.Interfaces`. `IMediaProcessor` lives in `Aspose.LLM.Core.Services` (still publicly accessible after ILRepack merges).

## How substitution works

The SDK's default implementations (`LocalFilesystemProvider`, `HuggingFaceProvider`, built-in formatters, `MediaManager`) are wired into `Engine` in `Configuration.AddLlamaServices`. The facade path (`AsposeLLMApi.Create`) instantiates `Engine` the same way.

Neither entry point provides an easy "replace this one implementation" option out of the box. To substitute an interface, you take the DI path and configure the container yourself:

```csharp
using Microsoft.Extensions.DependencyInjection;
using Aspose.LLM.Abstractions.Interfaces;
using Aspose.LLM.Core.DependencyInjection;

var services = new ServiceCollection();
services.AddLlamaServices(new Qwen25Preset());

// Substitute the default file provider with your implementation:
services.AddSingleton<IModelFileProvider, MyS3ModelFileProvider>();

// Build the container and resolve Engine.
var sp = services.BuildServiceProvider();
var engine = sp.GetRequiredService<Engine>();
```

Note: substitution of some services requires matching the downstream wiring. The SDK's `Engine` constructor has specific parameter shapes — review the [AsposeLLMApi](/net/developer-reference/asposellmapi/) and [Dependency injection](/net/developer-reference/dependency-injection/) references to confirm what each service needs to provide.

{{% alert color="primary" %}}
Extensibility is an advanced path. Custom implementations replace well-tested production code — validate thoroughly before using in production. The default implementations in the SDK are the reference for the contract each interface expects; read their source if available, or contact [Aspose support](https://forum.aspose.com/) for the exact requirements.
{{% /alert %}}

## What's next

- [Custom model loader](/net/developer-reference/extensibility/custom-model-loader/) — `IModelLoader`.
- [Custom file provider](/net/developer-reference/extensibility/custom-file-provider/) — `IModelFileProvider`.
- [Custom prompt formatter](/net/developer-reference/extensibility/custom-prompt-formatter/) — `IPromptFormatter`.
- [Custom media processor](/net/developer-reference/extensibility/custom-media-processor/) — `IMediaProcessor`.
- [Dependency injection](/net/developer-reference/dependency-injection/) — the standard `AddLlamaServices` path.

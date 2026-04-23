---
weight: 60
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/dependency-injection/
feedback: LLMNET
version: 26.5.0
title: Dependency injection
description: Register Aspose.LLM for .NET services in a Microsoft.Extensions.DependencyInjection container via AddLlamaServices. Alternative entry point for ASP.NET Core and Worker Service applications.
keywords:
- dependency injection
- AddLlamaServices
- IServiceCollection
- ASP.NET Core
- Worker Service
- DI
- Engine
---

Aspose.LLM for .NET provides an extension method for `IServiceCollection` that registers the engine and its dependencies in a Microsoft.Extensions.DependencyInjection container. This is the idiomatic entry point for ASP.NET Core, Worker Service, and any application that wires services through DI.

The [`AsposeLLMApi`](/net/developer-reference/asposellmapi/) facade and the DI path are **two ways to reach the same engine**. Use whichever suits your application shape:

- **Facade** — simple console apps, CLI tools, scripts. One object to instantiate and dispose.
- **DI** — ASP.NET Core, hosted services, Workers. Engine and its dependencies become DI-resolvable singletons.

## Method signature

```csharp
namespace Aspose.LLM.Core.DependencyInjection;

public static class Configuration
{
    public static IServiceCollection AddLlamaServices(
        this IServiceCollection services,
        PresetCoreBase preset,
        Action<PresetCoreBase>? configure = null);
}
```

Arguments:

- `services` — the DI container's service collection.
- `preset` — the preset that supplies engine, model, context, sampler, and binary settings.
- `configure` — optional callback to tweak the preset programmatically before registration.

## What gets registered

`AddLlamaServices` registers these services as **singletons**:

| Service | Purpose |
|---|---|
| `PresetCoreBase` | The preset itself, resolvable by type or subtype. |
| `EngineParameters` | Extracted from the preset. |
| `BinaryManagerParameters` | Extracted from the preset. |
| `LocalFilesystemProvider` | Resolves local-path models. |
| `HuggingFaceProvider` | Downloads from Hugging Face. |
| `ModelManager` | Resolves and loads models. |
| `BinaryManager` | Deploys native binaries. |
| `Engine` | The core inference engine. |
| `NativeLoggerAdapter` | Bridges `llama.cpp` native logs to `ILogger`. |
| `ILoggerFactory` & friends | Console + file logging, level controlled by `EnableDebugLogging`. |

`Engine` is constructed with `presetToLoad: preset`, which means **the model is loaded synchronously the first time `Engine` is resolved** — same as the facade's `Create`.

Logging is automatically configured:

- A `console` logger provider.
- A file logger provider that writes to `EngineParameters.LogDirectoryPath`.
- Level `Debug` when `EngineParameters.EnableDebugLogging = true`; otherwise `None`.

## Minimal ASP.NET Core setup

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;
using Aspose.LLM.Core.DependencyInjection;
using Aspose.LLM.Core.Services;

var builder = WebApplication.CreateBuilder(args);

// Apply license once during startup.
var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

// Register LLM services.
builder.Services.AddLlamaServices(new Qwen25Preset());

var app = builder.Build();

app.MapPost("/chat", async (string message, Engine engine, CancellationToken ct) =>
{
    string sessionId = await engine.InitiateNewSession(
        preset: app.Services.GetRequiredService<PresetCoreBase>());
    return await engine.GetChatSessionResponse(sessionId, message, null, ct);
});

app.Run();
```

Resolve `Engine` — or any registered service — via constructor injection or `app.Services.GetRequiredService<T>()`.

## Customizing the preset at registration

Pass a `configure` callback to adjust the preset before it is stored:

```csharp
builder.Services.AddLlamaServices(new Qwen25Preset(), preset =>
{
    preset.ContextParameters.ContextSize = 16384;
    preset.SamplerParameters.Temperature = 0.3f;
    preset.ChatParameters.SystemPrompt = "You are a concise enterprise assistant.";
    preset.BaseModelInferenceParameters.GpuLayers = 999;
});
```

The callback runs once, before the preset is registered as a singleton. Mutations inside the callback are visible to every resolved service.

## Worker Service example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;
using Aspose.LLM.Core.DependencyInjection;
using Aspose.LLM.Core.Services;
using Microsoft.Extensions.Hosting;

await Host.CreateDefaultBuilder(args)
    .ConfigureServices((ctx, services) =>
    {
        var license = new Aspose.LLM.License();
        license.SetLicense("Aspose.LLM.lic");

        services.AddLlamaServices(new Qwen25Preset());
        services.AddHostedService<ChatWorker>();
    })
    .Build()
    .RunAsync();

public class ChatWorker : BackgroundService
{
    private readonly Engine _engine;
    private readonly PresetCoreBase _preset;

    public ChatWorker(Engine engine, PresetCoreBase preset)
    {
        _engine = engine;
        _preset = preset;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        string sessionId = await _engine.InitiateNewSession(preset: _preset);
        while (!stoppingToken.IsCancellationRequested)
        {
            string reply = await _engine.GetChatSessionResponse(
                sessionId,
                "Summarize the latest event queue.",
                null,
                stoppingToken);

            // ... do something with reply
            await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
        }
    }
}
```

## Constraints that still apply

- **Single instance per process.** The underlying `Engine` shares the single-instance guard with `AsposeLLMApi`. Registering `AddLlamaServices` in a web host and also creating an `AsposeLLMApi` on the side throws. Pick one entry point.
- **License is still required** before chat methods. Apply the license before any request handler calls `Engine` methods.
- **Model loads on first resolve** — the first request hitting `Engine` can take minutes on a cold machine. Consider eager resolution at startup:

  ```csharp
  app.Services.GetRequiredService<Engine>(); // trigger first-time model load
  ```

  Or warm up in a hosted service so the first user request is not slow.

## What's next

- [AsposeLLMApi facade](/net/developer-reference/asposellmapi/) — the simpler non-DI entry point.
- [Engine parameters](/net/developer-reference/parameters/engine/) — logging and threading defaults applied via `AddLlamaServices`.
- [Licensing](/net/licensing/) — license application in host startup.

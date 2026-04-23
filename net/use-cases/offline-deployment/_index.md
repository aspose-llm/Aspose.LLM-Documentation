---
weight: 80
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/offline-deployment/
feedback: LLMNET
version: 26.5.0
title: Offline deployment
description: Deploy Aspose.LLM for .NET to air-gapped, firewalled, or no-internet environments — pre-download native binaries and model files, wire up local cache paths.
keywords:
- offline
- air-gapped
- firewall
- no internet
- cache
- BinaryPath
- ModelCachePath
- pre-download
---

Aspose.LLM for .NET downloads native `llama.cpp` binaries and model files from the internet on first use. In an air-gapped or firewalled deployment this fails. The solution: pre-download everything on a connected machine, copy the caches to the target host, and point the preset at the local paths.

## When to use this pattern

- Production hosts behind a strict firewall or in a VPC without public internet.
- Air-gapped lab or compliance environments.
- CI/CD pipelines that should not hit external services during test runs.
- Offline edge devices (kiosks, field equipment).

## Prerequisites

- A connected machine where you can run the SDK once to populate the caches.
- The same platform / acceleration profile on the connected machine as the production host (so the downloaded binaries match).
- [Install the NuGet package](/net/installation/) on both.
- [Apply a license](/net/licensing/) on the production host.

## Step 1. Populate caches on a connected machine

Run your target preset once on a connected machine. The SDK downloads:

- Native `llama.cpp` binaries to `BinaryManagerParameters.BinaryPath` (default `<LocalAppData>/Aspose.LLM/runtimes`).
- Model and `mmproj` files to `EngineParameters.ModelCachePath` (default `<LocalAppData>/Aspose.LLM/models`).

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();
// Optional — set explicit paths so you know what to copy.
preset.BinaryManagerParameters.BinaryPath = @"C:\aspose-prep\runtimes";
preset.EngineParameters.ModelCachePath = @"C:\aspose-prep\models";

using var api = AsposeLLMApi.Create(preset);
await api.SendMessageAsync("ping");
Console.WriteLine("Caches populated.");
```

After this runs, `C:\aspose-prep\runtimes` and `C:\aspose-prep\models` contain everything needed.

## Step 2. Copy caches to the offline host

Transfer both folders to the target host. On Linux:

```bash
# On the connected machine:
tar czf aspose-prep.tar.gz -C /var/cache/aspose-llm runtimes models

# On the offline host:
tar xzf aspose-prep.tar.gz -C /opt/aspose-llm
```

On Windows, copy the folders via your usual transfer mechanism (SFTP, USB, system image).

## Step 3. Point the production preset at the local paths

On the offline host, configure the preset to use the pre-populated caches:

```csharp
var preset = new Qwen25Preset();
preset.BinaryManagerParameters.BinaryPath = @"/opt/aspose-llm/runtimes";
preset.EngineParameters.ModelCachePath = @"/opt/aspose-llm/models";

using var api = AsposeLLMApi.Create(preset);
```

The engine finds the binaries and model in the cache and skips network calls.

## Step 4. Verify offline

Disconnect the host from the network (or set no-internet firewall rules), apply the license, and run a smoke test:

```csharp
string reply = await api.SendMessageAsync("Say hello.");
Console.WriteLine(reply);
```

If the call succeeds without network access, the deployment is ready.

## Multi-preset offline

Pre-download every preset your production code might use. Each preset references its own Hugging Face repo and file, so the connected-machine step must instantiate every preset:

```csharp
// Populate caches for several presets in one pass.
var presets = new PresetCoreBase[]
{
    new Qwen25Preset(),
    new Llama32Preset(),
    new Qwen3VL2BPreset(),
};

foreach (var preset in presets)
{
    preset.BinaryManagerParameters.BinaryPath = @"C:\aspose-prep\runtimes";
    preset.EngineParameters.ModelCachePath = @"C:\aspose-prep\models";

    using var api = AsposeLLMApi.Create(preset);
    await api.SendMessageAsync("ping"); // force full load
    // api disposed here — next preset can take the single-instance slot.
}
```

Native binaries for the same `ReleaseTag` and platform are shared across presets — downloaded once.

## Pinning `ReleaseTag`

For deterministic offline deployments, pin `BinaryManagerParameters.ReleaseTag` explicitly rather than relying on defaults:

```csharp
preset.BinaryManagerParameters.ReleaseTag = "b8816";
```

The default in v26.5.0 is already `b8816`. If you upgrade the SDK later, a new default `ReleaseTag` may require a fresh binary download — re-populate the cache.

## Docker / container deployments

Bake the populated caches into your container image:

```dockerfile
FROM mcr.microsoft.com/dotnet/runtime:10.0 AS base

# Pre-populated on the build machine:
COPY aspose-caches/runtimes /var/aspose/runtimes
COPY aspose-caches/models /var/aspose/models

COPY app/ /app/
WORKDIR /app

ENV ASPOSE_BINARY_PATH=/var/aspose/runtimes
ENV ASPOSE_MODEL_CACHE=/var/aspose/models

ENTRYPOINT ["dotnet", "MyApp.dll"]
```

Read the env vars in your application and set them on the preset:

```csharp
preset.BinaryManagerParameters.BinaryPath =
    Environment.GetEnvironmentVariable("ASPOSE_BINARY_PATH") ?? preset.BinaryManagerParameters.BinaryPath;

preset.EngineParameters.ModelCachePath =
    Environment.GetEnvironmentVariable("ASPOSE_MODEL_CACHE") ?? preset.EngineParameters.ModelCachePath;
```

## Common errors

- **`FileNotFoundException` at Create** — cache paths wrong, or the pre-populated folder is missing an asset for the requested `ReleaseTag`. Verify both folders.
- **License fails** — license file also needs to travel to the offline host.
- **Binary mismatch** — connected and offline machines have different CPUs (one has AVX-512, the other does not). Pre-download on a machine with the same or lower AVX level.

## What's next

- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — full reference for `BinaryPath`, `ReleaseTag`, `PreferredAcceleration`.
- [Engine parameters](/net/developer-reference/parameters/engine/) — `ModelCachePath`.
- [Installation](/net/installation/) — initial NuGet setup.

---
weight: 20
date: "2026-11-05"
author: "Alex Golshtein"
type: docs
url: /net/installation/
feedback: LLMNET
version: 26.11.1
title: Installation
description: Install the Aspose.LLM NuGet package, understand the package payload, and prepare for the first-run native binary download.
keywords:
- install
- NuGet
- dotnet add package
- package
- setup
- ILRepack
- .NET 8
- .NET 10
---

Install Aspose.LLM for .NET from NuGet to add the SDK and resolve its managed dependencies.

## Prerequisites

- A supported .NET runtime. See [System requirements](/llm/net/system-requirements/).
- A NuGet package source reachable from your build machine (nuget.org by default).

## Add the package

### `dotnet` CLI

From your project directory:

```bash
dotnet add package Aspose.LLM
```

To pin a specific version:

```bash
dotnet add package Aspose.LLM --version 26.11.1
```

### Visual Studio

1. Open your solution.
2. Right-click the project, choose **Manage NuGet Packages**.
3. Browse for **Aspose.LLM** and click **Install**.
4. Accept the license prompt.

### Package Manager Console

```powershell
Install-Package Aspose.LLM
```

## What ships in the package

The NuGet package contains a single managed assembly: **`Aspose.LLM.dll`**. The internal assemblies (`Aspose.LLM.Abstractions`, `Aspose.LLM.Core`, `Aspose.LLM.Interop`) are merged into it with ILRepack, so your project references only one DLL.

NuGet resolves these direct dependencies during restore:

- `System.Text.Json` 9.0.0: model source handling and session serialization.
- `Microsoft.Extensions.Logging.Abstractions` 2.1.1: optional `ILogger` integration.
- `Microsoft.Extensions.DependencyInjection.Abstractions` 2.1.1: service registration.
- `Microsoft.Extensions.Hosting.Abstractions` 2.1.1: host integration.

Additional internal package references are merged into the main assembly and do not show up as separate references in your project.

**Native `llama.cpp` binaries are not bundled.** They are downloaded from GitHub on first use.

### Use the DLL from the MSI installer or ZIP archive

Reference `Aspose.LLM.dll` in your project and add the managed dependencies from the matching NuGet package metadata as separate package references. The MSI and ZIP include the Aspose.LLM assembly. NuGet supplies the Microsoft and System dependencies.

## First-run native binary download

The first time you call `AsposeLLMApi.Create(preset)` in a fresh environment, the SDK downloads the native `llama.cpp` binaries that match your platform and acceleration backend. This is a one-time operation:

- Size: 100-500 MB, depending on the acceleration backend (CUDA downloads are larger than CPU).
- Source: GitHub release assets for the configured `ReleaseTag` (default: `b10988`). The configured supplementary runtime repository can supply backends absent from the primary release.
- Location: a per-user cache folder; override via `BinaryManagerParameters.BinaryPath` on the preset or `ASPOSE_LLM_RUNTIME_CACHE_DIR` when the preset keeps its default path.

Subsequent runs use the local cache.

{{% alert color="primary" %}}
If your build server or production machine runs behind a strict firewall or proxy, pre-download the native binaries on a machine with internet access and point `BinaryManagerParameters.BinaryPath` at the pre-populated cache folder. For model files, follow the instructions for [using a local GGUF model](/llm/net/use-cases/bring-your-own-gguf/).
{{% /alert %}}

## Verify the install

A minimal compile-only check:

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

// Build should succeed after the package is restored.
_ = typeof(AsposeLLMApi);
_ = typeof(Qwen25Preset);
```

For a runnable first example, continue to [Hello, world!](/llm/net/hello-world/).

To verify the same package in a new .NET 10 project:

```bash
dotnet new console -f net10.0 -n AsposeLlmQuickStart
cd AsposeLlmQuickStart
dotnet add package Aspose.LLM --version 26.11.1
dotnet build
```

The `net10.0` project consumes the package's `netstandard2.0` assembly; no separate .NET 10 package is required. Use `-f net8.0` to create a .NET 8 project with the same package.

.NET Framework 4.7.2 projects use the same package. Version 26.11.1 removes the `NETStandard.Library` framework assembly reference that could trigger `MSB3245` on .NET Framework.

## Updating

Update to a newer version via the same tooling:

```bash
dotnet add package Aspose.LLM --version 26.11.1
```

See the [26.11.1 release notes](https://releases.aspose.com/llm/net/release-notes/2026/aspose-llm-for-net-26-11-1-release-notes/). If you are upgrading from a version earlier than 26.11.0, update any public API identifiers that were renamed in 26.11.0.

## What's next

- [Licensing](/llm/net/licensing/): apply a license; required for inference.
- [Hello, world!](/llm/net/hello-world/): run the first example.
- [Architecture](/llm/net/product-overview/architecture/): understand the layers and what happens on first `Create`.

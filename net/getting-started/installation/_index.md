---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/installation/
feedback: LLMNET
version: 26.5.0
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

Install Aspose.LLM for .NET from NuGet and add it to your project. A single package contains the full SDK — no additional references are needed.

## Prerequisites

- A supported .NET runtime. See [System requirements](/net/system-requirements/).
- A NuGet package source reachable from your build machine (nuget.org by default).

## Add the package

### `dotnet` CLI

From your project directory:

```bash
dotnet add package Aspose.LLM
```

To pin a specific version:

```bash
dotnet add package Aspose.LLM --version 26.5.0
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

Direct NuGet dependencies:

- `System.Text.Json` 8.0.0 — session serialization.
- `Microsoft.Extensions.Logging.Abstractions` 2.1.1 — optional `ILogger` integration.

Additional internal package references are merged into the main assembly and do not show up as separate references in your project.

**Native `llama.cpp` binaries are not bundled.** They are downloaded from GitHub on first use.

## First-run native binary download

The first time you call `AsposeLLMApi.Create(preset)` in a fresh environment, the SDK downloads the native `llama.cpp` binaries that match your platform and acceleration backend. This is a one-time operation:

- Size: 100-500 MB, depending on the acceleration backend (CUDA downloads are larger than CPU).
- Source: `github.com/ggml-org/llama.cpp/releases/tag/<ReleaseTag>` (default tag: `b8816`).
- Location: a per-user cache folder; override via `BinaryManagerParameters.BinaryPath` on the preset.

Subsequent runs use the local cache.

{{% alert color="primary" %}}
If your build server or production machine runs behind a strict firewall or proxy, pre-download the binaries on a machine with internet access and point `BinaryManagerParameters.BinaryPath` at the pre-populated cache folder. The same applies to Hugging Face model downloads (`EngineParameters.ModelCachePath`).
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

For a runnable first example, continue to [Hello, world!](/net/hello-world/).

## Updating

Update to a newer version via the same tooling:

```bash
dotnet add package Aspose.LLM --version 26.6.0
```

Between minor versions, the API is generally stable. Between major versions, review the release notes for any breaking changes, especially to the session persistence format.

## What's next

- [Licensing](/net/licensing/) — apply a license; required for inference.
- [Hello, world!](/net/hello-world/) — run the first example.
- [Architecture](/net/product-overview/architecture/) — understand the layers and what happens on first `Create`.

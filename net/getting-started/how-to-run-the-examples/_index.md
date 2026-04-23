---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/how-to-run-the-examples/
feedback: LLMNET
version: 26.5.0
title: How to run examples
description: Run the code samples from this documentation — set up a project, apply a license, and execute the Aspose.LLM for .NET examples.
keywords:
- examples
- samples
- run
- dotnet run
- project setup
---

Every code sample in this documentation is a self-contained, compilable fragment. Drop it into a .NET 8 or .NET 10 console project, apply a license, and run. This page shows the minimum project setup.

## Create a project

From an empty directory:

```bash
dotnet new console -n AsposeLlmQuickStart
cd AsposeLlmQuickStart
dotnet add package Aspose.LLM
```

The resulting project targets the .NET version you have installed by default. Verify the target framework in the generated `.csproj`:

```xml
<PropertyGroup>
  <OutputType>Exe</OutputType>
  <TargetFramework>net10.0</TargetFramework>
  <ImplicitUsings>enable</ImplicitUsings>
  <Nullable>enable</Nullable>
</PropertyGroup>
```

`net8.0` and `net6.0` are also supported — see [System requirements](/net/system-requirements/) for the full list.

## Place the license

Put your `Aspose.LLM.lic` file next to the executable (or at a path you pass to `SetLicense`). The simplest approach during development is to include it in the project and copy it to the output:

```xml
<ItemGroup>
  <None Update="Aspose.LLM.lic">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </None>
</ItemGroup>
```

For production, consider embedding the license as a resource or loading it from a secret store — see [Licensing](/net/licensing/).

## Run a sample

Paste the sample code into `Program.cs`. For [Hello, world!](/net/hello-world/):

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(preset);

string response = await api.SendMessageAsync("Say hello in one short sentence.");
Console.WriteLine(response);
```

Run:

```bash
dotnet run
```

## First-run expectations

On first run with a given preset:

1. NuGet restores the `Aspose.LLM` package (seconds).
2. `AsposeLLMApi.Create(preset)` downloads native binaries from GitHub (1-5 minutes, 100-500 MB).
3. The engine downloads the preset's GGUF model from Hugging Face (2-15 minutes, 2-15 GB).
4. The model loads into memory (5-30 seconds).

Subsequent runs use the local caches and complete in seconds.

## IDE usage

Any IDE with .NET support works — see [System requirements](/net/system-requirements/) for recommended options. The `dotnet` CLI used above is all you need to build and run; the IDE is optional.

## Where to find more samples

- [Quick wins](/net/quick-wins/) — five compact recipes.
- [Use cases](/net/use-cases/) — full running examples for common scenarios.

## What's next

- [Hello, world!](/net/hello-world/) — the first complete example.
- [Quick wins](/net/quick-wins/) — more compact recipes.
- [Licensing](/net/licensing/) — full license setup options (file, stream, embedded resource).

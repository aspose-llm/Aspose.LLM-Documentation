---
weight: 40
date: "2026-11-05"
author: "Alex Golshtein"
type: docs
url: /net/how-to-run-the-examples/
feedback: LLMNET
version: 26.11.0
title: How to run examples
description: Run the code samples from this documentation, set up a project, apply a license, and execute the Aspose.LLM for .NET examples.
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
dotnet new console -f net10.0 -n AsposeLlmQuickStart
cd AsposeLlmQuickStart
dotnet add package Aspose.LLM --version 26.11.0
```

The `-f net10.0` option selects .NET 10 explicitly. Verify the target framework in the generated `.csproj`:

```xml
<PropertyGroup>
  <OutputType>Exe</OutputType>
  <TargetFramework>net10.0</TargetFramework>
  <ImplicitUsings>enable</ImplicitUsings>
  <Nullable>enable</Nullable>
</PropertyGroup>
```

To run the same example on .NET 8, change the `dotnet new` option to `-f net8.0`; the Aspose.LLM package version stays the same. See [System requirements](/llm/net/system-requirements/) for other supported runtimes.

## Place the license

Put your `Aspose.LLM.lic` file next to the executable (or at a path you pass to `SetLicense`). The simplest approach during development is to include it in the project and copy it to the output:

```xml
<ItemGroup>
  <None Update="Aspose.LLM.lic">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </None>
</ItemGroup>
```

For production, consider embedding the license as a resource or loading it from a secret store: see [Licensing](/llm/net/licensing/).

## Run a sample

Obtain a compatible GGUF file separately after reviewing its publisher's license. Set `ASPOSE_LLM_MODEL_FILE` to the local file path, then paste the sample into `Program.cs`. For [Hello, world!](/llm/net/hello-world/):

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();
var modelPath = Environment.GetEnvironmentVariable("ASPOSE_LLM_MODEL_FILE");
if (string.IsNullOrWhiteSpace(modelPath) || !File.Exists(modelPath))
    throw new InvalidOperationException("Provide a local Qwen 2.5 7B Instruct GGUF via ASPOSE_LLM_MODEL_FILE. Preset source: https://huggingface.co/bartowski/Qwen2.5-7B-Instruct-GGUF; model license: https://www.apache.org/licenses/LICENSE-2.0");
preset.BaseModelSourceParameters.ModelFilePath = modelPath;
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
3. The engine loads the local GGUF file into memory.

Subsequent runs reuse the native runtime cache. Version 26.11.0 may fetch a configured remote model if no local model path is provided; an opt-in control is not yet implemented.

## IDE usage

Any IDE with .NET support works: see [System requirements](/llm/net/system-requirements/) for recommended options. The `dotnet` CLI used above is all you need to build and run; the IDE is optional.

## Where to find more samples

- [Quick wins](/llm/net/quick-wins/): five compact recipes.
- [Use cases](/llm/net/use-cases/): full running examples for common scenarios.

## What's next

- [Hello, world!](/llm/net/hello-world/): the first complete example.
- [Quick wins](/llm/net/quick-wins/): more compact recipes.
- [Licensing](/llm/net/licensing/): full license setup options (file, stream, embedded resource).

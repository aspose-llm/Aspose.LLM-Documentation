---
weight: 50
date: "2026-11-05"
author: "Alex Golshtein"
type: docs
url: /net/hello-world/
feedback: LLMNET
version: 26.11.0
title: Hello, world!
description: Minimal end-to-end example with Aspose.LLM for .NET, create the API from a preset, send a message, and print the response.
keywords:
- hello world
- example
- minimal
- first run
- Qwen25Preset
---

This page walks through the smallest possible Aspose.LLM for .NET program. It applies a license, creates an API instance from a preset, sends one message, and prints the response.

## Prerequisites

- [Install](/llm/net/installation/) the Aspose.LLM NuGet package.
- Obtain a [license](/llm/net/licensing/) (a free [temporary license](https://purchase.aspose.com/temporary-license) works for evaluation).
- Enough RAM for the chosen model. `Qwen25Preset` needs roughly 8-12 GB: see [System requirements](/llm/net/system-requirements/).
- Obtain the model file from its publisher's chosen source after reviewing its license. Set `ASPOSE_LLM_MODEL_FILE` to its local path.
- Internet access for native runtime binaries if they are not already cached.

{{% alert color="primary" %}}
The example uses a model file you have obtained separately. Native `llama.cpp` binaries may be downloaded from GitHub on first use and cached locally. Version 26.11.0 can also fetch a configured remote model when no local path is supplied; the opt-in download control is not yet implemented. Set the local path as shown before creating the API.
{{% /alert %}}

## Code

### Step 1. Apply the license

Apply the license once, before constructing the API. Aspose.LLM does not run inference in evaluation mode.

```csharp
using Aspose.LLM;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");
```

### Step 2. Create the API

Create an instance from a built-in preset. `Qwen25Preset` is a good general-purpose starting point.

```csharp
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen25Preset();
var modelPath = Environment.GetEnvironmentVariable("ASPOSE_LLM_MODEL_FILE");
if (string.IsNullOrWhiteSpace(modelPath) || !File.Exists(modelPath))
    throw new InvalidOperationException("Provide a local Qwen 2.5 7B Instruct GGUF via ASPOSE_LLM_MODEL_FILE. Preset source: https://huggingface.co/bartowski/Qwen2.5-7B-Instruct-GGUF; model license: https://www.apache.org/licenses/LICENSE-2.0");
preset.BaseModelSourceParameters.ModelFilePath = modelPath;
using var api = AsposeLLMApi.Create(preset);
```

The `using` pattern disposes the API automatically when the block exits. Disposal unloads the model and releases the single-instance guard.

### Step 3. Send a message

If no chat session exists yet, `SendMessageAsync` creates one implicitly.

```csharp
string response = await api.SendMessageAsync("Say hello in one short sentence.");
Console.WriteLine(response);
// Output: Hello! Nice to meet you.
```

## Full example

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

Save this as `Program.cs` in a new .NET 8 or .NET 10 project, ensure `Aspose.LLM.lic` is next to the executable, and run:

```bash
dotnet run
```

## Troubleshooting

- **`Not licensed for this method`**: the license has not been applied. Call `SetLicense` before `AsposeLLMApi.Create`. See [Licensing](/llm/net/licensing/).
- **First `Create` takes a long time**: the native runtime may need downloading, and model loading can take time. Check the local model path and monitor progress via `ILogger` (pass one to `AsposeLLMApi.Create(preset, logger)`).
- **`Only one AsposeLLMApi instance can be created at a time`**: a previous instance has not been disposed yet. Keep one instance for the process lifetime, or dispose the old one first.

## What's next

- [Quick wins](/llm/net/quick-wins/): image input, session save/restore, CPU-only and GPU runs.
- [Multi-turn chat](/llm/net/use-cases/multi-turn-chat/): start an explicit session and exchange several messages.
- [Developer's reference](/llm/net/developer-reference/): presets, sessions, persistence, and API details.

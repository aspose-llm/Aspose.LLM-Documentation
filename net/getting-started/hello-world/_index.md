---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/hello-world/
feedback: LLMNET
version: 26.5.0
title: Hello, world!
description: Minimal end-to-end example with Aspose.LLM for .NET — create the API from a preset, send a message, and print the response.
keywords:
- hello world
- example
- minimal
- first run
- Qwen25Preset
---

This page walks through the smallest possible Aspose.LLM for .NET program. It applies a license, creates an API instance from a preset, sends one message, and prints the response.

## Prerequisites

- [Install](/net/installation/) the Aspose.LLM NuGet package.
- Obtain a [license](/net/licensing/) (a free [temporary license](https://purchase.aspose.com/temporary-license) works for evaluation).
- Enough RAM for the chosen model. `Qwen25Preset` needs roughly 8-12 GB — see [System requirements](/net/system-requirements/).
- Internet access on first run (see the warning below).

{{% alert color="primary" %}}
The first `AsposeLLMApi.Create(preset)` call downloads native `llama.cpp` binaries (100-500 MB) from GitHub and the model file (~4 GB for `Qwen25Preset` Q4_K_M) from Hugging Face. Expect several minutes on first run; subsequent runs use local caches.
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
using var api = AsposeLLMApi.Create(preset);

string response = await api.SendMessageAsync("Say hello in one short sentence.");
Console.WriteLine(response);
```

Save this as `Program.cs` in a new .NET 8 or .NET 10 project, ensure `Aspose.LLM.lic` is next to the executable, and run:

```bash
dotnet run
```

## Troubleshooting

- **`Not licensed for this method`** — the license has not been applied. Call `SetLicense` before `AsposeLLMApi.Create`. See [Licensing](/net/licensing/).
- **First `Create` hangs for a long time** — it is downloading binaries and the model. On a slow network, budget 5-15 minutes. Monitor progress via `ILogger` (pass one to `AsposeLLMApi.Create(preset, logger)`).
- **`Only one AsposeLLMApi instance can be created at a time`** — a previous instance has not been disposed yet. Keep one instance for the process lifetime, or dispose the old one first.

## What's next

- [Quick wins](/net/quick-wins/) — image input, session save/restore, CPU-only and GPU runs.
- [Multi-turn chat](/net/use-cases/multi-turn-chat/) — start an explicit session and exchange several messages.
- [Developer's reference](/net/developer-reference/) — presets, sessions, persistence, and API details.

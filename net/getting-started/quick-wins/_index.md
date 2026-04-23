---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/quick-wins/
feedback: LLMNET
version: 26.5.0
title: Quick wins
description: Five compact recipes for common Aspose.LLM for .NET tasks — first message, image input, session save/restore, CPU-only run, and CUDA GPU run.
keywords:
- quick wins
- recipes
- examples
- first steps
- chat
- vision
- CPU
- GPU
---

Compact recipes for the tasks you are most likely to try first. Each recipe is 10-20 lines and links to a full use case with the running end-to-end example.

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/) or accept evaluation-mode limits.
- Be ready for a one-time binary download on first `Create` (100-500 MB, depending on preset and acceleration).

## Baseline pattern

Every recipe below assumes this outer pattern:

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(preset);

// ... recipe body goes here.
```

Recipes show only the body. For full runnable examples, follow the links at the end of each recipe.

## 1. Send your first message

Ask the model a single question and print the reply.

```csharp
string reply = await api.SendMessageAsync("What is 2 + 2?");
Console.WriteLine(reply);
// Output: 2 + 2 equals 4.
```

`SendMessageAsync` creates a session if none exists and returns the complete response.

→ Full example: [Simple chat](/net/use-cases/simple-chat/).

## 2. Attach an image to a message

Use a vision preset and pass image bytes via the `media` parameter.

```csharp
// Replace the preset in the baseline pattern:
var preset = new Qwen25VL3BPreset();
using var api = AsposeLLMApi.Create(preset);

byte[] imageBytes = File.ReadAllBytes("cat.jpg");

string reply = await api.SendMessageAsync(
    "Describe what you see in one sentence.",
    media: new[] { imageBytes });

Console.WriteLine(reply);
// Output: A gray tabby cat sits on a wooden floor near a window.
```

Supported image formats: JPEG, PNG, BMP, GIF, WebP. Per-attachment limit: 50 MB.

→ Full example: [Use cases](/net/use-cases/) (vision-qa — in preparation).

## 3. Save and resume a chat session

Persist a conversation to disk and continue it later.

```csharp
// Start and exchange a few messages.
string sessionId = await api.StartNewChatAsync();
await api.SendMessageToSessionAsync(sessionId, "Remember: my project is called Helios.");

// Save it.
api.SaveChatSession(sessionId, "helios-chat.json");

// ... later, in the same or another process:
string resumedId = await api.LoadChatSession("helios-chat.json");
string reply = await api.SendMessageToSessionAsync(resumedId, "What is my project called?");
Console.WriteLine(reply);
// Output: Your project is called Helios.
```

The saved file contains the message history and the KV cache positions. The format is internal and may change between major SDK versions.

→ Full example: [Save and restore session](/net/use-cases/save-and-restore-session/).

## 4. Run on CPU only

Force all computation to the CPU by setting `GpuLayers = 0` on the preset before creating the API.

```csharp
// Between creating the preset and creating the API:
var preset = new Qwen25Preset();
preset.BaseModelInferenceParameters.GpuLayers = 0;

using var api = AsposeLLMApi.Create(preset);

string reply = await api.SendMessageAsync("Summarize CPU inference in one sentence.");
Console.WriteLine(reply);
```

On first use, `BinaryManager` still selects the best CPU variant available (AVX2, AVX512, or a no-AVX fallback). Expect lower throughput than GPU inference; a 7B Q4 model typically produces 5-15 tokens/second on a modern CPU.

→ Related: [Use cases](/net/use-cases/) (cpu-only-deployment — in preparation).

## 5. Run on a CUDA GPU

Request CUDA acceleration and offload all layers to the GPU. Make sure an NVIDIA driver and a compatible CUDA runtime are installed.

```csharp
using Aspose.LLM.Abstractions.Acceleration;

var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.GpuLayers = 999; // offload all layers

using var api = AsposeLLMApi.Create(preset);

string reply = await api.SendMessageAsync("Summarize GPU inference in one sentence.");
Console.WriteLine(reply);
```

`GpuLayers = 999` is an idiomatic way to request full offload. The engine caps the value at the model's actual layer count.

On first use, `BinaryManager` downloads the CUDA variant of the native binaries — larger than the CPU variant (typically 400-800 MB). Subsequent runs use the local cache.

→ Related: [Use cases](/net/use-cases/) (gpu-deployment-cuda — in preparation).

## What's next

- [Hello, world!](/net/hello-world/) — the minimum-possible example, explained step by step.
- [Use cases](/net/use-cases/) — full running examples for every scenario.
- [Supported presets](/net/product-overview/supported-presets/) — pick a preset that fits your model and hardware.
- [Architecture](/net/product-overview/architecture/) — what happens behind the scenes on first `Create`.

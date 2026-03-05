---
weight: 50
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/hello-world/
feedback: LLMNET
title: Hello, world!
description: Minimal example with Aspose.LLM for .NET: create API, send a message, get a response.
keywords:
- hello world
- example
- minimal
---

This example shows how to create an `AsposeLLMApi` instance from a preset, send a single message, and print the response.

## Prerequisites

- [Install](/net/installation/) the Aspose.LLM NuGet package.
- Apply a [license](/net/licensing/) or run in evaluation mode.
- Enough memory and (if needed) disk space for the model used by the preset.

## Code

1. Create an API instance using a preset (e.g. `Qwen25Preset` from `Aspose.LLM.Abstractions.Parameters.Presets`):

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(preset);
```

2. Send a message. If no session exists, the API starts one automatically:

```csharp
string response = await api.SendMessageAsync("Hello, say one short sentence about yourself.");
Console.WriteLine(response);
```

3. Dispose the API when done (or use `using` as above).

Full example:

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(preset);
string response = await api.SendMessageAsync("Hello, say one short sentence about yourself.");
Console.WriteLine(response);
```

## What's next

- [Multi-turn chat](/net/use-cases/multi-turn-chat/) — start a session explicitly and send several messages.
- [Save and restore session](/net/use-cases/save-and-restore-session/) — persist conversation state to disk.
- [Developer's reference](/net/developer-reference/) — presets, sessions, and API details.

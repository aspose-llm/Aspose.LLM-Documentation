---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/simple-chat/
feedback: LLMNET
version: 26.5.0
title: Simple chat
description: Send messages to Aspose.LLM for .NET without managing sessions yourself — the engine creates and tracks a session implicitly.
keywords:
- simple
- chat
- SendMessageAsync
- current session
- one-off
---

For a quick question, a short exchange, or any scenario where you do not need multiple parallel conversations, use `SendMessageAsync` without managing sessions yourself. The engine creates and tracks a **current session** implicitly and uses it across calls.

## When to use this pattern

- Single question and single answer.
- A short back-and-forth where every message belongs to the same conversation.
- CLI tools, scripts, or test harnesses that run one conversation at a time.
- Prototypes where session management is not yet needed.

For concurrent conversations (e.g., multiple users) or scenarios that need explicit control over session IDs, use [Multi-turn chat](/net/use-cases/multi-turn-chat/) with `StartNewChatAsync` and `SendMessageToSessionAsync`.

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).
- RAM for the chosen preset — see [System requirements](/net/system-requirements/).

## Minimal example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(preset);

string reply = await api.SendMessageAsync("What is 2 + 2?");
Console.WriteLine(reply);
// Output: 2 + 2 equals 4.
```

Each call to `SendMessageAsync` without a session ID uses or creates the current session. Sequential calls form one coherent conversation:

```csharp
await api.SendMessageAsync("My name is Alice.");
string reply = await api.SendMessageAsync("What is my name?");
Console.WriteLine(reply);
// Output: Your name is Alice.
```

## Override the preset for one message

Pass a different preset to a single call without changing the default:

```csharp
using Aspose.LLM.Abstractions.Parameters.Presets;

// Default preset used by the API
var defaultPreset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(defaultPreset);

// Use a more deterministic preset for this one question
var strictPreset = new Qwen25Preset();
strictPreset.SamplerParameters.Temperature = 0.1f;

string reply = await api.SendMessageAsync(
    "What is the square root of 144?",
    preset: strictPreset);

Console.WriteLine(reply);
// Output: The square root of 144 is 12.
```

The override applies only to the message. Subsequent calls without a `preset` argument continue to use `defaultPreset`.

## Cancellation

Pass a `CancellationToken` to abort long generations:

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(30));

try
{
    string reply = await api.SendMessageAsync(
        "Write a 500-word essay about migration patterns of the Arctic tern.",
        cancellationToken: cts.Token);
    Console.WriteLine(reply);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Generation timed out.");
}
```

The `CancellationToken` stops token generation. The partial output is discarded; the session state is not corrupted, so you can continue the conversation with the next call.

## Full runnable example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

internal class SimpleChatDemo
{
    public static async Task Main()
    {
        var license = new Aspose.LLM.License();
        license.SetLicense("Aspose.LLM.lic");

        var preset = new Qwen25Preset();
        using var api = AsposeLLMApi.Create(preset);

        Console.WriteLine("Ask me anything. Type 'quit' to exit.");
        while (true)
        {
            Console.Write("> ");
            string? line = Console.ReadLine();
            if (line == null || line.Equals("quit", StringComparison.OrdinalIgnoreCase))
                break;

            try
            {
                string reply = await api.SendMessageAsync(line);
                Console.WriteLine(reply);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error: {ex.Message}");
            }
        }
    }
}
```

Run:

```bash
dotnet run
```

## Common errors

- **`Not licensed for this method`** — apply a license before calling any chat method. See [Licensing](/net/licensing/).
- **`Only one AsposeLLMApi instance can be created at a time`** — an earlier instance has not been disposed. Keep one instance for the process lifetime.
- **Slow first response** — the engine downloads native binaries and the model file on first `Create`. Expect 5-15 minutes on a fresh machine. Subsequent runs use the local cache.

## What's next

- [Multi-turn chat](/net/use-cases/multi-turn-chat/) — explicit sessions and multiple concurrent conversations.
- [Save and restore session](/net/use-cases/save-and-restore-session/) — persist the conversation across process restarts.
- [Custom preset](/net/use-cases/custom-preset/) — tailor a preset to your model and requirements.
- [Chat sessions reference](/net/developer-reference/chat-sessions/) — full semantics of `SendMessageAsync` and the current session.

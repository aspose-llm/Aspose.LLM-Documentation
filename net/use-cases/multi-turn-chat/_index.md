---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/multi-turn-chat/
feedback: LLMNET
version: 26.5.0
title: Multi-turn chat
description: Run an explicit multi-turn conversation in Aspose.LLM for .NET — manage sessions, handle multiple concurrent conversations, and trim the KV cache when it grows.
keywords:
- multi-turn
- chat
- session
- StartNewChatAsync
- SendMessageToSessionAsync
- concurrent sessions
- cache cleanup
---

When you need a sustained conversation and want explicit control over it — for example, one session per user, per request, or per workflow stage — start sessions explicitly with `StartNewChatAsync` and address them by ID.

## When to use this pattern

- Multiple users, each with their own conversation on a single process.
- Multi-stage workflows where each stage has its own dialogue history.
- Long conversations where you need to clean up the KV cache manually.
- Any scenario where the current-session shortcut from [Simple chat](/net/use-cases/simple-chat/) is not enough.

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).
- A working `AsposeLLMApi` instance (see [Hello, world!](/net/hello-world/)).

## Start a session

```csharp
string sessionId = await api.StartNewChatAsync();
```

Optional overrides:

```csharp
string sessionId = await api.StartNewChatAsync(
    preset: myPreset,               // use a different preset for this session
    sessionId: "user-42-conv-1");   // your own identifier; uniqueness is your responsibility
```

## Send messages to the session

Use `SendMessageToSessionAsync` with the session ID. The model sees the full history of the session and answers in context.

```csharp
string reply1 = await api.SendMessageToSessionAsync(sessionId, "I want to plan a trip to Lisbon in May.");
string reply2 = await api.SendMessageToSessionAsync(sessionId, "Suggest three neighborhoods to stay in.");
string reply3 = await api.SendMessageToSessionAsync(sessionId, "Which one is closest to the beach?");
```

Every message adds to the session's KV cache. The third reply can reference the first two turns because the earlier context is preserved.

## Full example — single session

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();
preset.ChatParameters.SystemPrompt = "You are a concise travel assistant. Answer in one or two sentences.";

using var api = AsposeLLMApi.Create(preset);

string sessionId = await api.StartNewChatAsync();

string[] userMessages =
{
    "I want to plan a trip to Lisbon in May.",
    "Suggest three neighborhoods to stay in.",
    "Which one is closest to the beach?",
    "How long does it take to walk from there to the historic center?",
    "Any recommendations for lunch on the way?",
};

foreach (string message in userMessages)
{
    Console.WriteLine($"> {message}");
    string reply = await api.SendMessageToSessionAsync(sessionId, message);
    Console.WriteLine(reply);
    Console.WriteLine();
}
```

Each follow-up sees the previous turns, so the model can track the topic across the whole exchange.

## Multiple concurrent sessions

A single `AsposeLLMApi` instance can host many sessions at once. Each session keeps its own history and KV region.

```csharp
string sessionA = await api.StartNewChatAsync(sessionId: "conv-A");
string sessionB = await api.StartNewChatAsync(sessionId: "conv-B");

await api.SendMessageToSessionAsync(sessionA, "Let us talk about ancient Rome.");
await api.SendMessageToSessionAsync(sessionB, "Let us talk about modern cooking.");

string replyA = await api.SendMessageToSessionAsync(sessionA, "Name three emperors.");
string replyB = await api.SendMessageToSessionAsync(sessionB, "Name three essential knives.");

Console.WriteLine($"A: {replyA}");
Console.WriteLine($"B: {replyB}");
```

The topics do not mix — `sessionA` sees only the Rome thread, `sessionB` only the cooking thread.

{{% alert color="primary" %}}
Do not call `SendMessageToSessionAsync` concurrently on the **same** session ID. Serialize your calls per session. Across sessions, serialize at the application level — the native model and KV pool are shared and a single inference call holds native resources.
{{% /alert %}}

## Manage KV cache in long sessions

As a session grows, its KV cache approaches the preset's `ContextParameters.ContextSize`. When the next message would overflow, the engine trims automatically according to `ChatParameters.CacheCleanupStrategy` (default `RemoveOldestMessages`).

You can also trim explicitly — for example, at a natural topic boundary in your application:

```csharp
api.ForceCacheCleanup(CacheCleanupStrategy.KeepSystemPromptOnly);
```

This operates on the **current session**. To trim a different session, set it as current first (e.g., by calling `SendMessageToSessionAsync` once on that session, which does not change `ChatParameters` but keeps the session warm), or use the default strategy and let the engine trim when needed.

See [Chat sessions — Manage the KV cache](/net/developer-reference/chat-sessions/#manage-the-kv-cache) for all five strategies and how to pick one.

## Pick a session ID strategy

- **Generated IDs** — let the engine choose (`StartNewChatAsync()` with no `sessionId`). Safe for single-process scenarios where you store the returned ID in your code or database.
- **Your own IDs** — pass a meaningful ID like `"user-42-conv-1"`. Makes logs and persisted session files easier to inspect. You are responsible for uniqueness within the process.

## Common errors

- **`Not licensed for this method`** — apply a license before starting sessions.
- **Context overflow before trim** — if you see nonsensical output in a long session, the model may have lost track of the system prompt. Force an explicit `ForceCacheCleanup(KeepSystemPromptAndFirstUserMessage)` at topic boundaries.
- **Mixing sessions** — double-check you pass the right `sessionId` to `SendMessageToSessionAsync`. Using `SendMessageAsync` in a multi-session app sends to the current session, which may not be the one you expect.

## What's next

- [Save and restore session](/net/use-cases/save-and-restore-session/) — persist a long conversation across runs.
- [Custom preset](/net/use-cases/custom-preset/) — tune the preset for your scenario (system prompt, sampler, context size).
- [Chat sessions reference](/net/developer-reference/chat-sessions/) — full semantics of session creation, messaging, and cache management.

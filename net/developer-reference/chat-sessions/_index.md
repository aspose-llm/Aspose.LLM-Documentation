---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/chat-sessions/
feedback: LLMNET
version: 26.5.0
title: Chat sessions
description: Create chat sessions, send messages, manage the current session, trim the KV cache, and understand concurrency constraints in Aspose.LLM for .NET.
keywords:
- chat
- session
- StartNewChatAsync
- SendMessageAsync
- SendMessageToSessionAsync
- ForceCacheCleanup
- KV cache
- current session
---

Aspose.LLM for .NET groups conversations into **chat sessions**. Each session has a unique identifier, an independent message history, and its own slice of the KV cache. A single `AsposeLLMApi` instance can host many sessions concurrently, one per conversation you track.

## Start a session

Call `StartNewChatAsync` to create a new chat session:

```csharp
string sessionId = await api.StartNewChatAsync();
```

Optional parameters let you override the preset or provide a custom session ID:

```csharp
string sessionId = await api.StartNewChatAsync(
    preset: myPreset,           // override the default preset for this session
    sessionId: "user-42-conv-1" // your own identifier; otherwise one is generated
);
```

The returned identifier targets the session in later calls. If you pass your own `sessionId`, keep it unique across active sessions in this process.

When you start a new session, it becomes the **current session** for subsequent `SendMessageAsync` calls that do not specify a session ID.

## Send a message

### To the current session

`SendMessageAsync` sends to the current session. If there is no current session, the API starts one implicitly using the default preset.

```csharp
string response = await api.SendMessageAsync("Your message here.");
```

Optional parameters:

```csharp
string response = await api.SendMessageAsync(
    "Describe this image.",
    media: new[] { imageBytes },            // for vision presets
    preset: visionPreset,                   // override the default preset
    cancellationToken: cancellationToken);
```

### To a specific session

`SendMessageToSessionAsync` targets a session by ID. Use it when you manage multiple concurrent sessions.

```csharp
string response = await api.SendMessageToSessionAsync(sessionId, "Follow-up question.");
```

Optional `media` and `cancellationToken` parameters work the same way as on `SendMessageAsync`.

## Signature reference

```csharp
public Task<string> StartNewChatAsync(
    PresetCoreBase? preset = null,
    string? sessionId = null);

public Task<string> SendMessageAsync(
    string message,
    IEnumerable<byte[]>? media = null,
    PresetCoreBase? preset = null,
    CancellationToken cancellationToken = default);

public Task<string> SendMessageToSessionAsync(
    string sessionId,
    string message,
    IEnumerable<byte[]>? media = null,
    CancellationToken cancellationToken = default);
```

## Current session

The engine tracks a **current session** that `SendMessageAsync` uses when you do not specify a session ID. Two things set the current session:

- **`StartNewChatAsync`** — the newly created session becomes current.
- **`LoadChatSession(filePath)`** — the loaded session becomes current (see [Session persistence](/net/developer-reference/session-persistence/)).

`SendMessageToSessionAsync(sessionId, ...)` targets the specified session without changing the current one.

If you manage multiple users, threads, or conversations, prefer `SendMessageToSessionAsync` with explicit IDs. The current-session shortcut is convenient for simple apps with one active conversation at a time.

## Manage the KV cache

As a conversation grows, the KV cache that holds every past token approaches the preset's `ContextParameters.ContextSize`. You can trim it explicitly with `ForceCacheCleanup`:

```csharp
api.ForceCacheCleanup(CacheCleanupStrategy.KeepSystemPromptOnly);
```

This operates on the **current session**. It throws `InvalidOperationException` if no session is active.

`CacheCleanupStrategy` offers five modes (full details in the cache management reference, which is on the roadmap for Phase 3):

| Strategy | Keeps in cache | When to use |
|---|---|---|
| `RemoveOldestMessages` (default) | System prompt + most recent turns | General-purpose; default for automatic trimming. |
| `KeepSystemPromptOnly` | System prompt only | Hard reset; use before starting a new topic in the same session. |
| `KeepSystemPromptAndHalf` | System prompt + last half of the history | Balanced recall and room for new tokens. |
| `KeepSystemPromptAndFirstUserMessage` | System prompt + first user turn | Recall-heavy tasks where the original ask matters. |
| `KeepSystemPromptAndLastUserMessage` | System prompt + most recent user turn | Focus on the latest question, drop middle context. |

Cache cleanup also happens automatically when a session would otherwise overflow the context during generation.

## Multiple sessions

Create several sessions on a single `AsposeLLMApi` instance and target them by ID:

```csharp
string sessionA = await api.StartNewChatAsync(sessionId: "conv-A");
string sessionB = await api.StartNewChatAsync(sessionId: "conv-B");

await api.SendMessageToSessionAsync(sessionA, "Topic A question.");
await api.SendMessageToSessionAsync(sessionB, "Topic B question.");
```

Each session keeps its own history and KV region. The loaded model is shared across all sessions.

## Concurrency and thread safety

- **One `AsposeLLMApi` instance per process.** `AsposeLLMApi.Create` enforces this via `Interlocked.CompareExchange`. A second call while the first instance is alive throws `InvalidOperationException`.
- **Do not call inference concurrently on the same session.** Serialize `SendMessageToSessionAsync(sessionId, ...)` for a given `sessionId`.
- **Serialize inference across sessions** at the application level. The native model and KV pool are shared, and a single inference call holds native resources. If you need concurrent request handling, queue requests or route them through a single worker.

See [Architecture — Threading and concurrency](/net/product-overview/architecture/#threading-and-concurrency) for more.

## Lifecycle

A session remains alive until:

- The `AsposeLLMApi` instance is disposed (all sessions are released).
- Or it is replaced by a `LoadChatSession(filePath)` call with the same session ID (the old session is disposed).

There is no explicit "delete session" API in the current version. For very long-lived applications with many ephemeral sessions, dispose and recreate the `AsposeLLMApi` periodically.

## What's next

- [Session persistence](/net/developer-reference/session-persistence/) — save and load sessions to disk.
- [Presets](/net/developer-reference/presets/) — configure parameter bags that affect session behavior.
- [Multi-turn chat](/net/use-cases/multi-turn-chat/) — full use case with multiple messages in one session.

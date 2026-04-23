---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/chat-sessions/chat-history/
feedback: LLMNET
version: 26.5.0
title: Chat history structure
description: ChatMessage type reference in Aspose.LLM for .NET — roles, media attachments, and KV cache tracking fields used to locate and evict messages from the native KV region.
keywords:
- ChatMessage
- chat history
- KVSize
- KVStartPosition
- KVStatus
- role
- conversation history
---

Every chat session keeps its conversation history as a list of `ChatMessage` objects. Each message records the role, the content, any attached media, and KV-cache tracking metadata that lets the engine locate and evict the message from the native KV region.

## `ChatMessage` reference

```csharp
namespace Aspose.LLM.Abstractions.Models;

public class ChatMessage
{
    public int Id { get; set; }
    public string Role { get; set; }
    public string Content { get; set; }
    public List<MediaAttachment> Media { get; set; }

    // KV cache tracking
    public int KVSize { get; set; }           // default -1
    public int KVStartPosition { get; set; }  // default -1
    public int KVStatus { get; set; }         // default 1

    // Ctors
    public ChatMessage();
    public ChatMessage(string content, string role, params MediaAttachment[] media);

    // Factory methods
    public static ChatMessage CreateUserMessage(string content, params MediaAttachment[] media);
    public static ChatMessage CreateAssistantMessage(string content);
    public static ChatMessage CreateSystemMessage(string content);

    // Convenience
    public bool HasMedia { get; }
    public long TotalMediaSize { get; }
    public void Validate();
    public ChatMessage ToTextOnly();
    public override string ToString();
}
```

## Roles

`Role` is a plain string. Three canonical values:

| Role | Produced by | Purpose |
|---|---|---|
| `"system"` | `CreateSystemMessage` | Role and format instructions applied at session start. |
| `"user"` | `CreateUserMessage` | The human side of the dialogue. |
| `"assistant"` | `CreateAssistantMessage` | The model's previous replies. |

Use the factory methods rather than constructing `ChatMessage` manually — they set `Role` correctly and accept media as params.

## Media attachments

`Media` is a `List<MediaAttachment>`. For text-only presets, leave empty. For vision presets, attach image bytes:

```csharp
var attachment = MediaAttachment.FromBytes(File.ReadAllBytes("photo.jpg"));
var msg = ChatMessage.CreateUserMessage("Describe this image.", attachment);
```

See [Attaching images](/net/developer-reference/multimodal/attaching-images/) for details on `MediaAttachment`.

Helpers:

- `HasMedia` — `true` when the list has at least one attachment.
- `TotalMediaSize` — sum of attachment byte lengths.

## KV cache tracking fields

Three integer fields track how the message sits in the native KV cache. You do not set these yourself — the engine manages them. They are visible because session persistence serializes them, and the cache-cleanup policy uses them to decide what to evict.

| Field | Default | Meaning |
|---|---:|---|
| `KVSize` | `-1` | Number of tokens this message occupies in the KV cache. `-1` means not yet committed. |
| `KVStartPosition` | `-1` | Starting token index in the KV cache. `-1` means not yet placed. |
| `KVStatus` | `1` | `1` = loaded; `0` = marked for cleanup; `-1` = unloaded. |

The cleanup strategy in [`CacheCleanupStrategy`](/net/developer-reference/cache-management/) walks these fields to decide what to evict. A `KVStatus` of `0` on a message means "this has been flagged for removal but not yet freed"; `-1` means the tokens have been physically removed from the native cache.

When you save a session via `SaveChatSession`, these values are preserved so that `LoadChatSession` can resume where the run left off without re-tokenizing the entire history.

## Validation

`ChatMessage.Validate` enforces:

- Non-empty text content **or** at least one media attachment.
- Each attached `MediaAttachment` passes its own validation (non-empty data, ≤50 MB, supported format).

Throws `InvalidOperationException` on failure. The engine calls `Validate` automatically before sending a message; you can call it earlier in your application to fail-fast.

## Text-only copies

`ToTextOnly` returns a copy without media — useful for logging or display where you need the text but want to discard image bytes.

```csharp
var textCopy = message.ToTextOnly();
_logger.LogInformation("User said: {Text}", textCopy.Content);
```

## Pre-filling history

You can seed a preset's history so new sessions start with pre-existing turns. The engine tokenizes these when the session is created, so they count toward `ContextSize`.

```csharp
using Aspose.LLM.Abstractions.Models;

preset.ChatParameters.History = new List<ChatMessage>
{
    ChatMessage.CreateSystemMessage("You are a translation assistant."),
    ChatMessage.CreateUserMessage("Translate to French: The sky is blue."),
    ChatMessage.CreateAssistantMessage("Le ciel est bleu."),
};
```

Every new session created from this preset starts with these three turns. This is one common way to implement few-shot prompting.

## Inspecting a live session's history

The runtime history of an active session is accessible via the `IChatSession` interface when you reach the engine directly through DI. From the facade (`AsposeLLMApi`), history inspection is not exposed in the current version — use session persistence (`SaveChatSession`) to snapshot a session when you need to examine it.

## What's next

- [Chat sessions](/net/developer-reference/chat-sessions/) — session lifecycle.
- [Attaching images](/net/developer-reference/multimodal/attaching-images/) — `MediaAttachment` details.
- [Cache management](/net/developer-reference/cache-management/) — how the `KV*` fields drive eviction.
- [Session persistence](/net/developer-reference/session-persistence/) — save/load and the nuance around restored sessions.

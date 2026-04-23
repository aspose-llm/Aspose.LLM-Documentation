---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/session-persistence/
feedback: LLMNET
version: 26.5.0
title: Session persistence
description: Save and load chat sessions with Aspose.LLM for .NET — file format, what is preserved, and parameter behavior on load.
keywords:
- save
- load
- session
- SaveChatSession
- LoadChatSession
- persistence
- JSON
---

Aspose.LLM for .NET can serialize an alive chat session to disk and restore it later. This lets you pause a conversation, ship session state between machines, or back up important dialogues.

## Signature reference

```csharp
public void SaveChatSession(string sessionId, string? filePath = null);
public Task<string> LoadChatSession(string filePath);
```

## Save a session

Call `SaveChatSession` on the `AsposeLLMApi` instance. Pass the session ID and an optional file path.

```csharp
api.SaveChatSession(sessionId, "session-42.json");
```

If you omit `filePath`, the library uses the session ID as the file name in the current working directory. Pass a full path for deterministic placement:

```csharp
api.SaveChatSession(sessionId, @"C:\chat-backups\user-42-conv-1.json");
```

The call is **synchronous** — it blocks until the file is written.

Throws `KeyNotFoundException` when the `sessionId` is not an active session on the instance.

### What is saved

The serialized file contains:

- The session identifier.
- The full chat history (every `ChatMessage` with role, content, and media metadata).
- KV cache positions and sizes per message (`KVSize`, `KVStartPosition`, `KVStatus`), so the restored session can continue where the saved one left off.

The file format is **JSON**. The schema is internal and may change between major SDK versions. Do not parse or edit these files by hand.

## Load a session

`LoadChatSession` reads a file, creates a new `ChatSession` from it, and returns the session ID.

```csharp
string sessionId = await api.LoadChatSession("session-42.json");
string reply = await api.SendMessageToSessionAsync(sessionId, "What did we discuss?");
```

Throws `FileNotFoundException` when the path does not exist. Throws `InvalidOperationException` when the file cannot be deserialized (unsupported format or SDK version mismatch).

The loaded session becomes the **current session** for subsequent `SendMessageAsync` calls that do not specify a session ID.

If a session with the same ID already lives on the instance, it is disposed and replaced by the loaded one. A log entry is emitted at `Information` level when this happens.

### Parameter behavior on load

`LoadChatSession` creates the new session with **default** `ContextParameters`, `ChatParameters`, and `SamplerParameters` — not with the values from the preset or from the original session's parameters.

If you need the restored session to use the same parameters as the preset you use in the rest of your code, start a new session explicitly instead of loading, and re-play the history from the saved file against that session. A helper for preset-aware loading is on the roadmap.

## Typical workflow

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

// Session 1: pause and save.
{
    var license = new Aspose.LLM.License();
    license.SetLicense("Aspose.LLM.lic");

    using var api = AsposeLLMApi.Create(new Qwen25Preset());
    string sessionId = await api.StartNewChatAsync();
    await api.SendMessageToSessionAsync(sessionId, "Plan a trip to Madrid in May.");
    await api.SendMessageToSessionAsync(sessionId, "Three museums worth visiting?");
    api.SaveChatSession(sessionId, "madrid-trip.json");
}

// Session 2: restore and continue.
{
    var license = new Aspose.LLM.License();
    license.SetLicense("Aspose.LLM.lic");

    using var api = AsposeLLMApi.Create(new Qwen25Preset());
    string sessionId = await api.LoadChatSession("madrid-trip.json");
    string answer = await api.SendMessageToSessionAsync(sessionId, "Which one has the shortest queues?");
    Console.WriteLine(answer);
}
```

## Portability between machines

A session file can move between machines under these conditions:

- **Same SDK version.** Both sides use the same major and minor Aspose.LLM version. Format changes only between major versions, but be safe and match.
- **Same model.** Load on a machine running the same model (same Hugging Face repo, file name, and quantization). A different model produces garbled output because KV cache positions are tied to the original tokenization.
- **Same `ReleaseTag`.** Match `BinaryManagerParameters.ReleaseTag` on both sides. Different `llama.cpp` runtime versions can interpret the KV cache differently.

For long-term archives, store the `.json` file together with a small manifest recording the SDK version, preset type, model source, and `ReleaseTag` used at save time.

## Use cases

- **Resumable chat.** Save on app exit, load on next launch, continue where the user left off.
- **Backup.** Periodically save important conversations to durable storage.
- **Hand-off.** Move state between development and production runs, or between geographically distributed instances.

## Limitations

- No incremental save. Each call writes the whole session.
- No built-in encryption. Wrap the file in your own encryption layer for sensitive dialogues.
- No automatic cleanup of old files. Your application is responsible for retention.
- Loading on a different SDK version or with a different model produces undefined behavior — often garbled output or a deserialization exception.

## What's next

- [Chat sessions](/net/developer-reference/chat-sessions/) — create and use sessions at runtime.
- [Save and restore session](/net/use-cases/save-and-restore-session/) — a complete runnable example.
- [Presets](/net/developer-reference/presets/) — configure the parameters that apply to new sessions.

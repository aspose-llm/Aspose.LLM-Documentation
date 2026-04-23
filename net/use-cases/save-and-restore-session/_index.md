---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/save-and-restore-session/
feedback: LLMNET
version: 26.5.0
title: Save and restore session
description: Persist an Aspose.LLM for .NET chat session to disk and resume it in a later process or on another machine.
keywords:
- save
- restore
- session
- persistence
- SaveChatSession
- LoadChatSession
---

Aspose.LLM for .NET can save a chat session to disk and restore it later. This lets you pause a conversation, survive application restarts, or migrate dialogue state between machines.

## When to use this pattern

- A desktop or server application whose users expect the chat to persist between launches.
- A long-running workflow where the conversation needs to outlive the process.
- Backup and audit — snapshot an active conversation at a known point.
- Migrating session state between machines that run the same SDK version and the same preset.

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).
- A working `AsposeLLMApi` instance.

## Save a session

After exchanging messages, call `SaveChatSession`:

```csharp
api.SaveChatSession(sessionId, "session-42.json");
```

If you omit `filePath`, the session ID is used as the file name in the current working directory:

```csharp
api.SaveChatSession(sessionId); // writes <sessionId>.json next to the executable
```

A full path gives deterministic placement:

```csharp
string path = Path.Combine(Path.GetTempPath(), "chats", $"{sessionId}.json");
Directory.CreateDirectory(Path.GetDirectoryName(path)!);
api.SaveChatSession(sessionId, path);
```

The call is synchronous and writes the entire session state as JSON.

## Restore a session

`LoadChatSession` reads a file and returns the session ID of the restored session. The loaded session becomes the current session — you can use `SendMessageAsync` immediately without passing the ID.

```csharp
string sessionId = await api.LoadChatSession("session-42.json");
string reply = await api.SendMessageToSessionAsync(sessionId, "What did we discuss?");
```

## Full example — save, restart, resume

This example saves after a short conversation, then — in a different `AsposeLLMApi` instance — reloads and continues.

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

// ---------- First run: save ----------
{
    var license = new Aspose.LLM.License();
    license.SetLicense("Aspose.LLM.lic");

    using var api = AsposeLLMApi.Create(new Qwen25Preset());

    string sessionId = await api.StartNewChatAsync(sessionId: "support-ticket-1234");

    await api.SendMessageToSessionAsync(sessionId,
        "Customer reports that the migration from v25 to v26 broke their startup script.");
    await api.SendMessageToSessionAsync(sessionId,
        "Their environment: Windows Server 2022, .NET 8, CUDA 12.6.");
    await api.SendMessageToSessionAsync(sessionId,
        "What questions should I ask them next?");

    api.SaveChatSession(sessionId, "support-ticket-1234.json");
    Console.WriteLine("Session saved.");
}

// ---------- Second run: resume ----------
{
    var license = new Aspose.LLM.License();
    license.SetLicense("Aspose.LLM.lic");

    using var api = AsposeLLMApi.Create(new Qwen25Preset());

    string sessionId = await api.LoadChatSession("support-ticket-1234.json");
    Console.WriteLine($"Resumed session: {sessionId}");

    string reply = await api.SendMessageToSessionAsync(sessionId,
        "They replied that they use a custom build step that copies native DLLs. How should I proceed?");
    Console.WriteLine(reply);
}
```

The second run picks up the context of the first — the model remembers the customer's scenario and your earlier diagnostic steps.

## What is saved

The JSON file contains:

- The session identifier.
- The full message history (role, content, any media metadata).
- KV cache positions and sizes per message so the restored session continues exactly where you paused.

The format is internal and may change between major SDK versions. Do not parse or edit saved files manually.

## Portability constraints

A saved file can move between processes or machines under these conditions:

- **Same SDK major version.** Format compatibility is not guaranteed across major versions.
- **Same model file.** Load on a machine that resolves the same Hugging Face model and quantization. A different model produces garbled output because KV cache positions are tied to the original tokenization.
- **Same `BinaryManagerParameters.ReleaseTag`.** Different `llama.cpp` runtime versions can interpret the KV cache differently.

For archival, store alongside the `.json` file a small manifest with the SDK version, preset type, model source, and release tag you used at save time. That metadata is how you decide whether an old file is still safe to load.

## A known load-time nuance

`LoadChatSession` creates the restored session with **default** `ContextParameters`, `ChatParameters`, and `SamplerParameters` — not with the preset's values or the values that were in effect at save time.

Practical impact:

- The session continues with the model already loaded by the current `AsposeLLMApi` instance — that is correct.
- But the **sampler temperature**, **max tokens**, **context size knobs**, and **system prompt** revert to library defaults.

If your application depends on specific sampler settings or a system prompt, re-apply them to the session's parameters after loading, or start a fresh session and replay the saved history against it.

## Common errors

- **`FileNotFoundException`** — check the path. Relative paths are resolved against the current working directory.
- **`InvalidOperationException` on load** — the file is from an incompatible SDK version, or the JSON is corrupted.
- **Garbled output after load** — the model or `ReleaseTag` does not match the one used at save time.

## Security

The saved file is plain JSON. It contains every user and assistant message verbatim. Wrap the file in your own encryption layer (e.g., DPAPI on Windows, or a cross-platform library) before writing it to untrusted storage.

## What's next

- [Multi-turn chat](/net/use-cases/multi-turn-chat/) — manage sessions at runtime.
- [Custom preset](/net/use-cases/custom-preset/) — configure the preset used when the session is first created.
- [Session persistence reference](/net/developer-reference/session-persistence/) — full semantics of save and load.

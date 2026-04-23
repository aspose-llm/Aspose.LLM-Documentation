---
weight: 5
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/asposellmapi/
feedback: LLMNET
version: 26.5.0
title: AsposeLLMApi facade
description: Detailed reference for AsposeLLMApi — the single-instance facade class of Aspose.LLM for .NET. Covers construction, every public method, exception semantics, and lifecycle.
keywords:
- AsposeLLMApi
- facade
- Create
- Dispose
- StartNewChatAsync
- SendMessageAsync
- LoadChatSession
- single instance
---

`AsposeLLMApi` is the single public facade class of Aspose.LLM for .NET. Every chat operation in your application goes through it — model loading, session creation, message exchange, cache management, session persistence, and disposal.

This page covers the class surface in depth. For the short API-card summary, see [API reference](/net/developer-reference/api-reference/). For a compact hello-world, see [Hello, world!](/net/hello-world/).

## Class overview

```csharp
namespace Aspose.LLM;

public partial class AsposeLLMApi : IDisposable
{
    // Construction
    public AsposeLLMApi(PresetCoreBase preset, ILogger? logger = null);
    public static AsposeLLMApi Create(PresetCoreBase preset, ILogger? logger = null);

    // Default accessors
    public PresetCoreBase DefaultPreset { get; }
    public PresetCoreBase GetDefaultPreset();
    public Task<(ModelInferenceParameters, ContextParameters, ChatParameters, SamplerParameters)>
        GetDefaultParametersAsync();

    // Session lifecycle
    public Task<string> StartNewChatAsync(PresetCoreBase? preset = null, string? sessionId = null);
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

    // Persistence
    public void SaveChatSession(string sessionId, string? filePath = null);
    public Task<string> LoadChatSession(string filePath);

    // Cache
    public void ForceCacheCleanup(CacheCleanupStrategy strategy = CacheCleanupStrategy.KeepSystemPromptOnly);

    // Disposal
    public void Dispose();
}
```

## Single instance per process

The class enforces a hard single-instance guard via `Interlocked.CompareExchange`. A second `Create` (or `new AsposeLLMApi(...)`) call while a previous instance is alive throws:

```
System.InvalidOperationException: Only one AsposeLLMApi instance can be created at a time.
```

Dispose the current instance (or let the `using` block exit) before creating a new one. There is no supported way to run two instances simultaneously in the same process.

## Construction

Two equivalent ways to create the instance:

```csharp
// Via constructor:
var api = new AsposeLLMApi(preset, logger);

// Via static factory (idiomatic):
var api = AsposeLLMApi.Create(preset, logger);
```

Both accept the same arguments:

| Argument | Type | Required | Purpose |
|---|---|---|---|
| `preset` | `PresetCoreBase` | yes | Default preset for chat operations. Throws `ArgumentNullException` on null. |
| `logger` | `ILogger?` | no | Optional logger for engine + native debug output. |

The preset is applied once at construction. Later mutations to the preset have no effect — the engine has already read it. See [Presets](/net/developer-reference/presets/) for the override-before-Create pattern.

Construction **synchronously blocks** on three stages:

1. Single-instance guard.
2. Native binary deployment (downloads from GitHub on first run — 100-500 MB).
3. Model load (downloads from Hugging Face on first run — 2-15 GB).

On a cold machine, `Create` can take several minutes. Subsequent runs use the local caches. See [Architecture](/net/product-overview/architecture/#what-happens-when-you-create-the-api) for the full flow.

## Default accessors

### `DefaultPreset`

Returns the preset passed to `Create`. Useful for inspecting the current configuration at runtime.

```csharp
PresetCoreBase current = api.DefaultPreset;
int contextSize = current.ContextParameters.ContextSize ?? 0;
```

Read-only — set via constructor.

### `GetDefaultPreset()`

Returns a **fresh** `Qwen25Preset` instance. Convenience helper for code that needs a sensible preset without a concrete choice. Not a snapshot of `DefaultPreset`.

```csharp
PresetCoreBase fallback = api.GetDefaultPreset(); // new Qwen25Preset()
```

### `GetDefaultParametersAsync()`

Returns the engine's default parameter values as a tuple — inference, context, chat, sampler.

```csharp
var (inference, context, chat, sampler) = await api.GetDefaultParametersAsync();
Console.WriteLine($"Default context size: {context.ContextSize}");
Console.WriteLine($"Default temperature: {sampler.Temperature}");
```

## Session lifecycle

### `StartNewChatAsync`

Creates a new chat session and returns its identifier.

```csharp
public Task<string> StartNewChatAsync(PresetCoreBase? preset = null, string? sessionId = null);
```

Arguments:

- `preset` (optional) — override the preset for this session. When `null`, the API uses `DefaultPreset`.
- `sessionId` (optional) — your own identifier. When `null`, the engine generates one.

Returns the session identifier. The new session becomes the **current session** for subsequent `SendMessageAsync` calls that do not specify a session ID.

```csharp
string sessionId = await api.StartNewChatAsync();
string customId = await api.StartNewChatAsync(sessionId: "user-42-conv-1");
```

Throws `Exception("Not licensed for this method")` when no license is applied.

### `SendMessageAsync`

Sends a message to the current session, or creates one if none exists.

```csharp
public Task<string> SendMessageAsync(
    string message,
    IEnumerable<byte[]>? media = null,
    PresetCoreBase? preset = null,
    CancellationToken cancellationToken = default);
```

Arguments:

- `message` — the prompt text.
- `media` (optional) — image byte arrays for vision presets. Formats detected by magic bytes.
- `preset` (optional) — override the preset for this single call. Affects sampling, cache strategy, and max tokens for this message only.
- `cancellationToken` — cancel in-flight generation. The partial output is discarded; the session remains intact.

Returns the full assistant response as a single string — no streaming.

```csharp
string reply = await api.SendMessageAsync("Describe this image.", new[] { imageBytes });
```

### `SendMessageToSessionAsync`

Targets a specific session by ID. Does not change the current session pointer.

```csharp
public Task<string> SendMessageToSessionAsync(
    string sessionId,
    string message,
    IEnumerable<byte[]>? media = null,
    CancellationToken cancellationToken = default);
```

Preferred when you manage multiple sessions (multiple users, concurrent workflows).

## Persistence

### `SaveChatSession`

```csharp
public void SaveChatSession(string sessionId, string? filePath = null);
```

Serializes the session to disk as JSON. Synchronous.

- `sessionId` — must be an active session on this instance. Throws `KeyNotFoundException` otherwise.
- `filePath` (optional) — destination path. When `null`, uses the session ID as the file name in the current working directory.

### `LoadChatSession`

```csharp
public Task<string> LoadChatSession(string filePath);
```

Restores a session from a JSON file and returns its ID. The restored session becomes the current session.

Throws:

- `FileNotFoundException` — path does not exist.
- `InvalidOperationException` — file is corrupt or from an incompatible SDK version.

{{% alert color="primary" %}}
`LoadChatSession` restores the session with **default** `ContextParameters`, `ChatParameters`, and `SamplerParameters` — not the preset's values or the values in effect when the session was saved. See [Session persistence](/net/developer-reference/session-persistence/) for the full nuance.
{{% /alert %}}

## Cache management

### `ForceCacheCleanup`

Trims the KV cache of the current session according to the chosen strategy.

```csharp
public void ForceCacheCleanup(
    CacheCleanupStrategy strategy = CacheCleanupStrategy.KeepSystemPromptOnly);
```

Throws `InvalidOperationException` when no current session exists.

Five strategies — see [Cache management](/net/developer-reference/cache-management/) for the trade-offs.

## Disposal

`AsposeLLMApi` implements `IDisposable`. `Dispose` (or the end of a `using` block):

1. Unloads the model from native memory.
2. Releases all session KV caches.
3. Resets the single-instance guard so a new `AsposeLLMApi` can be created.

```csharp
using var api = AsposeLLMApi.Create(preset);
// ... operations
// api.Dispose() is called automatically when the block exits.
```

Calling methods after `Dispose` throws `ObjectDisposedException`.

## Exception reference

| Exception | Raised when |
|---|---|
| `ArgumentNullException` | `preset` is `null` during construction. |
| `InvalidOperationException` | Second `AsposeLLMApi` while one is already alive. |
| `Exception("Not licensed for this method")` | A chat method is called without a valid license. |
| `KeyNotFoundException` | `SaveChatSession` called with a non-existent session ID. |
| `FileNotFoundException` | `LoadChatSession` given a path that does not exist. |
| `InvalidOperationException` | `LoadChatSession` fails to deserialize; or `ForceCacheCleanup` with no active session. |
| `OperationCanceledException` | `cancellationToken` fires during generation. |
| `ObjectDisposedException` | Any method called after `Dispose`. |

## What's next

- [Chat sessions](/net/developer-reference/chat-sessions/) — session lifecycle and messaging semantics.
- [Session persistence](/net/developer-reference/session-persistence/) — save / load details.
- [Cache management](/net/developer-reference/cache-management/) — trimming strategies.
- [Presets](/net/developer-reference/presets/) — the preset passed at construction.
- [Dependency injection](/net/developer-reference/dependency-injection/) — the alternative `AddLlamaServices` path.

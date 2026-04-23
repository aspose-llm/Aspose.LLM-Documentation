---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/how-to/reduce-first-token-latency/
feedback: LLMNET
version: 26.5.0
title: Reduce first-token latency
description: Cut the time to first response in Aspose.LLM for .NET — warm up the engine, shorten system prompts, size batches correctly, and avoid cold starts.
keywords:
- first-token latency
- cold start
- warm up
- time to first token
- TTFT
---

"First-token latency" is the time between sending a message and starting to see output. It has two components:

1. **Cold-start** — binary download, model load, session creation. Happens once per `AsposeLLMApi` instance.
2. **Per-message** — prompt tokenization, KV cache prefill, first-token generation.

Both can be reduced.

## Warm up at startup

The first `AsposeLLMApi.Create` call is slow:

- First ever: downloads binaries + model (100-500 MB + 2-15 GB). Several minutes.
- Cached: model load only. 5-30 seconds.

Do it during application startup, not on the first user request.

```csharp
// At application startup:
var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();
preset.EngineParameters.EnableDebugLogging = false;

var api = AsposeLLMApi.Create(preset); // slow the first time
```

In ASP.NET Core, warm up from `ApplicationStarted`:

```csharp
app.Lifetime.ApplicationStarted.Register(() =>
{
    _ = app.Services.GetRequiredService<Engine>(); // triggers model load
});
```

In a Worker Service, do it inside `ExecuteAsync` before entering the main loop.

## Pre-create a session

Starting a session takes tens to hundreds of milliseconds. For a chat server that processes user requests, create a session before the first request arrives:

```csharp
string warmupSessionId = await api.StartNewChatAsync();
await api.SendMessageToSessionAsync(warmupSessionId, "ping");
// Keep this session; the engine is now fully warm.
```

## Shorten system prompts

Every new session tokenizes and evaluates the system prompt before the first user turn. A 500-token system prompt costs hundreds of milliseconds on CPU, tens on GPU. Keep system prompts short.

```csharp
// 50 tokens — fast first-turn.
preset.ChatParameters.SystemPrompt = "You are a concise assistant. Answer briefly.";

// 500 tokens of preamble — slow.
// preset.ChatParameters.SystemPrompt = "<long preamble with many instructions and examples>";
```

If you need extensive priming, use `ChatParameters.History` with a few-shot example set — the examples are tokenized once per session creation and cached across turns.

## Size `NBatch` correctly

`ContextParameters.NBatch` controls how many tokens the engine processes per `llama_decode` call during prompt ingestion. Larger `NBatch` is faster **as long as it fits in memory**.

```csharp
// Built-in presets typically set NBatch = 2048. For prompt-heavy scenarios:
preset.ContextParameters.NBatch = 4096;
preset.ContextParameters.NUbatch = 4096;
```

Too large, and you blow VRAM budgets; too small, and prompt prefill is slow. Tune on your hardware.

## Enable flash attention

Flash attention improves prefill time dramatically on long prompts:

```csharp
preset.ContextParameters.FlashAttentionMode = FlashAttentionType.Enabled;
```

Always enable when supported.

## Keep sessions alive

Reuse sessions across requests instead of creating a fresh one each time. Session creation costs prefill time; reusing amortizes it across turns.

In HTTP hosts, map user IDs to session IDs — see [Multiple concurrent sessions](/net/use-cases/multiple-concurrent-sessions/).

## Pre-populate binary and model caches

In offline or container deployments, download binaries and models on the build machine; ship them with your image. The runtime host skips the multi-minute initial download.

See [Offline deployment](/net/use-cases/offline-deployment/).

## Measure

```csharp
var totalSw = System.Diagnostics.Stopwatch.StartNew();
using var api = AsposeLLMApi.Create(preset);
Console.WriteLine($"Create: {totalSw.Elapsed}");

var firstSw = System.Diagnostics.Stopwatch.StartNew();
string reply = await api.SendMessageAsync("Say hello.");
Console.WriteLine($"First message: {firstSw.Elapsed}");

var secondSw = System.Diagnostics.Stopwatch.StartNew();
string reply2 = await api.SendMessageAsync("Say hello again.");
Console.WriteLine($"Second message: {secondSw.Elapsed}");
```

The second message is noticeably faster than the first — session is already warm.

## Typical numbers (modern GPU)

| Stage | Time |
|---|---|
| `Create` (cold, first ever) | 2-10 min (download + load) |
| `Create` (cached) | 5-30 s |
| `StartNewChatAsync` | 50-200 ms |
| First token after a 100-token prompt | 200-500 ms |
| Subsequent tokens | 10-20 ms each |

CPU numbers are roughly 5-10× higher for each stage.

## What's next

- [Architecture](/net/product-overview/architecture/) — what happens during `Create`.
- [Context parameters](/net/developer-reference/parameters/context/) — `NBatch`, flash attention.
- [Offline deployment](/net/use-cases/offline-deployment/) — skip the initial download at runtime.

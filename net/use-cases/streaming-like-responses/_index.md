---
weight: 70
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/streaming-like-responses/
feedback: LLMNET
version: 26.5.0
title: Streaming-like responses
description: Aspose.LLM for .NET does not expose token streaming. This page documents the limitation, the workarounds, and the patterns that deliver a responsive user experience without streaming.
keywords:
- streaming
- no streaming
- limitation
- chunked response
- perceived latency
---

Aspose.LLM for .NET does **not** expose token-by-token streaming in the current release. `SendMessageAsync` and `SendMessageToSessionAsync` return the complete response as a single string after generation finishes. This page explains the limitation and the patterns that achieve a responsive user experience without streaming.

## The limitation

Method signatures return `Task<string>` — a single result, not an async sequence:

```csharp
public Task<string> SendMessageAsync(...);
public Task<string> SendMessageToSessionAsync(...);
```

Neither `IAsyncEnumerable<string>` nor a streaming callback is available. The native `llama.cpp` runtime can produce tokens one at a time, but the SDK's public API wraps them into a full response before returning.

## Practical impact

For typical chat UIs, the user sees the reply **after** the model finishes generating. On a modern GPU:

- Short replies (64 tokens) — ~0.5-1 second.
- Medium replies (256 tokens) — ~2-3 seconds.
- Long replies (1024 tokens) — ~8-15 seconds.

On CPU:

- Short replies — 3-10 seconds.
- Long replies — 1-3 minutes.

If your UX requires tokens to appear as they are generated (typical chat-window feel), the SDK in its current form does not provide that directly.

## Workarounds

### 1. Cap `MaxTokens` for shorter replies

Reduce the worst case.

```csharp
preset.ChatParameters.MaxTokens = 256;
```

A 256-token response completes in under a second on GPU, which is within the threshold where users typically do not need streaming feedback.

### 2. Break long tasks into smaller turns

Instead of one request that takes 30 seconds, design the conversation as multiple short turns. Each turn returns in 1-3 seconds; the user sees progress.

```csharp
string outline = await api.SendMessageAsync("Outline a 500-word article on X.");
// Show outline to user

string section1 = await api.SendMessageAsync("Expand section 1 of that outline.");
// Show section 1

string section2 = await api.SendMessageAsync("Expand section 2.");
// Show section 2
```

### 3. Show "thinking" indicator

When the wait is unavoidable, communicate that work is in progress. A spinner, progress bar, or status text ("Analyzing your question…") is better than dead air.

For ASP.NET Core, return a 102 Processing response or emit server-sent events with status updates before the final reply.

### 4. Run ahead with speculative small model

Some applications run a small, fast model for an initial "draft" response while a larger model generates the final version. The small model's output streams immediately; the larger response replaces it when ready. Implement at the application level:

```csharp
// Fast preset for immediate feedback.
var fastPreset = new Phi4Preset();
// Slow, higher-quality preset for the real answer.
var slowPreset = new Qwen25Preset();

// ... two AsposeLLMApi instances via multi-process setup (one instance per process).
```

Only possible with two processes because of the single-instance constraint.

### 5. Queue + background job pattern for long tasks

For tasks that would take minutes (document summarization, batch analysis), respond immediately with a job ID and deliver the result via polling or webhook:

```csharp
// POST /jobs — enqueue, return job ID.
// GET  /jobs/{id} — poll status and result.
// POST /webhooks/llm — deliver when ready.
```

The user is not blocked on a single HTTP request. This is the standard long-running-job pattern for HTTP services.

## What future releases may offer

Token streaming is a commonly requested feature. A future SDK version may expose:

- `IAsyncEnumerable<string>` for streamed tokens.
- Event-based callback during generation.
- Server-sent events integration for ASP.NET Core.

Until then, design your UX around the full-response API.

## Comparison with alternatives

| Feature | Aspose.LLM for .NET | Hosted APIs (OpenAI, Anthropic) |
|---|---|---|
| Token streaming | Not exposed in current release | Yes |
| On-premise | Yes | No (hosted) |
| Model control | Full | Limited |
| Data privacy | Local | Sent to provider |
| Latency floor | Model-dependent | Network round-trip |

The main reason to use Aspose.LLM is on-premise execution with no data egress. If token streaming is a hard requirement and hosted APIs are acceptable, use a hosted provider. If on-premise is a hard requirement, plan the UX around the current full-response API.

## What's next

- [Architecture](/net/product-overview/architecture/) — what happens during generation.
- [Features](/net/product-overview/features/) — full capability list and limitations.
- [Integration with ASP.NET Core](/net/use-cases/integration-with-aspnet-core/) — queue / job patterns for responsive HTTP.

---
weight: 90
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/multiple-concurrent-sessions/
feedback: LLMNET
version: 26.5.0
title: Multiple concurrent sessions
description: Serve many users or workflows from a single Aspose.LLM for .NET instance — per-user sessions, serialized inference, request routing patterns.
keywords:
- concurrent sessions
- multi-user
- per-user
- session routing
- queue
- chat server
---

A single `AsposeLLMApi` instance hosts many chat sessions. Each session has its own history and KV region — conversations do not cross-contaminate. This is the foundation for chat servers, multi-user tools, and background-job dispatchers.

Inference itself is serialized: the native layer does not run multiple inference calls in parallel on one instance. You route requests through a queue or single worker.

## When to use this pattern

- A chat service with one session per user.
- A workflow engine where each job has its own conversation state.
- Multi-stage agents where each stage keeps its own session.
- A REPL that runs several parallel dialogues (review, brainstorm, Q&A).

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).
- Familiarity with [Chat sessions](/net/developer-reference/chat-sessions/).

## Per-user sessions

Assign each user a session when they first appear. Use your own ID so you can find it later:

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();
preset.ChatParameters.SystemPrompt = "You are a support agent. Be concise and friendly.";

using var api = AsposeLLMApi.Create(preset);

// Map userId -> sessionId in your own state store (in-memory, Redis, DB).
var sessions = new Dictionary<string, string>();

async Task<string> EnsureSession(string userId)
{
    if (!sessions.TryGetValue(userId, out var sessionId))
    {
        sessionId = await api.StartNewChatAsync(sessionId: $"user:{userId}");
        sessions[userId] = sessionId;
    }
    return sessionId;
}

async Task<string> Ask(string userId, string message)
{
    string sessionId = await EnsureSession(userId);
    return await api.SendMessageToSessionAsync(sessionId, message);
}

// Usage:
string reply = await Ask("alice", "I can't log in.");
Console.WriteLine(reply);
```

## Serialize inference with a queue

Concurrent requests to `SendMessageToSessionAsync` on one instance are not safe. Use a channel or work queue to serialize:

```csharp
using System.Threading.Channels;

var queue = Channel.CreateUnbounded<(string SessionId, string Message, TaskCompletionSource<string> Result)>();

// Worker task: serializes inference.
_ = Task.Run(async () =>
{
    await foreach (var item in queue.Reader.ReadAllAsync())
    {
        try
        {
            string reply = await api.SendMessageToSessionAsync(item.SessionId, item.Message);
            item.Result.SetResult(reply);
        }
        catch (Exception ex)
        {
            item.Result.SetException(ex);
        }
    }
});

// API for callers: enqueue and await.
async Task<string> EnqueueAsk(string sessionId, string message)
{
    var tcs = new TaskCompletionSource<string>(TaskCreationOptions.RunContinuationsAsynchronously);
    await queue.Writer.WriteAsync((sessionId, message, tcs));
    return await tcs.Task;
}

// Many concurrent callers, one worker processing requests in order.
await Task.WhenAll(
    EnqueueAsk("user:alice", "Hi there."),
    EnqueueAsk("user:bob",   "I have a question."),
    EnqueueAsk("user:carol", "Tell me about your return policy."));
```

Callers can be concurrent; the inference itself runs sequentially. Throughput is limited by the model's tokens-per-second, not by how many users you have.

## Separate sessions, separate preset overrides

If different users need different tuning (temperature, max tokens, system prompt), start each session with an overriding preset:

```csharp
var terseSysPrompt = "You are a terse assistant. Keep answers to one sentence.";
var warmSysPrompt = "You are a warm and encouraging assistant.";

var tersePreset = new Qwen25Preset();
tersePreset.ChatParameters.SystemPrompt = terseSysPrompt;

var warmPreset = new Qwen25Preset();
warmPreset.ChatParameters.SystemPrompt = warmSysPrompt;

string terseId = await api.StartNewChatAsync(preset: tersePreset, sessionId: "mode:terse");
string warmId  = await api.StartNewChatAsync(preset: warmPreset,  sessionId: "mode:warm");
```

The underlying model stays the same (one load), but each session uses its own chat parameters for system prompt and max tokens.

## Memory sizing

Each session contributes KV cache equal to its history length (up to `ContextParameters.ContextSize`). A conservative budget:

```
peak KV memory = max_concurrent_sessions × avg_history_tokens × per_token_kv_bytes
```

For 100 concurrent sessions averaging 4K tokens of history, with 32-layer model and F16 KV cache, peak KV is ~6-8 GB on top of model weights.

Keep session count bounded — evict idle sessions explicitly when your memory budget is tight:

```csharp
// Example: evict sessions idle for more than 30 minutes.
// (Requires application-side tracking of last use time.)
```

The SDK does not expose an explicit "delete session" API in the current release. To fully reclaim a session's memory, dispose and recreate the `AsposeLLMApi` periodically in long-running services.

## Full example — minimal chat server

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;
using System.Threading.Channels;

internal class ChatServer
{
    private record Job(string SessionId, string Message, TaskCompletionSource<string> Result);

    public static async Task Main()
    {
        var license = new Aspose.LLM.License();
        license.SetLicense("Aspose.LLM.lic");

        var preset = new Qwen25Preset();
        preset.ChatParameters.SystemPrompt = "You are a support agent. Be concise and friendly.";

        using var api = AsposeLLMApi.Create(preset);

        var queue = Channel.CreateUnbounded<Job>();
        var sessions = new Dictionary<string, string>();

        // Worker loop.
        var worker = Task.Run(async () =>
        {
            await foreach (var job in queue.Reader.ReadAllAsync())
            {
                try
                {
                    string reply = await api.SendMessageToSessionAsync(job.SessionId, job.Message);
                    job.Result.SetResult(reply);
                }
                catch (Exception ex)
                {
                    job.Result.SetException(ex);
                }
            }
        });

        // Start a few sessions and enqueue interleaved requests.
        async Task<string> GetSession(string user) =>
            sessions.TryGetValue(user, out var id)
                ? id
                : (sessions[user] = await api.StartNewChatAsync(sessionId: user));

        async Task<string> Ask(string user, string message)
        {
            string sid = await GetSession(user);
            var tcs = new TaskCompletionSource<string>(TaskCreationOptions.RunContinuationsAsynchronously);
            await queue.Writer.WriteAsync(new Job(sid, message, tcs));
            return await tcs.Task;
        }

        var replies = await Task.WhenAll(
            Ask("alice", "Hi, I cannot log in."),
            Ask("bob",   "Is there a refund policy?"),
            Ask("alice", "My password reset email never arrived."),
            Ask("bob",   "How long does the refund take?"));

        foreach (var r in replies) Console.WriteLine(r);

        queue.Writer.Complete();
        await worker;
    }
}
```

## What's next

- [Chat sessions](/net/developer-reference/chat-sessions/) — full session reference.
- [Integration with ASP.NET Core](/net/use-cases/integration-with-aspnet-core/) — putting this pattern behind HTTP.
- [Cache management](/net/developer-reference/cache-management/) — control session memory over time.

---
weight: 160
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/integration-with-aspnet-core/
feedback: LLMNET
version: 26.5.0
title: Integration with ASP.NET Core
description: Host Aspose.LLM for .NET behind ASP.NET Core — DI registration, routing, per-user sessions, and graceful shutdown.
keywords:
- ASP.NET Core
- Minimal API
- dependency injection
- HTTP chat
- web service
- host
---

ASP.NET Core is the idiomatic way to put Aspose.LLM for .NET behind HTTP. The SDK registers as a DI singleton; requests flow through a handler that resolves the shared `Engine` and routes to per-user sessions.

## When to use this pattern

- Chat backend for a web or mobile application.
- Internal microservice that serves LLM inference to other services.
- API gateway that aggregates LLM and other services.
- Long-running service that benefits from a single model load across many requests.

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).
- Familiarity with [Dependency injection](/net/developer-reference/dependency-injection/).

## Minimal API — single-user demo

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;
using Aspose.LLM.Core.DependencyInjection;
using Aspose.LLM.Core.Services;

var builder = WebApplication.CreateBuilder(args);

// Apply license at startup.
var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

// Register LLM services.
builder.Services.AddLlamaServices(new Qwen25Preset());

var app = builder.Build();

// Warm up the model on the first request (eager alternative: resolve Engine at startup).
app.MapPost("/chat", async (ChatRequest req, Engine engine, CancellationToken ct) =>
{
    // Start a session or reuse an existing one.
    string sessionId = req.SessionId ?? await engine.InitiateNewSession();
    string reply = await engine.GetChatSessionResponse(sessionId, req.Message, null, ct);
    return Results.Ok(new ChatResponse(sessionId, reply));
});

app.Run();

public record ChatRequest(string Message, string? SessionId = null);
public record ChatResponse(string SessionId, string Reply);
```

Start the host:

```bash
dotnet run
```

Send a request:

```bash
curl -X POST http://localhost:5000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello!"}'
```

The response includes a session ID; pass it back on subsequent calls to continue the conversation.

## Serialize inference behind a channel

Concurrent HTTP requests can arrive at the endpoint; native inference is serialized. Use a channel worker so HTTP threads do not block native code:

```csharp
using System.Threading.Channels;

public record InferenceJob(
    string SessionId,
    string Message,
    TaskCompletionSource<string> Result,
    CancellationToken Token);

public class InferenceQueue
{
    private readonly Channel<InferenceJob> _channel = Channel.CreateUnbounded<InferenceJob>();

    public ChannelWriter<InferenceJob> Writer => _channel.Writer;
    public ChannelReader<InferenceJob> Reader => _channel.Reader;
}

public class InferenceWorker : BackgroundService
{
    private readonly InferenceQueue _queue;
    private readonly Engine _engine;
    private readonly ILogger<InferenceWorker> _logger;

    public InferenceWorker(InferenceQueue queue, Engine engine, ILogger<InferenceWorker> logger)
    {
        _queue = queue;
        _engine = engine;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        await foreach (var job in _queue.Reader.ReadAllAsync(stoppingToken))
        {
            try
            {
                string reply = await _engine.GetChatSessionResponse(
                    job.SessionId, job.Message, null, job.Token);
                job.Result.SetResult(reply);
            }
            catch (OperationCanceledException oce)
            {
                job.Result.SetCanceled(oce.CancellationToken);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Inference failed for session {SessionId}", job.SessionId);
                job.Result.SetException(ex);
            }
        }
    }
}
```

Register and use:

```csharp
builder.Services.AddSingleton<InferenceQueue>();
builder.Services.AddHostedService<InferenceWorker>();

app.MapPost("/chat", async (
    ChatRequest req,
    Engine engine,
    InferenceQueue queue,
    CancellationToken ct) =>
{
    string sessionId = req.SessionId ?? await engine.InitiateNewSession();

    var tcs = new TaskCompletionSource<string>(TaskCreationOptions.RunContinuationsAsynchronously);
    await queue.Writer.WriteAsync(new InferenceJob(sessionId, req.Message, tcs, ct), ct);

    string reply = await tcs.Task;
    return Results.Ok(new ChatResponse(sessionId, reply));
});
```

HTTP threads enqueue and await; the worker processes jobs sequentially.

## Per-user sessions

Map external user identifiers to internal session IDs. A simple in-memory store:

```csharp
public class UserSessionMap
{
    private readonly Dictionary<string, string> _map = new();

    public string? Get(string userId) => _map.TryGetValue(userId, out var s) ? s : null;
    public void Set(string userId, string sessionId) => _map[userId] = sessionId;
}

builder.Services.AddSingleton<UserSessionMap>();

app.MapPost("/chat", async (
    ChatRequest req,
    Engine engine,
    UserSessionMap sessions,
    InferenceQueue queue,
    HttpContext ctx,
    CancellationToken ct) =>
{
    // Pull user ID from auth or header.
    string userId = ctx.Request.Headers["X-User-Id"].ToString();
    if (string.IsNullOrEmpty(userId)) return Results.BadRequest("X-User-Id required.");

    string sessionId = sessions.Get(userId)
        ?? await engine.InitiateNewSession(sessionId: $"user:{userId}");
    sessions.Set(userId, sessionId);

    var tcs = new TaskCompletionSource<string>(TaskCreationOptions.RunContinuationsAsynchronously);
    await queue.Writer.WriteAsync(new InferenceJob(sessionId, req.Message, tcs, ct), ct);

    string reply = await tcs.Task;
    return Results.Ok(new ChatResponse(sessionId, reply));
});
```

For production, persist the session map in Redis or a database.

## Warm-up at startup

The first request triggers model load (several seconds to minutes on a cold machine). Warm up during host startup so the first user does not wait:

```csharp
app.Lifetime.ApplicationStarted.Register(async () =>
{
    var engine = app.Services.GetRequiredService<Engine>(); // forces construction
    _ = await engine.InitiateNewSession(); // optional: warm session creation
    Console.WriteLine("LLM engine warmed up.");
});
```

## Graceful shutdown

The hosted service handles `StopAsync`. To drain the queue before shutdown:

```csharp
public class InferenceWorker : BackgroundService
{
    // ... existing fields

    public override async Task StopAsync(CancellationToken cancellationToken)
    {
        // Stop accepting new jobs.
        _queue.Writer.Complete();

        // Drain is handled by ExecuteAsync exiting when the channel is empty.
        await base.StopAsync(cancellationToken);
    }
}
```

## Constraints

- **Single `AsposeLLMApi` / `Engine` per process.** Stick to the DI path; do not construct `AsposeLLMApi` manually on the side.
- **Model on disk, not RAM.** Model files live in the cache; first load takes time. Budget cold-start seconds.
- **No streaming.** `GetChatSessionResponse` returns the full text. For user-perceived responsiveness, return HTTP 200 after the full generation — typical 1-5 seconds.
- **Serialize inference.** Do not hit `Engine.GetChatSessionResponse` concurrently; the channel pattern handles this.

## Full project structure

```
MyLlmService/
├── Program.cs              # Minimal API + DI registration
├── Endpoints/
│   └── ChatEndpoint.cs     # /chat handler
├── Services/
│   ├── InferenceQueue.cs   # Channel-backed queue
│   ├── InferenceWorker.cs  # Hosted service draining the queue
│   └── UserSessionMap.cs   # User → session state
├── Contracts/
│   └── ChatRequest.cs      # DTOs
├── appsettings.json
└── MyLlmService.csproj
```

## What's next

- [Dependency injection](/net/developer-reference/dependency-injection/) — full DI reference.
- [Multiple concurrent sessions](/net/use-cases/multiple-concurrent-sessions/) — the session-routing pattern.
- [Cache management](/net/developer-reference/cache-management/) — keep session memory under control in long-running hosts.

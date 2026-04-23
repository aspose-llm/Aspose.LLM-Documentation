---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/how-to/handle-cancellation/
feedback: LLMNET
version: 26.5.0
title: Handle cancellation
description: Cancel in-flight Aspose.LLM for .NET generation cleanly — CancellationToken on message methods, session state after cancel, typical UX patterns.
keywords:
- cancellation
- CancellationToken
- cancel generation
- abort
- timeout
---

Both `SendMessageAsync` and `SendMessageToSessionAsync` accept a `CancellationToken`. Firing it stops token generation promptly. The session state remains intact — you can continue the conversation with the next call.

## Cancel on timeout

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(30));

try
{
    string reply = await api.SendMessageAsync(
        "Write a 500-word essay about migration patterns of the Arctic tern.",
        cancellationToken: cts.Token);
    Console.WriteLine(reply);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Generation timed out.");
}
```

## Cancel on user action (Ctrl+C)

```csharp
using var cts = new CancellationTokenSource();
Console.CancelKeyPress += (_, e) =>
{
    e.Cancel = true;   // prevent default process termination
    cts.Cancel();
};

try
{
    string reply = await api.SendMessageAsync(prompt, cancellationToken: cts.Token);
    Console.WriteLine(reply);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Cancelled by user.");
}
```

## Cancel from an HTTP host

ASP.NET Core passes a request cancellation token to endpoint handlers. Forward it to the SDK:

```csharp
app.MapPost("/chat", async (ChatRequest req, Engine engine, CancellationToken ct) =>
{
    string sessionId = req.SessionId ?? await engine.InitiateNewSession();
    try
    {
        string reply = await engine.GetChatSessionResponse(sessionId, req.Message, null, ct);
        return Results.Ok(new { sessionId, reply });
    }
    catch (OperationCanceledException)
    {
        return Results.StatusCode(499); // client closed request
    }
});
```

When the client disconnects, `ct` fires; the SDK stops generating.

## Session state after cancellation

- The partial output is **discarded** — the user's message goes into the history, but no assistant message is recorded.
- The session remains alive and can accept a new message immediately.

```csharp
try
{
    await api.SendMessageToSessionAsync(sessionId, "Long prompt...", cancellationToken: ct);
}
catch (OperationCanceledException)
{
    // Session is still usable.
}

// Continue without issue:
string reply = await api.SendMessageToSessionAsync(sessionId, "Short follow-up.");
```

## Combine timeout with linked tokens

For a handler that has both a user-cancellation token and a timeout:

```csharp
using var timeoutCts = new CancellationTokenSource(TimeSpan.FromSeconds(60));
using var linkedCts = CancellationTokenSource.CreateLinkedTokenSource(
    externalCancellationToken, timeoutCts.Token);

string reply = await api.SendMessageAsync(prompt, cancellationToken: linkedCts.Token);
```

Either source cancels the operation. Inspect which one fired if your UX needs to tell them apart:

```csharp
catch (OperationCanceledException) when (timeoutCts.IsCancellationRequested)
{
    // Timed out.
}
catch (OperationCanceledException)
{
    // User or external cancellation.
}
```

## What cancellation does not cover

- **Model load** — the synchronous model-load inside `AsposeLLMApi.Create` is not interruptible via `CancellationToken`. Budget for the cold start; do not try to cancel it.
- **Binary download** — same. The first-run binary deployment runs during `Create` and is synchronous.

For application-level time limits on startup, wrap `Create` in a `Task.Run` with an external watchdog — but be aware that even if you stop waiting on the task, the background work continues until it completes or the process terminates.

## What's next

- [AsposeLLMApi facade](/net/developer-reference/asposellmapi/) — method signatures including `CancellationToken`.
- [Integration with ASP.NET Core](/net/use-cases/integration-with-aspnet-core/) — cancellation in HTTP hosts.
- [Chat sessions](/net/developer-reference/chat-sessions/) — session lifecycle around cancellation.

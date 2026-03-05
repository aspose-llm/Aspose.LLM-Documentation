---
weight: 10
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/use-cases/simple-chat/
feedback: LLMNET
title: Simple chat
description: Send messages without explicit session management.
keywords:
- simple
- chat
- SendMessageAsync
---

For a quick question or a short exchange, you can use `SendMessageAsync` without managing sessions yourself. The API starts a session automatically if none exists.

## Example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var preset = new Qwen25Preset();
using var api = AsposeLLMApi.Create(preset);

string reply = await api.SendMessageAsync("What is 2 + 2?");
Console.WriteLine(reply);
```

Each call to `SendMessageAsync` without a session uses or creates the current session, so a few sequential calls form one conversation:

```csharp
await api.SendMessageAsync("My name is Alice.");
string reply = await api.SendMessageAsync("What is my name?");
// reply should reflect "Alice"
```

## When to use

- Single question/answer.
- Short back-and-forth where you do not need to save the session or share it with other code paths.

For multiple distinct conversations or explicit control, use [Multi-turn chat](/net/use-cases/multi-turn-chat/) with `StartNewChatAsync` and `SendMessageToSessionAsync`.

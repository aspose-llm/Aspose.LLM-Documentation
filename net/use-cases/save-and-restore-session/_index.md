---
weight: 30
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/use-cases/save-and-restore-session/
feedback: LLMNET
title: Save and restore session
description: Persist a chat session and resume it later.
keywords:
- save
- restore
- session
- persistence
---

You can save a chat session to a file and load it later to continue the same conversation (e.g. after closing the app or moving to another machine).

## Save a session

After exchanging messages with a session, save it with a path of your choice:

```csharp
string sessionId = await api.StartNewChatAsync();
await api.SendMessageToSessionAsync(sessionId, "Remember: my favorite color is blue.");
api.SaveChatSession(sessionId, "my-chat.json");
```

If you omit the file path, the library uses the session ID as the file name (location may depend on the implementation).

## Restore and continue

Later, load the session and keep chatting:

```csharp
string sessionId = await api.LoadChatSession("my-chat.json");
string reply = await api.SendMessageToSessionAsync(sessionId, "What is my favorite color?");
// reply should refer to "blue"
```

## Use cases

- **Resumable conversations** — User closes the app and reopens; load the session and continue.
- **Backup** — Periodically save important conversations.
- **Migration** — Save on one machine, copy the file, load on another (same API version and model/preset compatibility).

For implementation details, see [Session persistence](/net/developer-reference/session-persistence/) in the Developer's reference.

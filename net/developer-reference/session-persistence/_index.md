---
weight: 30
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/developer-reference/session-persistence/
feedback: LLMNET
title: Session persistence
description: Saving and loading chat sessions with Aspose.LLM for .NET.
keywords:
- save
- load
- session
- SaveChatSession
- LoadChatSession
---

You can save a chat session to disk and load it later to continue the conversation.

## Saving a session

```csharp
api.SaveChatSession(sessionId, filePath: "my-session.json");
```

If you omit `filePath`, the session is saved using the session ID as the file name (in the current working directory or a default location, depending on the implementation). The saved file contains the conversation state (e.g. message history and context) so it can be restored later.

## Loading a session

```csharp
string sessionId = await api.LoadChatSession("my-session.json");
```

This restores the session and returns its identifier. You can then send new messages to that session with `SendMessageToSessionAsync(sessionId, message)`.

## Typical use

1. Start a chat with `StartNewChatAsync` or use an existing session.
2. Exchange messages with `SendMessageToSessionAsync`.
3. When you want to pause, call `SaveChatSession(sessionId, filePath)`.
4. Later, call `LoadChatSession(filePath)` to get the session ID and continue the conversation.

This is useful for long-running conversations, resumable chats, or persisting state across application restarts.

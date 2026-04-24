---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/chat/
feedback: LLMNET
version: 26.5.0
title: Chat parameters
description: Configure session-level conversation settings in Aspose.LLM for .NET — system prompt, prior history, max tokens per response, and KV cache cleanup strategy.
keywords:
- ChatParameters
- SystemPrompt
- MaxTokens
- CacheCleanupStrategy
- History
- chat session
---

`ChatParameters` holds the settings that apply to every chat session created from a preset: the system prompt, the initial conversation history, the per-response token budget, and how the KV cache is pruned when it overflows.

## Class reference

```csharp
namespace Aspose.LLM.Abstractions.Models;

public class ChatParameters
{
    public string SystemPrompt { get; set; } = "";
    public List<ChatMessage>? History { get; set; }
    public int MaxTokens { get; set; } = 2048;
    public CacheCleanupStrategy CacheCleanupStrategy { get; set; }
        = CacheCleanupStrategy.RemoveOldestMessages;
}

public enum CacheCleanupStrategy
{
    KeepSystemPromptOnly,
    KeepSystemPromptAndHalf,
    KeepSystemPromptAndFirstUserMessage,
    KeepSystemPromptAndLastUserMessage,
    RemoveOldestMessages,
}
```

## Detailed field reference

Each field has a dedicated page with full defaults, scenario tables, code examples, and interactions.

- [SystemPrompt](/net/developer-reference/parameters/chat/system-prompt/)
- [History](/net/developer-reference/parameters/chat/history/)
- [MaxTokens](/net/developer-reference/parameters/chat/max-tokens/)
- [CacheCleanupStrategy](/net/developer-reference/parameters/chat/cache-cleanup-strategy/)

## Fields

| Field | Type | Default | Purpose |
|---|---|---|---|
| `SystemPrompt` | `string` | `""` | Default system prompt applied to every new session created from this preset. |
| `History` | `List<ChatMessage>?` | `null` | Initial chat history injected into newly created sessions. |
| `MaxTokens` | `int` | `2048` | Hard cap on tokens generated per single assistant response. |
| `CacheCleanupStrategy` | enum | `RemoveOldestMessages` | Policy for trimming the KV cache when context fills up. |

### `SystemPrompt`

The default system prompt for sessions created from this preset. Applied at session-creation time — both when you call `StartNewChatAsync` and when `SendMessageAsync` creates the current session implicitly.

Empty string is a valid choice for presets whose chat template does not want a system turn (some Gemma variants, for example). To disable the system turn entirely, keep this empty:

```csharp
preset.ChatParameters.SystemPrompt = "";
```

For consistent behavior across sessions, set it in your preset subclass or before calling `AsposeLLMApi.Create`:

```csharp
preset.ChatParameters.SystemPrompt =
    "You are a precise technical assistant. Cite sources and say when you do not know.";
```

### `History`

Optional pre-filled conversation history. When set, new sessions start with these turns already in the KV cache — useful for few-shot priming or for restoring a context that you assembled in your application.

Leave `null` for a blank session:

```csharp
preset.ChatParameters.History = null;
```

Or inject turns:

```csharp
using Aspose.LLM.Abstractions.Models;

preset.ChatParameters.History = new List<ChatMessage>
{
    ChatMessage.CreateUserMessage("What is the capital of France?"),
    ChatMessage.CreateAssistantMessage("Paris."),
};
```

The history is applied to every new session created from this preset. If you only want to prime one session, construct it and send the priming turns manually instead of setting `History`.

### `MaxTokens`

Upper bound on tokens the engine generates for a single assistant response. The default `2048` fits most general-purpose models and prompts.

{{% alert color="primary" %}}
**Reasoning-model budget.** Qwen3, DeepSeek-R1, and other chain-of-thought models emit a hidden `<think>…</think>` block before the actual answer. That block alone routinely uses 300-500 tokens for even trivial questions. Set `MaxTokens` to **at least 512** — ideally 1024-2048 — when using such models, or the response is truncated mid-reasoning and you get no visible answer. The `/no_think` directive (Qwen3) is unreliable across versions; raise the token budget instead.
{{% /alert %}}

Pick based on the task:

| Scenario | Recommended `MaxTokens` |
|---|---:|
| Short answers, classifications | 128 - 256 |
| Conversational replies | 512 - 1024 |
| Essays, long summaries | 2048 - 4096 |
| Reasoning-model output (Qwen3, DeepSeek-R1) | 1024 - 4096 |
| Code generation | 1024 - 4096 |

Raising `MaxTokens` does not cost anything upfront — it is a cap, not an allocation. The engine generates as many tokens as needed up to this limit.

### `CacheCleanupStrategy`

Policy the engine applies when the current session's KV cache would overflow `ContextParameters.ContextSize`. The engine trims automatically as generation proceeds; you can also trigger explicit trimming with `AsposeLLMApi.ForceCacheCleanup(strategy)`.

| Strategy | Keeps | When to use |
|---|---|---|
| `RemoveOldestMessages` (default) | System prompt + the most recent turns | General-purpose. Preserves recency, drops the earliest history. |
| `KeepSystemPromptOnly` | System prompt only | Hard reset of session context. Useful between topics in a long-running session. |
| `KeepSystemPromptAndHalf` | System prompt + the second half of history | Balanced recall and room for new turns. |
| `KeepSystemPromptAndFirstUserMessage` | System prompt + the first user turn | Recall-heavy tasks where the original ask matters (long analyses, debugging sessions). |
| `KeepSystemPromptAndLastUserMessage` | System prompt + the most recent user turn | Focus on the current question, drop the middle. |

See [Chat sessions — Manage the KV cache](/net/developer-reference/chat-sessions/#manage-the-kv-cache) for the call-site details and runtime semantics.

## Typical recipes

### Concise assistant with moderate output

```csharp
var preset = new Qwen25Preset();
preset.ChatParameters.SystemPrompt =
    "You are a concise assistant. Each answer fits in two short sentences.";
preset.ChatParameters.MaxTokens = 256;
```

### Long-form technical writer

```csharp
var preset = new Qwen25Preset();
preset.ChatParameters.SystemPrompt =
    "You are a technical writer. Produce detailed, well-structured paragraphs.";
preset.ChatParameters.MaxTokens = 4096;
preset.ChatParameters.CacheCleanupStrategy = CacheCleanupStrategy.KeepSystemPromptAndFirstUserMessage;
```

### Reasoning model with enough budget

```csharp
var preset = new DeepseekR1Qwen3Preset();
preset.ChatParameters.SystemPrompt = "You are a careful reasoning assistant.";
preset.ChatParameters.MaxTokens = 2048; // leaves room for <think> plus final answer
```

### Few-shot priming via history

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ChatParameters.History = new List<ChatMessage>
{
    ChatMessage.CreateUserMessage("Translate to French: The cat sits on the mat."),
    ChatMessage.CreateAssistantMessage("Le chat est assis sur le tapis."),
    ChatMessage.CreateUserMessage("Translate to French: The dog runs in the park."),
    ChatMessage.CreateAssistantMessage("Le chien court dans le parc."),
};
// Now every new session starts with these examples.
```

## What's next

- [Chat sessions](/net/developer-reference/chat-sessions/) — how the engine uses these parameters at runtime.
- [Context parameters](/net/developer-reference/parameters/context/) — the `ContextSize` field that drives cache cleanup timing.
- [Multi-turn chat](/net/use-cases/multi-turn-chat/) — runnable example showing cache management in practice.

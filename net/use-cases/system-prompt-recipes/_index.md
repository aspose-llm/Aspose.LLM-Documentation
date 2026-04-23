---
weight: 150
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/system-prompt-recipes/
feedback: LLMNET
version: 26.5.0
title: System prompt recipes
description: Effective system prompts for Aspose.LLM for .NET chat sessions — concise assistant, structured output, role-play, tool-use stand-ins, and format enforcement.
keywords:
- system prompt
- prompt engineering
- ChatParameters
- structured output
- role
- format
- JSON output
---

The system prompt sets the assistant's role, tone, and output format for every turn in a session. Small changes to the system prompt produce large changes in output quality and style. This page collects working recipes for common scenarios.

Set the system prompt on `ChatParameters.SystemPrompt` before creating the API:

```csharp
preset.ChatParameters.SystemPrompt = "<your prompt here>";
using var api = AsposeLLMApi.Create(preset);
```

Or pass a preset with a different system prompt to a single `StartNewChatAsync` call.

## Concise assistant

Short, direct answers. Good for chat UIs, FAQs, quick lookups.

```
You are a concise assistant. Each answer fits in two sentences or fewer.
Answer directly. Do not explain your reasoning unless asked.
```

## Careful reasoning

Encourage step-by-step thinking. Combine with a larger `MaxTokens` budget.

```
You are a careful analyst. Think step by step before answering.
If you are unsure, say "I don't know" instead of guessing.
Cite the part of the question you are addressing in your first sentence.
```

For reasoning-tuned models (DeepSeek-R1, Qwen3 with chain-of-thought), raise `MaxTokens` to 1024-2048.

## Structured output — JSON

Ask for machine-readable output. Works best with low temperature.

```
You are an extractor. For every user message, respond with JSON only.
Do not include any explanation or markdown code fences.

Schema:
{
  "topic": "<topic keyword>",
  "sentiment": "<positive|neutral|negative>",
  "summary": "<one-sentence summary>"
}
```

Combine with:

```csharp
preset.SamplerParameters.Temperature = 0.1f;
preset.SamplerParameters.TopP = 0.9f;
```

Validate the output on the caller side — the model can still misbehave and produce narrative text. Use a JSON parser with a fallback:

```csharp
string reply = await api.SendMessageAsync(userMessage);
try
{
    var doc = System.Text.Json.JsonDocument.Parse(reply);
    // ... use doc
}
catch (JsonException)
{
    // Model did not comply. Retry with a stricter prompt or log for manual review.
}
```

## Structured output — tagged lists

For simpler cases where JSON is overkill, use tags:

```
You are a summarizer. For each input, return three tagged lines:

[title] One-line title
[summary] Two-sentence summary
[keywords] comma, separated, keywords
```

Easier to parse with regex; more forgiving of model mistakes.

## Role-play / persona

```
You are Ada Lovelace, a 19th-century mathematician. Speak in the register of
that era — polite, precise, with occasional reference to analytical engines.
Never break character.
```

Model compliance varies. Use higher temperature for creative variance.

## Domain-constrained expert

Anchor the model to a specific domain.

```
You are a senior cloud architect specialized in AWS. Answer questions about
AWS services, cost optimization, and security best practices.

If a question is outside your domain (e.g., non-AWS topics, personal advice,
general chit-chat), reply: "That's outside my domain — I can help with AWS."
```

## Tool-use stand-in

The SDK does not support built-in function calling (see [Features](/net/product-overview/features/)), but you can instruct the model to emit structured tool-call requests that your application parses:

```
You are a helpful assistant with access to these tools:

- get_weather(city: string) -> returns the current weather
- get_stock_price(ticker: string) -> returns the latest price

When you need to call a tool, respond with JSON exactly:
{"tool": "<tool_name>", "args": {...}}

Then stop. I will call the tool and send you the result as the next message.
Only respond in natural language when you have the final answer for the user.
```

Your application parses each reply, runs the tool, and sends back the result as the next user message. This is a manual-pipeline version of function calling.

## Format enforcement

Force the model into a specific format even for chit-chat questions:

```
Every response must follow this template exactly:

Q: <repeat the user's question>
A: <your answer in one short sentence>

Do not break this format under any circumstances.
```

Pair with low temperature and explicit rejection of malformed outputs on the caller side.

## Refusal / safety wrapper

```
You are a cautious assistant.

If the user asks for:
- Personal medical, legal, or financial advice — refuse and suggest consulting
  a qualified professional.
- Instructions for harmful or illegal activities — refuse politely.

For all other questions, be helpful and direct.
```

This is a simple filter at the system level. It does not replace a proper moderation layer in production — it is a line of defense, not the only one.

## Language enforcement

```
You are a translator. Translate every input to French.
Do not respond in any other language, even if the input is a question in
English. Only output the French translation.
```

## Few-shot priming via ChatParameters.History

For tasks where the system prompt is not enough, pre-fill the history with worked examples. See [Chat parameters — History](/net/developer-reference/parameters/chat/#history).

```csharp
using Aspose.LLM.Abstractions.Models;

preset.ChatParameters.SystemPrompt = "You classify email subjects.";
preset.ChatParameters.History = new List<ChatMessage>
{
    ChatMessage.CreateUserMessage("Your account has been charged"),
    ChatMessage.CreateAssistantMessage("billing"),
    ChatMessage.CreateUserMessage("Password reset requested"),
    ChatMessage.CreateAssistantMessage("security"),
};
```

Every new session starts after these examples. The model follows the pattern demonstrated in the priming history.

## Model-specific notes

- **Qwen3** — emits `<think>…</think>` blocks before answers. Raise `MaxTokens`; consider stripping think blocks in a custom `IPromptFormatter` (see [Custom prompt formatter](/net/developer-reference/extensibility/custom-prompt-formatter/)).
- **Gemma 3** — sometimes prefers an empty system prompt; test with `ChatParameters.SystemPrompt = ""`.
- **DeepSeek-R1** — reasoning model. Same budget considerations as Qwen3.

## What's next

- [Chat parameters](/net/developer-reference/parameters/chat/) — `SystemPrompt`, `MaxTokens`, `CacheCleanupStrategy`.
- [Sampler parameters](/net/developer-reference/parameters/sampler/) — temperature and sampling knobs that interact with the system prompt.
- [Custom preset](/net/use-cases/custom-preset/) — build a reusable subclass with your favorite system prompt baked in.

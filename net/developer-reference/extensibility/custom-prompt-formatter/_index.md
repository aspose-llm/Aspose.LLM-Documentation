---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/extensibility/custom-prompt-formatter/
feedback: LLMNET
version: 26.5.0
title: Custom prompt formatter
description: Replace IPromptFormatter in Aspose.LLM for .NET to support custom chat templates or post-process model output.
keywords:
- IPromptFormatter
- chat template
- prompt formatting
- custom
- extensibility
---

`IPromptFormatter` abstracts **prompt formatting** — how each `ChatMessage` is rendered into the model-ready text before inference, and how the raw model output is cleaned up before the API returns.

Substitute the default formatter when your model uses a custom chat template that the SDK's built-in templates do not match, or when you want to inject post-processing on responses (stripping chain-of-thought blocks, tagging entities, redaction).

## Interface reference

```csharp
namespace Aspose.LLM.Abstractions.Interfaces;

public interface IPromptFormatter
{
    string? ArtificalEOSToken { get; }
    string FormatPrompt(ChatMessage message, bool addGenerationPrompt = false);
    string RefineResponse(string response);
}
```

| Member | Purpose |
|---|---|
| `ArtificalEOSToken` | Optional stop token the engine respects during generation. Return `null` to use the model's native EOS. |
| `FormatPrompt` | Renders a single `ChatMessage` into template-formatted text. Called once per turn when building the prompt. `addGenerationPrompt` is `true` for the final turn — append the model's "assistant starts here" token. |
| `RefineResponse` | Cleans raw model output before the API returns. Typical work: strip template markup, trim control tokens. |

## Example — strip reasoning blocks

Qwen3 and DeepSeek-R1 emit `<think>…</think>` blocks before the actual answer. If you want to hide those from end users, post-process them away:

```csharp
using Aspose.LLM.Abstractions.Interfaces;
using Aspose.LLM.Abstractions.Models;
using System.Text;
using System.Text.RegularExpressions;

public class StripThinkFormatter : IPromptFormatter
{
    private readonly IPromptFormatter _inner;

    public StripThinkFormatter(IPromptFormatter inner) => _inner = inner;

    public string? ArtificalEOSToken => _inner.ArtificalEOSToken;

    public string FormatPrompt(ChatMessage message, bool addGenerationPrompt = false)
        => _inner.FormatPrompt(message, addGenerationPrompt);

    public string RefineResponse(string response)
    {
        // Delegate first to the inner formatter (which does template cleanup).
        string cleaned = _inner.RefineResponse(response);

        // Strip reasoning blocks.
        cleaned = Regex.Replace(
            cleaned,
            @"<think>[\s\S]*?</think>\s*",
            string.Empty,
            RegexOptions.Multiline);

        return cleaned.Trim();
    }
}
```

## Example — fully custom template

For a model with a non-standard format, implement both `FormatPrompt` and `RefineResponse` end-to-end:

```csharp
public class MyCustomFormatter : IPromptFormatter
{
    public string? ArtificalEOSToken => "<|eot|>";

    public string FormatPrompt(ChatMessage message, bool addGenerationPrompt = false)
    {
        if (message is null)
            throw new ArgumentNullException(nameof(message));

        var sb = new StringBuilder();

        switch (message.Role)
        {
            case "system":
                sb.Append("<|sys|>").Append(message.Content).Append("<|eot|>");
                break;
            case "user":
                sb.Append("<|usr|>").Append(message.Content).Append("<|eot|>");
                break;
            case "assistant":
                sb.Append("<|asst|>").Append(message.Content).Append("<|eot|>");
                break;
        }

        if (addGenerationPrompt && message.Role == "user")
            sb.Append("<|asst|>");

        return sb.ToString();
    }

    public string RefineResponse(string response)
    {
        // Trim anything after our EOS token.
        int eotIndex = response.IndexOf("<|eot|>", StringComparison.Ordinal);
        if (eotIndex >= 0)
            response = response.Substring(0, eotIndex);

        return response.Trim();
    }
}
```

## Caveats

- **Template correctness is load-bearing.** Getting `FormatPrompt` wrong produces garbled output — the model sees malformed sequences and generates nonsense.
- **Vision presets have their own templates.** The chat templates for vision are handled internally by `Aspose.LLM.Interop.Multimodal.VisualModelChatTemplates` based on model metadata. A custom `IPromptFormatter` does not override vision template selection — that path is not yet extensible in the current release.
- **Built-in presets use their own formatters.** Substituting `IPromptFormatter` in the DI container replaces the default, but each preset may wire its own formatter internally. Test carefully when substituting; when in doubt, contact [Aspose support](https://forum.aspose.com/).

## Registration

```csharp
using Microsoft.Extensions.DependencyInjection;
using Aspose.LLM.Abstractions.Interfaces;
using Aspose.LLM.Core.DependencyInjection;

services.AddLlamaServices(new Qwen25Preset());
services.AddSingleton<IPromptFormatter, MyCustomFormatter>();
```

## What's next

- [Chat templates (multimodal)](/net/developer-reference/multimodal/chat-templates/) — vision template auto-selection.
- [Chat sessions](/net/developer-reference/chat-sessions/) — how the formatter is invoked per turn.
- [Extensibility overview](/net/developer-reference/extensibility/) — other substitution points.

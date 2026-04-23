---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/session-persistence/portability/
feedback: LLMNET
version: 26.5.0
title: Session file portability
description: When you can safely move a saved Aspose.LLM for .NET chat session between machines, versions, and models — and what breaks compatibility.
keywords:
- session portability
- SaveChatSession
- LoadChatSession
- version compatibility
- model compatibility
- ReleaseTag
---

Session files produced by `SaveChatSession` move between processes reliably — that is the whole point. Moving them between different machines, different SDK versions, or different models is a different story. This page covers when portability holds and when it breaks.

## When a session is portable

A saved session loads correctly on another host when **all** of these match:

| Axis | Must match |
|---|---|
| SDK major version | Same major version (e.g., 26.x ↔ 26.x). |
| SDK minor version | Same or higher on the load side within the same major. |
| Model file | Same Hugging Face repository, same file name, same quantization. |
| `BinaryManagerParameters.ReleaseTag` | Same `llama.cpp` runtime tag. |
| Preset family | Same preset class, or a custom preset configured identically. |

When any axis differs, behavior ranges from silent garbled output to a load-time exception.

## What breaks compatibility

### Different model

The KV cache records token positions tied to the original tokenization. A different model has a different tokenizer — the positions no longer align with the new model's token stream. Output becomes garbled or nonsense.

### Different `ReleaseTag`

`llama.cpp` releases occasionally change internal KV layout. Sessions saved against one tag may deserialize without error but produce incorrect continuations when loaded against another.

Rule of thumb: pin `BinaryManagerParameters.ReleaseTag` explicitly in production so restores are deterministic.

### Different SDK major version

Session file format is internal and evolves across major versions. A 26.x file on a 27.x runtime typically throws `InvalidOperationException` at `LoadChatSession`.

Minor versions within the same major are expected to be compatible, but verify after an upgrade.

### Different hardware architecture

Same SDK version, same model, same tag — still works across different architectures (x64, ARM64, Apple Silicon) as long as the model loads on both. The session file is architecture-independent.

## Recommended archive format

When storing sessions long-term, wrap each JSON file with a manifest recording the axes above:

```json
{
  "sessionFile": "support-ticket-1234.json",
  "sdkVersion": "26.5.0",
  "releaseTag": "b8816",
  "preset": "Qwen25Preset",
  "modelRepo": "bartowski/Qwen2.5-7B-Instruct-GGUF",
  "modelFile": "Qwen2.5-7B-Instruct-Q4_K_M.gguf",
  "savedAt": "2026-04-23T12:00:00Z"
}
```

Before loading an archived session, compare the manifest against your current configuration. Abort the load if any axis differs unless you have verified compatibility.

## Known load-time nuance

`LoadChatSession` creates the restored session with **default** `ContextParameters`, `ChatParameters`, and `SamplerParameters` — not the preset's values or the values in effect at save time.

Practical impact:

- **Model** — loaded by the current `AsposeLLMApi` instance. Correct as long as the model matches the save-time model.
- **Sampler** — reverts to library defaults (temperature 0.7, top-p 0.9, etc.). If your application relies on specific sampler tuning, re-apply it after loading.
- **System prompt** — reverts to default (typically empty). Re-apply if needed.
- **Max tokens** — default `2048`. Verify this covers your use case, especially for reasoning models.

Workaround: after `LoadChatSession`, your application can re-apply configuration manually:

```csharp
string sessionId = await api.LoadChatSession("session.json");

// TODO: re-apply any preset-specific parameters the loaded session should use.
// The current API does not expose a per-session parameter override; this limitation
// is tracked for a future release.
```

For strict reproducibility, an alternative pattern is to **replay the history** against a fresh session configured with your preset:

```csharp
// Read the saved file manually or via a small JSON helper.
var archivedMessages = ParseHistoryFromFile("session.json");

// Start a fresh session with the desired preset.
string freshSessionId = await api.StartNewChatAsync();

// Replay user turns; the model regenerates replies.
foreach (var msg in archivedMessages.Where(m => m.Role == "user"))
{
    await api.SendMessageToSessionAsync(freshSessionId, msg.Content);
}
```

This costs re-inference time but guarantees the preset's full parameter set applies.

## Security note

Saved JSON is plain text — user prompts, assistant replies, and media metadata are visible to anyone with file access. Encrypt sensitive sessions at the application level before writing them to durable storage.

## What's next

- [Session persistence](/net/developer-reference/session-persistence/) — save / load semantics.
- [Save and restore session use case](/net/use-cases/save-and-restore-session/) — runnable example.
- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — pin `ReleaseTag` for deterministic restores.

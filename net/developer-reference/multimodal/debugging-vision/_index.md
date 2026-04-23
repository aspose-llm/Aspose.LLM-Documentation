---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/multimodal/debugging-vision/
feedback: LLMNET
version: 26.5.0
title: Debugging vision
description: Diagnose multimodal issues in Aspose.LLM for .NET — tagged log taxonomy, common misalignments, and the parse_mm_logs.zsh helper.
keywords:
- vision debugging
- multimodal logs
- parse_mm_logs
- mmproj
- tagged logs
- KV cache
- misalignment
---

Vision flows involve several extra stages compared to plain text: image preprocessing, projector evaluation, marker tokenization, KV alignment, and generation. When something goes wrong, the failure is rarely a clean exception — instead you see garbled, repetitive, or off-topic output. This page shows how to diagnose those failures.

## Turn on debug logging

```csharp
using Microsoft.Extensions.Logging;

using var loggerFactory = LoggerFactory.Create(builder =>
    builder.AddConsole().SetMinimumLevel(LogLevel.Debug));
var logger = loggerFactory.CreateLogger<Program>();

var preset = new Qwen3VL2BPreset();
preset.EngineParameters.EnableDebugLogging = true;
preset.MtmdContextParameters.Verbosity = 3; // mtmd debug level
preset.MtmdContextParameters.PrintTimings = true;

using var api = AsposeLLMApi.Create(preset, logger);
```

With these settings, the native layers emit tagged lines during every chat operation.

## Log tag taxonomy

Tags appear at the start of each log line. Grep by tag to isolate one concern at a time.

| Tag | Origin | What it shows |
|---|---|---|
| `[MM]` | Multimodal layer | Projector load, template selection, chunk counts, alignment. |
| `[CTX]` | Context manager | Context state, batch dispatch, KV reservations. |
| `[KV]` | KV cache | Evictions, defragmentation, cleanup strategy application. |
| `DBG tok:` | Tokenizer | Raw tokens emitted for each prompt and generation step. |
| `DBG mtmd-tokenize` | `mtmd_tokenize` | Per-chunk tokenization details when images are inserted. |

Typical line shapes:

```
[MM] selected template: Qwen3VL
[MM] projector loaded: mmproj-Qwen3VL-2B-Instruct-Q8_0.gguf (524 MB)
[MM] tokenize: text chunks = 3, image chunks = 1
[CTX] batch dispatched: tokens=1536, seq_id=0
[KV] cleanup: strategy=RemoveOldestMessages, freed=4096 tokens
```

## The `parse_mm_logs.zsh` helper

The Aspose.LLM SDK repository includes a helper at `parse_mm_logs.zsh` that filters a raw log file into digestible sections. It groups output by concern:

- **Pairing** — base model + projector match.
- **Projector load** — file path and size.
- **Template choice** — which of the eight templates was selected.
- **Marker tokenization** — how the image marker tokens were emitted in the prompt.
- **Chunks** — text/image chunk counts and their positions.
- **Alignment** — whether the image chunks line up with the prompt markers.
- **Eval** — projector evaluation timings.
- **Context adoption** — which KV positions hold image embeddings.
- **Conditioning** — tokens produced in the first decode step.
- **KV state** — reservation and eviction.
- **Token stream** — every generated token.
- **Final answer** — the trimmed response text.

Usage:

```bash
./parse_mm_logs.zsh < raw-run.log > sectioned-run.txt
```

The sectioned output makes misalignments obvious — the "Alignment" section is usually where the problem sits when output is garbled.

## Common vision failures

### Garbled output or literal marker tokens in reply

**Symptom**: the reply contains `<image>`, `<img>`, or other bracketed tokens verbatim. Or the model describes something unrelated to the image.

**Root cause**: chat template mismatch. The engine picked the wrong template (or the fallback) because the GGUF's metadata does not expose the model family expected by the dispatch table.

**Fix**:

- Verify you are using a supported preset family (Qwen VL, Gemma 3 Vision, Ministral, LLaVA, Pixtral, InternVL, Llama 4, MiniCPMV). See [Chat templates](/net/developer-reference/multimodal/chat-templates/).
- If using a custom GGUF, try a different export from Hugging Face with richer metadata.
- Inspect `[MM] selected template: ...` in logs — if it says "fallback", detection failed.

### Repeated first token or truncated reply

**Symptom**: the response loops on the first few tokens or cuts off mid-sentence after one word.

**Root cause**: `MaxTokens` too low for a reasoning model, or KV cache cleanup too aggressive.

**Fix**:

- Raise [`ChatParameters.MaxTokens`](/net/developer-reference/parameters/chat/) to 1024 or 2048.
- Change [`CacheCleanupStrategy`](/net/developer-reference/cache-management/) to `KeepSystemPromptAndFirstUserMessage` if the model loses the image reference during generation.

### Slow first response

**Symptom**: 30+ seconds before the first token on a well-specced GPU.

**Root cause**: projector loaded on CPU while base model is on GPU, or the image is very large.

**Fix**:

- Set [`MtmdContextParameters.UseGpu = true`](/net/developer-reference/parameters/multimodal-context/).
- Downscale large images before attaching.
- Enable `MtmdContextParameters.PrintTimings` to see per-stage time.

### Image format error

**Symptom**: `InvalidOperationException: Unknown or unsupported image format.`

**Fix**: convert to JPEG, PNG, BMP, GIF, or WebP. See [Attaching images](/net/developer-reference/multimodal/attaching-images/#supported-formats).

### Image too large

**Symptom**: `InvalidOperationException: Image size exceeds maximum allowed (50MB).`

**Fix**: downscale or recompress. The projector processes images at 336-448 pixels regardless — oversized source is wasted work.

## Collecting a bug report

When escalating to support:

1. Run with `EnableDebugLogging = true`, `MtmdContextParameters.Verbosity = 3`, `MtmdContextParameters.PrintTimings = true`.
2. Capture the full log file.
3. Pipe through `parse_mm_logs.zsh` for a sectioned version.
4. Include:
   - Preset class name.
   - Model and `mmproj` Hugging Face paths (or custom GGUF details).
   - A sample image that reproduces the issue (if legally shareable).
   - The exact prompt that triggered the failure.
   - The sectioned log.

Submit via the [Aspose Support Forum](https://forum.aspose.com/) or a paid support ticket.

## What's next

- [Chat templates](/net/developer-reference/multimodal/chat-templates/) — the eight supported template families.
- [Multimodal context parameters](/net/developer-reference/parameters/multimodal-context/) — GPU offload and verbosity for the projector.
- [Attaching images](/net/developer-reference/multimodal/attaching-images/) — correct image input.

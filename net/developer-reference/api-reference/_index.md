---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/api-reference/
feedback: LLMNET
version: 26.5.0
title: API reference
description: Links to the full class-level API reference and a summary of the key Aspose.LLM for .NET types.
keywords:
- API
- reference
- class
- method
- AsposeLLMApi
- License
---

The complete class-level API reference for Aspose.LLM for .NET is published separately with every class, method, property, and XML documentation comment.

- **[Aspose.LLM for .NET API Reference](https://reference.aspose.com/llm/net/)** — full reference, auto-generated from the shipped XML docs.

## Key types at a glance

The following types form the everyday public surface. The external reference has full signatures, overloads, and remarks.

### Aspose.LLM

| Type | Purpose |
|---|---|
| `AsposeLLMApi` | The facade class. Create once per process, reuse for all chat operations. |
| `License` | Applies a license file or stream; exposes the static `IsLicensed` property. |

### Aspose.LLM.Abstractions.Parameters

| Type | Purpose |
|---|---|
| `EngineParameters` | Engine-wide settings: cache path, debug logging, default threads. |
| `ModelSourceParameters` | Where to load the model: local path, Aspose ID, or Hugging Face repo. |
| `ModelInferenceParameters` | Model-load knobs: GPU layers, main GPU, split mode, memory mapping, KV overrides. |
| `ContextParameters` | `llama.cpp` context knobs: context size, batch sizes, rope scaling, flash attention, KV dtype. |
| `ChatParameters` | System prompt, history, max tokens, cache cleanup strategy. |
| `SamplerParameters` | Full sampler surface: temperature, top-p, top-k, penalties, DRY, XTC, mirostat, seed. |
| `BinaryManagerParameters` | Native binary release tag, binary cache path, preferred acceleration backend. |
| `MultimodalContextParameters` | Vision (`mtmd`) context: GPU use, timings, thread count, verbosity. |

### Aspose.LLM.Abstractions.Parameters.Presets

| Type | Purpose |
|---|---|
| `PresetCoreBase` | Base class holding all nine parameter bags. Extend it for custom presets. |
| `Qwen25Preset`, `Qwen3Preset`, `Gemma3Preset`, `Llama32Preset`, `Phi4Preset`, `Oss20Preset`, `DeepSeekCoder2Preset`, `DeepseekR1Qwen3Preset`, `UnifiedDefaultLlmParameters` | Built-in text presets. |
| `Qwen25VL3BPreset`, `Qwen3VL2BPreset`, `Gemma3VisionPreset`, `Ministral3VisionPreset` | Built-in vision presets (set both base model and `mmproj`). |

### Aspose.LLM.Abstractions.Models

| Type | Purpose |
|---|---|
| `ChatMessage` | A single turn in a chat session: role, content, optional media, KV cache metadata. |
| `MediaAttachment` | Image (or other media) payload with format detection and 50 MB size limit. |
| `MediaFormat` | Enum: `Unknown`, `JPEG`, `PNG`, `BMP`, `GIF`, `WebP`. |
| `HuggingFaceModel` | Metadata for a Hugging Face model file (repo ID, filename, size, quantization). |
| `SystemSpec` | Detected OS, architecture, and available accelerations. |
| `VisionModelInfo` / `VisionModelSelector` | Candidate `mmproj` info and scoring helpers. |

### Aspose.LLM.Abstractions.Acceleration

| Type | Purpose |
|---|---|
| `AccelerationType` | Enum: `None`, `CUDA`, `HIP`, `Metal`, `Vulkan`, `Kompute`, `OpenCL`, `SYCL`, `AVX512`, `AVX2`, `AVX`, `OpenBLAS`, `NoAVX`. |
| `CudaVersion` | Detected CUDA major/minor version and installation path. |

### Aspose.LLM.Abstractions.Interfaces

| Type | Purpose |
|---|---|
| `ILlamaModel` | Loaded text model contract. |
| `IMultimodalModel` | Vision-capable extension of `ILlamaModel`. |
| `IChatSession` | Session contract — history, media, KV tracking, response generation. |
| `IModelLoader`, `IModelFileProvider`, `IPromptFormatter` | Extensibility interfaces for custom implementations via DI. |

## `AsposeLLMApi` method summary

| Method | Purpose |
|---|---|
| `Create(preset, logger?)` | Static factory. Creates the single instance. |
| `DefaultPreset` | The preset passed to `Create`. |
| `StartNewChatAsync(preset?, sessionId?)` | Create a new chat session. |
| `SendMessageAsync(message, media?, preset?, ct)` | Send to the current session (create one if needed). |
| `SendMessageToSessionAsync(sessionId, message, media?, ct)` | Send to a specific session. |
| `GetDefaultParametersAsync()` | Tuple of engine-default inference/context/chat/sampler parameters. |
| `GetDefaultPreset()` | Returns `new Qwen25Preset()` as a sensible default. |
| `SaveChatSession(sessionId, filePath?)` | Serialize a session to disk. |
| `LoadChatSession(filePath)` | Deserialize a session and set it as current. |
| `ForceCacheCleanup(strategy)` | Trim the KV cache of the current session. |
| `Dispose()` | Release native resources and the single-instance guard. |

See [Chat sessions](/net/developer-reference/chat-sessions/) and [Session persistence](/net/developer-reference/session-persistence/) for detailed semantics.

## What's next

- [Presets](/net/developer-reference/presets/) — preset base class and parameter bags.
- [Chat sessions](/net/developer-reference/chat-sessions/) — session lifecycle and messaging methods.
- [Session persistence](/net/developer-reference/session-persistence/) — save and load session state.

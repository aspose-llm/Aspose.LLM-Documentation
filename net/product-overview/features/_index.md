---
weight: 15
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/product-overview/features/
feedback: LLMNET
version: 26.5.0
title: Features
description: Capabilities of Aspose.LLM for .NET — local inference, chat sessions, vision input, acceleration, session persistence, and what the SDK does not do.
keywords:
- features
- capabilities
- local inference
- chat session
- acceleration
- vision
- session persistence
- limitations
---

Aspose.LLM for .NET lets you run large language models on your own machine or server from a .NET application. This page lists what the SDK offers and the scenarios it does not cover. Use it to evaluate fit before integration.

## Core capabilities

### On-premise inference

Run models on your CPU or GPU. Input, prompts, and model weights do not leave your process. No calls to OpenAI, Anthropic, or any other hosted inference service.

### Preset-based model configuration

Each preset bundles the model source, chat template, context size, sampler settings, and inference parameters for a specific model family:

- **Text presets**: `Qwen25Preset`, `Qwen3Preset`, `Gemma3Preset`, `Llama32Preset`, `Phi4Preset`, `DeepSeekCoder2Preset`, `DeepseekR1Qwen3Preset`, `Oss20Preset`, `UnifiedDefaultLlmParameters`.
- **Vision presets**: `Qwen25VL3BPreset`, `Qwen3VL2BPreset`, `Gemma3VisionPreset`, `Ministral3VisionPreset`.

See [Supported presets](/net/product-overview/supported-presets/) for a full list with model sources.

### Custom presets

Extend `PresetCoreBase` to bring your own GGUF model. Override any parameter bag (context size, sampler, chat template, GPU layers, quantization) while keeping the rest at defaults. See the custom preset use case for a walkthrough.

### Chat sessions with history

Sessions hold the conversation state across turns. Each message extends the history and the KV cache that carries it:

- `StartNewChatAsync` creates a session and returns its ID.
- `SendMessageAsync` sends a message in the current session (creates one if none exists).
- `SendMessageToSessionAsync(sessionId, ...)` targets a specific session.

A single `AsposeLLMApi` instance can host many sessions concurrently, each with its own history and KV region.

### Session persistence

Save a session to disk and restore it later to continue the conversation:

```csharp
api.SaveChatSession(sessionId, "session-42.json");
// ... later, in the same or a different process
string sessionId = await api.LoadChatSession("session-42.json");
```

Useful for resumable conversations, backups, and migrations between machines running the same SDK version.

### Cache cleanup strategies

When a conversation approaches the context size, the KV cache needs to be trimmed to make room. `CacheCleanupStrategy` offers five modes:

- `RemoveOldestMessages` (default)
- `KeepSystemPromptOnly`
- `KeepSystemPromptAndHalf`
- `KeepSystemPromptAndFirstUserMessage`
- `KeepSystemPromptAndLastUserMessage`

Choose per scenario: instruction-heavy conversations benefit from preserving the system prompt; recall-heavy conversations may keep the first user turn.

### Multimodal input (vision)

Vision presets accept images alongside the prompt. Pass them as byte arrays via the `media` parameter of `SendMessageAsync` or `SendMessageToSessionAsync`.

Supported image formats: JPEG, PNG, BMP, GIF, WebP. The SDK detects the format from the file's magic bytes. Per-attachment size limit: 50 MB.

Eight vision chat templates are wired into the SDK: LLaVA, Qwen2VL, Qwen3VL, Pixtral, InternVL, Gemma3, Llama4, MiniCPMV. The correct template is selected automatically from the model's metadata.

### Hardware acceleration

At first use, `BinaryManager` downloads the matching `llama.cpp` native binary for your platform and acceleration backend:

- **CUDA** (NVIDIA GPUs)
- **HIP** / **ROCm** (AMD GPUs)
- **Metal** (Apple Silicon)
- **Vulkan** (cross-platform GPU)
- **CPU** with AVX2, AVX512, or no-AVX variants

Control offload via `BaseModelInferenceParameters.GpuLayers`, `MainGpu`, `SplitMode`, and `TensorSplit`. Set `GpuLayers = 0` for CPU-only runs.

### Configurable sampling

`SamplerParameters` exposes the full `llama.cpp` sampler surface: temperature, top-p, top-k, min-p, typical-p, top-n-sigma, dynamic temperature, DRY, XTC, repetition / presence / frequency penalties, Mirostat, logit bias, seed, and more. Tune the mix per preset or per session.

### Flexible context

`ContextParameters` exposes context size, batch/ubatch, YaRN rope scaling, pooling, attention type, Flash Attention mode, KV cache quantization (`TypeK` / `TypeV`), offload options, and threading. Some presets ship with very long context (131K or 262K tokens).

### Embedding mode

Set `ContextParameters.Embeddings = true` to configure the context for embedding extraction instead of generation. Details on the full embedding workflow will be covered in a dedicated use case.

### Extensibility via DI

Core services are registered in Microsoft.Extensions.DependencyInjection through `AddLlamaServices(preset)`. You can swap implementations of `IModelLoader`, `IModelFileProvider`, `IPromptFormatter`, and `IMediaProcessor` — useful for custom model stores, alternative prompt formats, or bespoke media pre-processing.

### Logging and diagnostics

Pass an `ILogger` to `AsposeLLMApi.Create(preset, logger)`. Enable native-level debug logs by setting `EngineParameters.EnableDebugLogging = true`. Logs are tagged (`[MM]`, `[CTX]`, `[KV]`) so you can filter for multimodal, context, or KV cache events.

### Licensing

- Evaluation mode with no license.
- Apply a commercial license via `License.SetLicense(fileOrStream)`.
- Check status with `License.IsLicensed`.

## What Aspose.LLM does not do

Honest limits to set expectations before integration.

### No streaming responses

`SendMessageAsync` and `SendMessageToSessionAsync` return the complete response as a single string. Token-by-token streaming is not exposed in the public API.

### No function calling / tool use

The SDK does not expose a built-in protocol for model-invoked tool calls. If the model emits tool-call JSON in its text output, your code parses and dispatches it manually.

### No fine-tuning or training

Aspose.LLM runs inference only. Training, fine-tuning, and LoRA application are not in scope.

### No audio input

Vision presets accept images. Audio input is not wired into the SDK, even though the underlying `mtmd` runtime supports audio chunks. Audio may appear in a future release.

### No distributed inference

Inference runs in a single process on a single machine. Multi-node or multi-GPU-host setups are not managed by the SDK. Within one machine, multi-GPU is supported via `TensorSplit` and `SplitMode`.

### Not an HTTP server

Aspose.LLM is an in-process library. There is no built-in OpenAI-compatible endpoint, no REST server, and no gRPC surface. Wrap the SDK in your own service if you need remote access.

### One instance per process

A hard constraint: `AsposeLLMApi` enforces a single active instance per process at runtime. To serve multiple models, run multiple processes or swap models by disposing and recreating the instance.

### No speech-to-text or text-to-speech

Out of scope. Use a dedicated audio library for STT / TTS and pass the resulting text into Aspose.LLM.

## What's next

- [Architecture](/net/product-overview/architecture/) — layers, runtime flow, and memory footprint.
- [Supported presets](/net/product-overview/supported-presets/) — full preset list with model sources.
- [Hello, world!](/net/hello-world/) — minimal runnable example.

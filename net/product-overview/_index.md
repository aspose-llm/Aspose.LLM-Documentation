---
weight: 10
date: "2026-04-23"
author: "Aleksandr Sudakov"
type: docs
url: /net/product-overview/
feedback: LLMNET
version: 26.5.0
title: Product overview
description: Introduction to Aspose.LLM for .NET — architecture, capabilities, supported presets, and what the SDK does not do.
keywords:
- overview
- introduction
- LLM
- chat
- preset
- architecture
- features
---

Aspose.LLM for .NET is a library for integrating large language models into your .NET applications. You run models on your own infrastructure — on-premise or in a controlled cloud environment — and interact with them through a managed API: create an instance from a preset, start chat sessions, send messages, and optionally save or load conversation state.

This section covers what the SDK does, how it is built, and what it supports.

## Sections

- [Architecture](/net/product-overview/architecture/) — four-layer design (facade, engine, P/Invoke, native), runtime flow on first `Create`, memory footprint, and lifecycle.
- [Features](/net/product-overview/features/) — capabilities in detail, plus explicit scope limits (no streaming, no function calling, no fine-tuning, no audio).
- [Supported presets](/net/product-overview/supported-presets/) — built-in text and vision presets with their Hugging Face model sources and default parameters.

## At a glance

- **Preset-based setup** — built-in presets for Qwen 2.5 / Qwen 3, Gemma 3, Llama 3.2, Phi 4, DeepSeek, and gpt-oss-20b. Extend `PresetCoreBase` to bring your own GGUF model.
- **Chat sessions** — create sessions with `StartNewChatAsync`, send messages with `SendMessageAsync` or `SendMessageToSessionAsync`, and maintain multi-turn conversations per session.
- **Session persistence** — `SaveChatSession` and `LoadChatSession` serialize a session to disk and restore it later.
- **Optional multimodal input** — pass images (JPEG, PNG, BMP, GIF, WebP; up to 50 MB each) alongside prompts when using a vision preset.
- **Hardware acceleration** — CUDA, HIP, Metal, Vulkan, or CPU with AVX2 / AVX512. Native binaries download automatically on first use.
- **Single instance per process** — one `AsposeLLMApi` instance at a time. Create it once and reuse it for all sessions.
- **Licensing** — apply a commercial license via `License.SetLicense`; check status with `License.IsLicensed`. A free [temporary license](https://purchase.aspose.com/temporary-license) is available for evaluation and proof-of-concept work. Inference requires an applied license — the SDK does not run chat APIs in evaluation mode.

## What's next

- [Architecture](/net/product-overview/architecture/) — layered design and runtime flow.
- [Features](/net/product-overview/features/) — full capability list and scope limits.
- [Supported presets](/net/product-overview/supported-presets/) — pick a preset for your model and hardware.
- [Getting started](/net/getting-started/) — install, license, and run the first example.

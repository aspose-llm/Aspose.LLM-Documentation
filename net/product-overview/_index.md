---
weight: 10
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/product-overview/
feedback: LLMNET
title: Product overview
description: Introduction to Aspose.LLM for .NET and its capabilities.
keywords:
- overview
- introduction
- LLM
- chat
- preset
---

Aspose.LLM for .NET is a library for integrating large language models (LLMs) into your .NET applications. You run models on your own infrastructure (on-premise or controlled cloud) and interact with them through a simple API: create an instance from a preset, start chat sessions, send messages, and optionally save or load conversation state.

## Why Aspose.LLM?

- **Preset-based setup** — Use built-in presets for popular models (Qwen 2.5, Qwen 3, Gemma, Llama, Phi, and others) or customize parameters via `PresetCoreBase`.
- **Chat sessions** — Start a new chat with `StartNewChatAsync`, send messages with `SendMessageAsync` or `SendMessageToSessionAsync`, and keep multi-turn conversations.
- **Session persistence** — Save and load chat sessions to disk with `SaveChatSession` and `LoadChatSession` to resume conversations later.
- **Optional multimodal input** — Pass optional media (e.g. image bytes) with messages for vision-capable presets.
- **Single instance** — One `AsposeLLMApi` instance per process; create it with `AsposeLLMApi.Create(preset)`.
- **Licensing** — Apply a license with `License.SetLicense`; check status with `License.IsLicensed`.

## Capabilities

- [Supported presets](/net/product-overview/supported-presets/)  
  Overview of built-in presets (text and vision) and model sources.

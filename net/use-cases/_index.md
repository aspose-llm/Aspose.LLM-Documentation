---
weight: 60
date: "2026-04-23"
author: "Aleksandr Sudakov"
type: docs
url: /net/use-cases/
feedback: LLMNET
version: 26.5.0
title: Use cases
description: End-to-end scenarios with runnable code for Aspose.LLM for .NET — simple chat, multi-turn conversations, session persistence, and custom presets.
keywords:
- use case
- example
- chat
- session
- preset
- scenario
---

This section shows how to build real applications with Aspose.LLM for .NET. Each use case is a complete, runnable scenario — paste it into a .NET 8 or .NET 10 project, apply a license, and run.

For compact snippets that illustrate a single call, see [Quick wins](/net/quick-wins/). For the first runnable example, see [Hello, world!](/net/hello-world/).

We assume familiarity with C# and `async`/`await`. If you are new to the SDK, read [Getting started](/net/getting-started/) first.

## Core scenarios

- [Simple chat](/net/use-cases/simple-chat/) — one or a few messages without managing sessions yourself.
- [Multi-turn chat](/net/use-cases/multi-turn-chat/) — an explicit session with several exchanges in one conversation.
- [Save and restore session](/net/use-cases/save-and-restore-session/) — persist conversation state to disk and resume later.
- [Custom preset](/net/use-cases/custom-preset/) — patterns for customizing a built-in preset or building one from scratch.

## Vision

- [Vision question answering](/net/use-cases/vision-qa/) — ask questions about images, compare images, transcribe documents.

## Throughput and scaling

- [Batch processing](/net/use-cases/batch-processing/) — run many prompts through one loaded model.
- [Streaming-like responses](/net/use-cases/streaming-like-responses/) — the no-streaming limitation and workarounds.
- [Multiple concurrent sessions](/net/use-cases/multiple-concurrent-sessions/) — serve many users or workflows from a single instance.

## Deployment

- [Offline deployment](/net/use-cases/offline-deployment/) — air-gapped, firewalled, or no-internet targets.
- [CPU-only deployment](/net/use-cases/cpu-only-deployment/) — no GPU, tune threads, set realistic expectations.
- [GPU deployment with CUDA](/net/use-cases/gpu-deployment-cuda/) — NVIDIA GPUs, single and multi-GPU.
- [Integration with ASP.NET Core](/net/use-cases/integration-with-aspnet-core/) — host behind HTTP via Minimal API.

## Tuning

- [Long context tuning](/net/use-cases/long-context-tuning/) — 128K-262K contexts with flash attention and KV quantization.
- [Low memory tuning](/net/use-cases/low-memory-tuning/) — fit into tight memory budgets.

## Advanced

- [Bring your own GGUF](/net/use-cases/bring-your-own-gguf/) — custom models from Hugging Face or disk.
- [System prompt recipes](/net/use-cases/system-prompt-recipes/) — effective system-prompt patterns.

## What's next

- [Quick wins](/net/quick-wins/) — compact recipes for single tasks.
- [Developer's reference](/net/developer-reference/) — conceptual reference for presets, sessions, and the API.
- [Supported presets](/net/product-overview/supported-presets/) — pick a preset for your scenario.

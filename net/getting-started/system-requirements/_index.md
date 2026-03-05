---
weight: 10
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/system-requirements/
feedback: LLMNET
title: System requirements
description: Minimum requirements for Aspose.LLM for .NET.
keywords:
- requirements
- .NET
- framework
---

## Supported frameworks

Aspose.LLM for .NET targets .NET runtimes supported by the package (check the [NuGet page](https://www.nuget.org/packages/Aspose.LLM/) for the current target frameworks). Typically this includes .NET 6.0 and later.

## Development

- Microsoft Visual Studio 2019 or later (or another IDE that supports .NET 6+).

## Runtime

- **Memory** — Sufficient RAM for loading the model (depends on the preset; larger models require more memory).
- **Disk** — Space for model files (downloaded or cached by the library).
- **GPU** — Optional. Some presets support GPU offload for faster inference; a CUDA-capable GPU and drivers may be required for that scenario.

## Platform

The library is built for supported .NET platforms (e.g. Windows, Linux). Build and run your application for the same platform as the target runtime.

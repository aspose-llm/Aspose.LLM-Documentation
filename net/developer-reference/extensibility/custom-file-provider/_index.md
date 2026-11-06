---
weight: 20
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/extensibility/custom-file-provider/
feedback: LLMNET
version: 26.11.0
title: Custom file provider
description: Understand the model file provider interface and source selection in Aspose.LLM for .NET.
keywords:
- IModelFileProvider
- file provider
- offline
- air-gapped
- extensibility
---

`IModelFileProvider` describes an operation that resolves a model source to a local file. In version 26.11.0, the built-in `ModelManager` creates and selects its concrete local and Hugging Face providers; registering an arbitrary `IModelFileProvider` alone does not replace that selection. `AsposeModelId` is reserved and is not consumed by the built-in manager.

## Use a local model

Obtain a compatible model from its source after reviewing its publisher's license, then set `BaseModelSourceParameters.ModelFilePath` before creating the API. See [Model source parameters](/llm/net/developer-reference/parameters/model-source/) and [Hello, world!](/llm/net/hello-world/). No model weights are included or distributed by Aspose.LLM.

Version 26.11.0 may fetch a configured remote model when no local path is supplied. A separate opt-in control is not yet implemented; provide a valid local path to avoid a model download.

## Related interfaces

- [Custom model loader](/llm/net/developer-reference/extensibility/custom-model-loader/): model loading extensibility.
- [Model source parameters](/llm/net/developer-reference/parameters/model-source/): built-in source selection.

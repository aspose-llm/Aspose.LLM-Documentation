---
weight: 20
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/model-source/aspose-model-id/
feedback: LLMNET
version: 26.5.0
title: AsposeModelId
description: Reserved, non-functional model source property in Aspose.LLM for .NET.
keywords:
- AsposeModelId
- reserved property
- model ID
- priority
---

`AsposeModelId` is reserved and non-functional in version 26.11.0. No provider consumes its value, and Aspose.LLM does not host or distribute model weights.

## Quick reference

| | |
|---|---|
| **Type** | `string?` |
| **Default** | `null` |
| **Priority** | None; the property is ignored |
| **Category** | Model source |
| **Field on** | `ModelSourceParameters.AsposeModelId` |

## What it does

Leave this property unset. Obtain a model separately after reviewing its publisher's license and set [`ModelFilePath`](/llm/net/developer-reference/parameters/model-source/model-file-path/) to the local file.

## Interactions

- [`ModelFilePath`](/llm/net/developer-reference/parameters/model-source/model-file-path/) takes priority over the configured Hugging Face source.
- Setting `AsposeModelId` does not change source selection.

## What's next

- [Model source hub](/llm/net/developer-reference/parameters/model-source/): resolution order.
- [HuggingFaceRepoId](/llm/net/developer-reference/parameters/model-source/hugging-face-repo-id/): remote source configuration.

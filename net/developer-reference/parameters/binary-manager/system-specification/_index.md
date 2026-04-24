---
weight: 50
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/binary-manager/system-specification/
feedback: LLMNET
version: 26.5.0
title: SystemSpecification
description: Override auto-detected system info in Aspose.LLM for .NET — null defaults to runtime detection; rarely needed.
keywords:
- SystemSpecification
- SystemSpec
- auto-detect
- OS detection
---

`SystemSpecification` is an override for the OS, architecture, and acceleration capabilities the SDK auto-detects. Almost always left `null`.

## Quick reference

| | |
|---|---|
| **Type** | `SystemSpec?` |
| **Default** | `null` (auto-detect) |
| **Category** | Native binary source |
| **Field on** | `BinaryManagerParameters.SystemSpecification` |

## What it does

At construction, the engine inspects the host OS, CPU architecture, and available accelerations (GPU, AVX levels). The result is a `SystemSpec` that drives the asset-selection logic in `BinaryManager`.

- `null` (default) — auto-detect. Correct for almost every deployment.
- Explicit `SystemSpec` — override for specific testing / cross-platform preparation scenarios.

## When to change it

| Scenario | Value |
|---|---|
| Default | `null` |
| Force a specific target during offline binary preparation | Custom `SystemSpec` |

Do not touch this for normal runtime usage.

## Example

```csharp
var preset = new Qwen25Preset();
// preset.BinaryManagerParameters.SystemSpecification = null; // default
```

## Interactions

- [`PreferredAcceleration`](/net/developer-reference/parameters/binary-manager/preferred-acceleration/) — higher-level acceleration selector; usually preferred over raw `SystemSpec` overrides.

## What's next

- [PreferredAcceleration](/net/developer-reference/parameters/binary-manager/preferred-acceleration/) — acceleration control.
- [Binary manager hub](/net/developer-reference/parameters/binary-manager/) — all binary-manager knobs.

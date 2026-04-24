---
weight: 20
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/binary-manager/repo/
feedback: LLMNET
version: 26.5.0
title: Repo
description: GitHub repository name for llama.cpp releases in Aspose.LLM for .NET — defaults to llama.cpp; paired with Owner to form the source URL.
keywords:
- Repo
- GitHub
- llama.cpp
- repository
---

`Repo` is the GitHub repository name paired with [`Owner`](/net/developer-reference/parameters/binary-manager/owner/) to form the source URL.

## Quick reference

| | |
|---|---|
| **Type** | `string` |
| **Default** | `"llama.cpp"` |
| **Category** | Native binary source |
| **Field on** | `BinaryManagerParameters.Repo` |

## What it does

Together with `Owner`, forms the GitHub path: `github.com/<Owner>/<Repo>/releases/tag/<ReleaseTag>`. The `BinaryManager` queries the GitHub API for the release listing and downloads the matching asset.

- `"llama.cpp"` (default) — the upstream project. Correct for the default `Owner = "ggml-org"`.
- Different repo name — only if you host releases in a differently-named repository with byte-compatible binaries.

## When to change it

| Scenario | Value |
|---|---|
| Default | `"llama.cpp"` |
| Enterprise mirror under a renamed repo | Mirror repo name |

Do not change unless you control a mirror.

## Example

```csharp
var preset = new Qwen25Preset();
// preset.BinaryManagerParameters.Repo = "llama.cpp"; // default
```

## Interactions

- [`Owner`](/net/developer-reference/parameters/binary-manager/owner/) — paired.
- [`ReleaseTag`](/net/developer-reference/parameters/binary-manager/release-tag/) — release selector.

## What's next

- [Owner](/net/developer-reference/parameters/binary-manager/owner/) — repository owner.
- [ReleaseTag](/net/developer-reference/parameters/binary-manager/release-tag/) — version pin.
- [Binary manager hub](/net/developer-reference/parameters/binary-manager/) — all binary-manager knobs.

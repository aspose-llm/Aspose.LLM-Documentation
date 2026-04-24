---
weight: 10
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/binary-manager/owner/
feedback: LLMNET
version: 26.5.0
title: Owner
description: GitHub repository owner for llama.cpp releases in Aspose.LLM for .NET — defaults to ggml-org upstream; change only for validated forks.
keywords:
- Owner
- GitHub
- llama.cpp
- ggml-org
---

`Owner` is the GitHub account that owns the `llama.cpp` repository used to download native binaries. The default `ggml-org` targets the upstream project.

## Quick reference

| | |
|---|---|
| **Type** | `string` |
| **Default** | `"ggml-org"` |
| **Category** | Native binary source |
| **Field on** | `BinaryManagerParameters.Owner` |

## What it does

The `BinaryManager` constructs a GitHub release URL from `github.com/<Owner>/<Repo>/releases/tag/<ReleaseTag>`. `Owner` fills the first slot.

- `"ggml-org"` (default) — upstream llama.cpp maintainers. Recommended.
- A different owner — only when you mirror upstream releases to a fork that stays byte-compatible.

## When to change it

| Scenario | Value |
|---|---|
| Default | `"ggml-org"` |
| Air-gapped enterprise mirror | Fork owner that hosts matching releases |

Changing `Owner` without a byte-compatible mirror produces binaries whose P/Invoke signatures differ from what the SDK expects — runtime crashes.

## Example

```csharp
var preset = new Qwen25Preset();
// preset.BinaryManagerParameters.Owner = "ggml-org"; // default
```

## Interactions

- [`Repo`](/net/developer-reference/parameters/binary-manager/repo/) — the repository name on that owner.
- [`ReleaseTag`](/net/developer-reference/parameters/binary-manager/release-tag/) — specific release under the owner/repo.

## What's next

- [Repo](/net/developer-reference/parameters/binary-manager/repo/) — repository name.
- [ReleaseTag](/net/developer-reference/parameters/binary-manager/release-tag/) — version pin.
- [Binary manager hub](/net/developer-reference/parameters/binary-manager/) — all binary-manager knobs.

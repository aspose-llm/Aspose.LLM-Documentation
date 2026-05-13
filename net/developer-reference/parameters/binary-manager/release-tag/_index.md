---
weight: 30
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/binary-manager/release-tag/
feedback: LLMNET
version: 26.5.0
title: ReleaseTag
description: Specific llama.cpp release tag for native binaries in Aspose.LLM for .NET — defaults to b8816 on SDK v26.5.0; pin explicitly in production.
keywords:
- ReleaseTag
- llama.cpp
- version pin
- b8816
- native binaries
---

`ReleaseTag` pins the specific `llama.cpp` release from which native binaries are downloaded. The SDK's P/Invoke signatures are validated against the default tag; changing it without a tested migration risks runtime failures.

## Quick reference

| | |
|---|---|
| **Type** | `string` |
| **Default** | `"b8816"` (on SDK v26.5.0) |
| **Category** | Native binary source |
| **Field on** | `BinaryManagerParameters.ReleaseTag` |

## What it does

Identifies a GitHub release under `<Owner>/<Repo>`. The engine downloads the asset whose name matches your platform and acceleration from that release.

- `"b8816"` (default on v26.5.0) — the upstream release validated against this SDK version.
- Older / newer tag — only when you control a matching validation story.

The Aspose team tracks upstream llama.cpp releases and bumps the default `ReleaseTag` in each SDK minor version. The `llama-cpp-migration` skill automates the per-release validation and updates.

## When to change it

| Scenario | Value |
|---|---|
| Default (recommended) | `"b8816"` or unset |
| Dev / testing a new upstream release | That tag, only in dev |
| Pinned to an older validated tag | Specific older tag |

{{% alert color="primary" %}}
Changing `ReleaseTag` to a tag whose P/Invoke signatures differ from the SDK's expectations produces runtime crashes. Never ship an unvalidated tag to production.
{{% /alert %}}

## Example

```csharp
var preset = new Qwen25Preset();
preset.BinaryManagerParameters.ReleaseTag = "b8816";  // explicit pin for reproducibility
```

## Interactions

- [`Owner`](/llm/net/developer-reference/parameters/binary-manager/owner/) and [`Repo`](/llm/net/developer-reference/parameters/binary-manager/repo/) — together with `ReleaseTag` form the source URL.
- [`BinaryPath`](/llm/net/developer-reference/parameters/binary-manager/binary-path/) — cache location per tag; different tags use different subfolders.
- [Session persistence portability](/llm/net/developer-reference/session-persistence/portability/) — sessions assume a consistent `ReleaseTag` across save / load.

## What's next

- [Binary download fails troubleshooting](/llm/net/troubleshooting/binary-download-fails/) — common issues.
- [Binary manager hub](/llm/net/developer-reference/parameters/binary-manager/) — all binary-manager knobs.
- [Offline deployment](/llm/net/use-cases/offline-deployment/) — pre-populating binaries for a pinned tag.

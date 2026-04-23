---
weight: 70
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/troubleshooting/
feedback: LLMNET
version: 26.5.0
title: Troubleshooting
description: Diagnose and fix common Aspose.LLM for .NET problems — binary download failures, out-of-memory, GPU not detected, model load errors, garbled output, license errors, performance issues.
keywords:
- troubleshooting
- errors
- debugging
- problems
---

Common problems and their fixes. Each page follows a fixed structure: **Symptom** (what you see), **Cause** (what is happening), **Resolution** (how to fix), and optional **Prevention**.

Before diving into specific problems, make sure:

- The license is applied (`Aspose.LLM.License.IsLicensed` returns `true`).
- `EngineParameters.EnableDebugLogging = true` and a logger is wired — see [Logging and diagnostics](/net/developer-reference/logging-and-diagnostics/).
- You can reproduce the issue with a minimal example.

## Topics

- [Binary download fails](/net/troubleshooting/binary-download-fails/) — `BinaryManager` cannot reach GitHub or extracts fail.
- [Out of memory](/net/troubleshooting/out-of-memory/) — GPU OOM, system RAM OOM, cache growth.
- [GPU not detected](/net/troubleshooting/gpu-not-detected/) — CUDA/HIP/Vulkan not picked up.
- [Model not loading](/net/troubleshooting/model-not-loading/) — corrupt GGUF, unsupported architecture, wrong file.
- [Garbled output](/net/troubleshooting/garbled-output/) — nonsense replies, template mismatches, repetition loops.
- [License errors](/net/troubleshooting/license-errors/) — `Not licensed for this method`, expired temporary licenses.
- [Performance issues](/net/troubleshooting/performance-issues/) — low throughput, spikes, thread contention.

## Asking for help

When none of these match, open a thread on the [Aspose Support Forum](https://forum.aspose.com/) with:

- SDK version (`Aspose.LLM` NuGet version).
- Host OS and architecture.
- GPU model and driver version (if applicable).
- Preset class name and any overrides you applied.
- Full log from a reproducing run with debug logging enabled.
- A minimal code sample that reproduces the issue.

For paid support, use the [Aspose Helpdesk](https://helpdesk.aspose.com/).

---
weight: 70
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/troubleshooting/
feedback: LLMNET
version: 26.5.0
title: Troubleshooting
description: Diagnose and fix common Aspose.LLM for .NET problems — binary download failures, out-of-memory, GPU detection, model load, garbled output, license, performance.
keywords:
- troubleshooting
- errors
- debugging
- problems
- diagnostics
---

When the SDK misbehaves, the failure is usually in one of seven well-known buckets. This section covers each one with the same structure on every page: **Symptom** (what you see), **Cause** (what is happening), **Resolution** (how to fix), and optional **Prevention**.

Pick the page that matches your symptom from the [topics list](#topics). If none matches, use the [diagnostic flow](#diagnostic-flow) below to narrow the problem, then check the closest page, then ask for help.

## Pre-flight checks

Before diving into a specific page, confirm the basics. The majority of tickets sent to support turn out to be one of these:

- **License applied**: `Aspose.LLM.License.IsLicensed` returns `true` before any chat method is called. The SDK does not run inference in evaluation mode — see [License errors](/net/troubleshooting/license-errors/).
- **Debug logging on**: set `EngineParameters.EnableDebugLogging = true` and pass an `ILogger` to `AsposeLLMApi.Create(preset, logger)`. Native tagged lines (`[MM]`, `[CTX]`, `[KV]`) reveal where a failure happens. See [Logging and diagnostics](/net/developer-reference/logging-and-diagnostics/).
- **Known good preset**: reproduce with a built-in preset like `Qwen25Preset` before suspecting the SDK. Custom presets or manual overrides are the most common source of garbled output.
- **Minimal repro**: strip down to the smallest possible snippet that fails. If the minimal snippet passes, the problem is in your integration, not the SDK.

## Diagnostic flow

Walk this decision tree when the symptom is not obvious.

1. **Does `Create` return at all?**
   - No → binary download or model load failed. See [Binary download fails](/net/troubleshooting/binary-download-fails/) or [Model not loading](/net/troubleshooting/model-not-loading/).
   - Yes → step 2.

2. **Does the first chat call throw?**
   - `Not licensed for this method` → see [License errors](/net/troubleshooting/license-errors/).
   - Out-of-memory → see [Out of memory](/net/troubleshooting/out-of-memory/).
   - Other → capture the full stack trace and open a support ticket.

3. **Does chat return, but slowly?**
   - Much slower than expected → [Performance issues](/net/troubleshooting/performance-issues/) or [GPU not detected](/net/troubleshooting/gpu-not-detected/).
   - Slow only on first request → see first-token latency guidance in [Reduce first-token latency](/net/how-to/reduce-first-token-latency/).

4. **Does chat return, but output is wrong?**
   - Nonsense / literal marker tokens / repetition loops → [Garbled output](/net/troubleshooting/garbled-output/).
   - Truncated mid-sentence → raise `ChatParameters.MaxTokens`; see [Chat parameters](/net/developer-reference/parameters/chat/).

5. **Does memory grow across long sessions?**
   - Yes → tune cache cleanup — see [Cache management](/net/developer-reference/cache-management/) and [Out of memory](/net/troubleshooting/out-of-memory/).

## Symptom → page shortcut

| Symptom | Start here |
|---|---|
| `HttpRequestException` during `Create` | [Binary download fails](/net/troubleshooting/binary-download-fails/) |
| `InvalidOperationException` during model load | [Model not loading](/net/troubleshooting/model-not-loading/) |
| `cudaErrorOutOfMemory`, `OutOfMemoryException` | [Out of memory](/net/troubleshooting/out-of-memory/) |
| Inference runs at CPU speed despite a GPU present | [GPU not detected](/net/troubleshooting/gpu-not-detected/) |
| Replies contain `<image>`, `<|im_start|>`, etc. verbatim | [Garbled output](/net/troubleshooting/garbled-output/) |
| Output loops or repeats | [Garbled output](/net/troubleshooting/garbled-output/) |
| `Not licensed for this method` | [License errors](/net/troubleshooting/license-errors/) |
| Unexpected high first-token latency | [Performance issues](/net/troubleshooting/performance-issues/) |
| Throughput well below hardware expectation | [Performance issues](/net/troubleshooting/performance-issues/) |

## Topics

- [Binary download fails](/net/troubleshooting/binary-download-fails/) — `BinaryManager` cannot reach GitHub, TLS interception, disk space.
- [Out of memory](/net/troubleshooting/out-of-memory/) — GPU VRAM, system RAM, KV cache growth.
- [GPU not detected](/net/troubleshooting/gpu-not-detected/) — driver, CUDA version, `PreferredAcceleration`, container flags.
- [Model not loading](/net/troubleshooting/model-not-loading/) — corrupt GGUF, unsupported architecture, wrong file name.
- [Garbled output](/net/troubleshooting/garbled-output/) — template mismatch, repetition loops, truncation, vision misalignment.
- [License errors](/net/troubleshooting/license-errors/) — missing `SetLicense`, expired temporary license, embedded resource mis-naming.
- [Performance issues](/net/troubleshooting/performance-issues/) — low throughput, latency spikes, thread contention, thermal throttling.

## Asking for help

When none of the above match, or the fix does not stick, open a thread on the [Aspose Support Forum](https://forum.aspose.com/) with:

- SDK version (`Aspose.LLM` NuGet version).
- Host OS and architecture.
- GPU model and driver version (if applicable).
- Preset class name and any overrides you applied.
- Full log from a reproducing run with `EnableDebugLogging = true`.
- A minimal code sample that reproduces the issue.
- The expected versus actual output.

For paid support, use the [Aspose Helpdesk](https://helpdesk.aspose.com/).

## What's next

- [Logging and diagnostics](/net/developer-reference/logging-and-diagnostics/) — tagged log taxonomy and `parse_mm_logs.zsh` helper.
- [Debugging vision](/net/developer-reference/multimodal/debugging-vision/) — multimodal-specific diagnosis.
- [How-to recipes](/net/how-to/) — related task-focused guides.

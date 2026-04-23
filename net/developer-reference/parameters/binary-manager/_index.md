---
weight: 70
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/binary-manager/
feedback: LLMNET
version: 26.5.0
title: Binary manager parameters
description: Configure how Aspose.LLM for .NET downloads and caches native llama.cpp binaries — release tag, cache path, system specification, and preferred acceleration backend.
keywords:
- BinaryManagerParameters
- ReleaseTag
- BinaryPath
- PreferredAcceleration
- llama.cpp
- native binary
- acceleration
---

`BinaryManagerParameters` controls how the SDK obtains the native `llama.cpp` binaries (`libllama`, `libmtmd`, `libggml-*`). On first use, the engine downloads the matching build from GitHub for your platform and acceleration backend, caches it, and reuses the cache on subsequent runs.

## Class reference

```csharp
namespace Aspose.LLM.Abstractions.Parameters;

public class BinaryManagerParameters
{
    public string Owner { get; set; } = "ggml-org";
    public string Repo { get; set; } = "llama.cpp";
    public string ReleaseTag { get; set; } = "b8816";
    public string BinaryPath { get; set; }              // <LocalAppData>/Aspose.LLM/runtimes
    public SystemSpec? SystemSpecification { get; set; }
    public AccelerationType? PreferredAcceleration { get; set; }
}
```

## Fields

| Field | Type | Default | Purpose |
|---|---|---|---|
| `Owner` | `string` | `"ggml-org"` | GitHub repository owner for `llama.cpp` releases. |
| `Repo` | `string` | `"llama.cpp"` | GitHub repository name. |
| `ReleaseTag` | `string` | `"b8816"` (as of SDK v26.5.0) | Specific `llama.cpp` release to pin. |
| `BinaryPath` | `string` | `<LocalAppData>/Aspose.LLM/runtimes` | Local cache for downloaded native binaries. |
| `SystemSpecification` | `SystemSpec?` | `null` (auto-detect) | Override the detected OS / architecture / acceleration capabilities. |
| `PreferredAcceleration` | `AccelerationType?` | `null` (auto-select) | Force a specific acceleration backend (CUDA, HIP, Metal, Vulkan, CPU variants). |

### `Owner` and `Repo`

Together they form `github.com/<Owner>/<Repo>/releases/...`. The defaults target the upstream `llama.cpp` repository. Change them only if you mirror releases to a fork that stays byte-compatible with upstream — for example, in an air-gapped enterprise setup that syncs selected releases into a private GitHub Enterprise instance.

### `ReleaseTag`

Pins a specific upstream release. As of SDK v26.5.0, the default is `b8816`. Each release tag corresponds to native binaries with matching P/Invoke signatures; changing the tag without a matching SDK version is unsupported.

Override only when:

- You are testing a newer upstream release against the current SDK (development only).
- You are locked to an older release because of a validated deployment.

```csharp
preset.BinaryManagerParameters.ReleaseTag = "b8816";
```

{{% alert color="primary" %}}
The SDK's P/Invoke layer is validated against the default `ReleaseTag`. Pinning a different tag can produce runtime errors if upstream changed a struct layout or function signature. Do not ship custom tags to production without a migration pass — see the `llama-cpp-migration` workflow used by the Aspose team.
{{% /alert %}}

### `BinaryPath`

Folder where downloaded binaries live. The default is `<LocalAppData>/Aspose.LLM/runtimes` — `%LOCALAPPDATA%\Aspose.LLM\runtimes` on Windows and the equivalent `LocalApplicationData` folder elsewhere.

Override when:

- **Shared cache** across multiple applications or services on the same host.
- **Read-only root filesystem** — point the cache at a writable volume.
- **Pre-populated deployment** — bundle the binaries with your application and point `BinaryPath` at them to skip the download on first run.

```csharp
preset.BinaryManagerParameters.BinaryPath = @"/var/lib/aspose-llm/runtimes";
```

### `SystemSpecification`

When `null` (the default), the SDK detects the host's OS, architecture, and available accelerations at engine construction. Override with an explicit `SystemSpec` only for diagnostics or cross-platform binary preparation — leaving this `null` is correct for normal deployments.

### `PreferredAcceleration`

When `null`, the SDK picks the best available acceleration for the host in this order: CUDA → HIP → Metal → Vulkan → CPU (AVX level best to worst). Set an explicit value to override the selection.

Supported values (see `Aspose.LLM.Abstractions.Acceleration.AccelerationType`):

| Value | Platform | Notes |
|---|---|---|
| `CUDA` | Windows, Linux | NVIDIA GPUs. |
| `HIP` | Linux | AMD GPUs via ROCm. |
| `Metal` | macOS (Apple Silicon) | M-series chips. |
| `Vulkan` | Windows, Linux | Cross-platform GPU. |
| `AVX512` | Any x64 with AVX-512 | Fastest CPU path. |
| `AVX2` | Any x64 with AVX2 | Default CPU fallback. |
| `AVX` | Older x64 | Slower. |
| `NoAVX` | Very old CPUs | Last-resort compatibility. |
| `Kompute`, `OpenCL`, `SYCL`, `OpenBLAS` | Platform-dependent | Less common; verify availability for your target. |

The enum has additional values (`None`) used internally — avoid setting them explicitly.

## Typical recipes

### Force CPU-only execution

```csharp
using Aspose.LLM.Abstractions.Acceleration;

var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.AVX2;
preset.BaseModelInferenceParameters.GpuLayers = 0; // complement on the inference side
```

### Force CUDA on a multi-GPU box

```csharp
var preset = new Qwen25Preset();
preset.BinaryManagerParameters.PreferredAcceleration = AccelerationType.CUDA;
preset.BaseModelInferenceParameters.GpuLayers = 999;
// Pick a specific GPU via BaseModelInferenceParameters.MainGpu = N;
```

### Pre-populated offline deployment

Download the binaries on a connected machine, copy the cache into your deployment, and point `BinaryPath` at it on the offline host:

```csharp
var preset = new Qwen25Preset();
preset.BinaryManagerParameters.BinaryPath = @"/opt/aspose-llm/runtimes";
// Also point EngineParameters.ModelCachePath at a pre-populated model cache.
```

### Shared cache across services

```csharp
preset.BinaryManagerParameters.BinaryPath = @"/srv/shared/aspose-llm/runtimes";
```

Make sure every service using this cache runs the same SDK version — version mismatches produce binary incompatibilities.

## What's next

- [System requirements](/net/system-requirements/) — what runtimes and hardware the binaries support.
- [Model inference parameters](/net/developer-reference/parameters/model-inference/) — complement `PreferredAcceleration` with `GpuLayers` and split settings.
- [Architecture](/net/product-overview/architecture/) — what happens during first-run binary deployment.

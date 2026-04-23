---
weight: 10
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/troubleshooting/binary-download-fails/
feedback: LLMNET
version: 26.5.0
title: Binary download fails
description: Diagnose why Aspose.LLM for .NET fails to download native llama.cpp binaries on first Create — proxy, firewall, rate limits, disk space, SSL.
keywords:
- binary download
- GitHub
- proxy
- firewall
- rate limit
- BinaryManager
---

On first `AsposeLLMApi.Create`, the SDK downloads native `llama.cpp` binaries from GitHub. In corporate or restricted environments, the download can fail. This page covers the common causes.

## Symptom

- `HttpRequestException` or a wrapped `InvalidOperationException` during `Create`.
- Log entries mentioning GitHub, `BinaryManager`, or HTTP status codes (403, 404, 429, 500).
- `Create` hangs for a long time and eventually times out.

## Cause

The SDK contacts `github.com/ggml-org/llama.cpp/releases/tag/<ReleaseTag>` to list release assets, then downloads the asset matching your platform and acceleration. Any of these can block:

- Corporate firewall or proxy blocks github.com.
- TLS interception strips the expected certificate chain.
- GitHub rate-limit (HTTP 429) on unauthenticated requests.
- The requested `ReleaseTag` does not exist on upstream.
- Disk space insufficient for the download or extraction.

## Resolution

### 1. Verify network access

Manually confirm GitHub is reachable from the host:

```bash
curl -I https://api.github.com/repos/ggml-org/llama.cpp/releases/tags/b8816
# Expect HTTP/2 200
```

If the curl fails, the problem is network; fix that first (proxy config, firewall rule).

### 2. Configure an HTTP proxy

On Windows, set `HTTP_PROXY` and `HTTPS_PROXY` environment variables before starting the process. On Linux / macOS, export them:

```bash
export HTTPS_PROXY=http://proxy.example.com:8080
dotnet run
```

.NET's default `HttpClient` honors these variables.

### 3. Pre-populate the cache

If the host cannot reach GitHub at all, pre-download on a machine with internet access and copy the cache to the target. Full workflow: [Offline deployment](/net/use-cases/offline-deployment/).

### 4. Check the `ReleaseTag`

Confirm `BinaryManagerParameters.ReleaseTag` matches a real upstream release:

```csharp
preset.BinaryManagerParameters.ReleaseTag = "b8816"; // default in SDK v26.5.0
```

Visit `https://github.com/ggml-org/llama.cpp/releases/tag/<tag>` in a browser to verify.

### 5. Free up disk space

Binary archives are 100-500 MB; extracted trees are similar. Ensure at least 1-2 GB free at `BinaryManagerParameters.BinaryPath`:

```bash
# Linux
df -h ~/.local/share/Aspose.LLM/runtimes

# Windows
# Check %LOCALAPPDATA%\Aspose.LLM\runtimes free space.
```

### 6. Handle GitHub rate limits

If logs mention HTTP 429, you are hitting GitHub's unauthenticated API limit (60 requests/hour per IP). Options:

- Wait and retry.
- Use an authenticated `HttpClient` (advanced — requires a custom `IModelFileProvider` in the extensibility layer).
- Pre-populate the cache so subsequent runs do not hit the API.

### 7. TLS interception

Corporate TLS-inspection proxies replace GitHub's certificate with a corporate one. .NET by default rejects that.

Options (choose one):

- Install the corporate root certificate into the host's certificate store.
- Bypass interception for `*.github.com` and `*.githubusercontent.com` on the proxy.

Do **not** disable TLS validation in production — it is a security regression.

## Prevention

- **For production**: always pre-populate caches in your deployment pipeline. Do not rely on first-run downloads in production environments.
- **For development**: keep `BinaryPath` and `ModelCachePath` on a shared network drive across your team so downloads happen once per team, not once per developer.
- **Pin `ReleaseTag`** explicitly in your preset — do not rely on the default across SDK upgrades.

## What's next

- [Offline deployment](/net/use-cases/offline-deployment/) — full pre-population workflow.
- [Binary manager parameters](/net/developer-reference/parameters/binary-manager/) — `BinaryPath`, `ReleaseTag`, `PreferredAcceleration`.
- [Architecture](/net/product-overview/architecture/) — the binary deployment stage.

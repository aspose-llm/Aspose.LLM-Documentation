---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/troubleshooting/model-not-loading/
feedback: LLMNET
version: 26.5.0
title: Model not loading
description: Fix Aspose.LLM for .NET errors during model load — corrupt GGUF, unsupported architecture, wrong file, disk space, permission issues.
keywords:
- model loading
- GGUF
- corrupt file
- architecture
- InvalidOperationException
- model load error
---

The SDK fails during model load — on the path from `AsposeLLMApi.Create` that downloads and initializes the GGUF file. This page covers the usual causes.

## Symptom

- `InvalidOperationException` during `Create`, often wrapping a lower-level error.
- Log messages mentioning "failed to load model", "invalid magic number", or a specific `llama_load_*` function.
- Native segfaults or access-violation exceptions during load (rare).
- Download completes but subsequent load fails.

## Cause

Several distinct failure modes:

- **Corrupted download** — partial or interrupted download; bad cached file.
- **Unsupported model architecture** — a GGUF whose architecture is not supported by the pinned `ReleaseTag`.
- **Wrong file name** — the file exists but does not match the expected quantization.
- **Disk or permission issues** — the cache directory is not writable.
- **`llama.cpp` release mismatch** — a tag older than the model's architecture.

## Resolution

### 1. Verify the cached file

Look at `EngineParameters.ModelCachePath` (default `<LocalAppData>/Aspose.LLM/models`) for the file the preset references. Check its size against the Hugging Face listing:

```bash
ls -la ~/.local/share/Aspose.LLM/models
```

If the size is far below the expected value, the download was truncated.

### 2. Delete the cached file and retry

Clear the partial/corrupt file and let the SDK re-download:

```bash
rm ~/.local/share/Aspose.LLM/models/Qwen2.5-7B-Instruct-Q4_K_M.gguf
```

Re-run your program. The SDK downloads the file again.

### 3. Validate the GGUF

Enable `CheckTensors` on the inference parameters to validate every tensor during load:

```csharp
preset.BaseModelInferenceParameters.CheckTensors = true;
```

Start-up takes longer, but you get clear errors on malformed tensors. If validation fails, the file is corrupt — delete and re-download.

Disable `CheckTensors` in production after confirming the file is good.

### 4. Confirm the model is supported

`llama.cpp` supports a fixed set of architectures per release. A brand-new model might not be supported by the default `ReleaseTag = "b8816"`. Check:

- The architecture name in the model's Hugging Face README (e.g., "Qwen2", "Llama", "Gemma", "Phi").
- The `llama.cpp` release notes for the tag you are using.

If the architecture is newer than the tag supports:

- Switch to a newer `ReleaseTag` if one exists (tested and validated by the Aspose team).
- Fall back to a comparable model with a supported architecture.
- File a [support request](https://forum.aspose.com/) if the architecture is critical for your use case.

### 5. Confirm the file name is correct

Hugging Face repos often have many quantization variants. The preset's default file name matches one of them; if the file has been removed or renamed upstream, the download fails.

Check the repo:

```
https://huggingface.co/bartowski/Qwen2.5-7B-Instruct-GGUF/tree/main
```

Verify `BaseModelSourceParameters.HuggingFaceFileName` matches an existing file. Override if needed:

```csharp
preset.BaseModelSourceParameters.HuggingFaceFileName = "Qwen2.5-7B-Instruct-Q5_K_M.gguf";
```

### 6. Check directory permissions

On Linux / macOS, confirm write access to the cache folder:

```bash
ls -la ~/.local/share/Aspose.LLM/
# Expect the folder to be owned by the user running the process.
```

On Windows, check folder ACLs — especially when the process runs under a service account different from the install user.

### 7. Check disk space

Large models (20B+) need 10-20+ GB free at both the cache location and temp directories used during extraction.

```bash
df -h ~/.local/share/Aspose.LLM/models
```

## Vision-specific: mmproj

Vision presets load both the base model and a projector. If base loads but projector fails, the failure message mentions `mmproj` or `mtmd_init_from_file`. Apply the same checks to `MmprojSourceParameters`:

```csharp
preset.MmprojSourceParameters.HuggingFaceFileName = "mmproj-F16.gguf";
```

## Prevention

- Pre-download and validate models in your CI / build pipeline. Failing early in CI beats failing at runtime.
- Commit manifest files that record the expected model hash alongside the preset selection. Compare on load.
- Pin a tested `ReleaseTag` — do not float on defaults across SDK upgrades without testing.

## What's next

- [Model source parameters](/net/developer-reference/parameters/model-source/) — priority and resolution.
- [Supported presets](/net/product-overview/supported-presets/) — confirmed compatible models and files.
- [Binary download fails](/net/troubleshooting/binary-download-fails/) — related network issues.

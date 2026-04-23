---
weight: 60
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/troubleshooting/license-errors/
feedback: LLMNET
version: 26.5.0
title: License errors
description: Fix license-related failures in Aspose.LLM for .NET — Not licensed for this method, expired temporary license, wrong file path, embedded resource not found.
keywords:
- license error
- Not licensed
- IsLicensed
- expired
- temporary license
- SetLicense
---

License errors appear when a chat method is called without a valid license. Aspose.LLM does not have an evaluation fallback for inference — every chat operation requires a license.

## Symptom

The chat APIs throw:

```
System.Exception: Not licensed for this method
```

Or `License.IsLicensed` returns `false` when you expected `true`.

## Cause

Several distinct cases:

- `SetLicense` was not called before the chat method.
- `SetLicense` threw, and your code continued past the failure.
- The license file path does not resolve to an actual file.
- The embedded resource name is wrong.
- The license is expired (especially temporary licenses, typically 30 days).
- The license is corrupted (partial download, file system damage).

## Resolution

### 1. Confirm `SetLicense` was called

Every process that calls chat methods must apply the license once. A common mistake is calling `SetLicense` in one context (e.g., a helper class constructor) and the API in another (a different process).

```csharp
var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

// Immediately after, verify:
Console.WriteLine($"IsLicensed: {Aspose.LLM.License.IsLicensed}");
// Should print True. If False, SetLicense failed.
```

### 2. Catch exceptions from `SetLicense`

`SetLicense` itself can throw when the file is missing, corrupt, or wrong format. Catch and log:

```csharp
try
{
    var license = new Aspose.LLM.License();
    license.SetLicense("Aspose.LLM.lic");
}
catch (Exception ex)
{
    _logger.LogError(ex, "License could not be applied.");
    throw;
}
```

The exception message usually states the cause (file not found, parse error, signature check failed).

### 3. Verify the license file path

When you pass only a file name, `SetLicense` searches several locations (see [Licensing](/net/licensing/)). If the file lives elsewhere, pass the full path:

```csharp
license.SetLicense(@"C:\licenses\Aspose.LLM.lic");
```

Confirm the file is copied to the process working directory or bin folder during build:

```xml
<ItemGroup>
  <None Update="Aspose.LLM.lic">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </None>
</ItemGroup>
```

### 4. Embedded resource — check the name

For embedded licenses, the resource name must match the file name exactly:

```xml
<ItemGroup>
  <EmbeddedResource Include="Aspose.LLM.lic" />
</ItemGroup>
```

```csharp
license.SetLicense("Aspose.LLM.lic"); // matches the file name
```

If the file is in a subfolder (e.g., `Resources/Aspose.LLM.lic`), the resource name becomes `<Namespace>.Resources.Aspose.LLM.lic`. Match that name exactly, or move the file to the project root.

### 5. Temporary license expired

Temporary licenses issued by [purchase.aspose.com/temporary-license](https://purchase.aspose.com/temporary-license) have a fixed expiry date (typically 30 days). After expiry, chat methods throw the same `Not licensed for this method` error.

Options:

- Request a new temporary license.
- Purchase a commercial license.

The application code does not change — swap the `.lic` file.

### 6. Corrupt license file

If the file is truncated or modified after issue, signature validation fails. Re-download from the Aspose purchase portal.

### 7. Stream-based license from a failed source

If you load via a stream:

```csharp
using var stream = File.OpenRead("Aspose.LLM.lic");
license.SetLicense(stream);
```

Ensure the stream is at position 0 before `SetLicense`. If an earlier read consumed bytes, `SetLicense` sees truncated data.

## Prevention

- Apply the license at application startup, once, with explicit exception handling.
- Log `License.IsLicensed` immediately after `SetLicense` to confirm.
- Monitor temporary license expiry dates — have a calendar reminder 7 days before expiry.
- In CI/CD, use an embedded license or pull from a secret store rather than bundling files.
- For air-gapped deployments, copy the license with the rest of the deployment artifacts.

## What's next

- [Licensing](/net/licensing/) — full license setup (file, stream, embedded resource, temporary).
- [License class reference](/net/developer-reference/license/) — API surface of `License`.
- [AsposeLLMApi facade](/net/developer-reference/asposellmapi/) — where license checks sit in the chat API surface.

---
weight: 30
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/licensing/
feedback: LLMNET
aliases:
- /net/evaluate-aspose-llm/
version: 26.5.0
title: Licensing
description: Apply an Aspose.LLM for .NET license from a file, stream, or embedded resource, and understand evaluation-mode limits.
keywords:
- license
- evaluation
- trial
- SetLicense
- IsLicensed
- embedded resource
---

Aspose.LLM for .NET is a commercially licensed product. Apply a license once per application process before calling chat APIs.

## Evaluation limits

Unlike some other Aspose products, Aspose.LLM does **not** run inference in evaluation mode. Calling `StartNewChatAsync`, `SendMessageAsync`, or `SendMessageToSessionAsync` without an applied license throws:

```
System.Exception: Not licensed for this method
```

To evaluate the SDK, request a free [temporary license](https://purchase.aspose.com/temporary-license) and apply it as shown below.

Non-inference operations — constructing the API, loading the model, and disposing — run without a license. Use them to validate your environment setup (NuGet install, binary download, model resolution) before committing to a license.

## Apply a license

Apply the license once per process, before calling any chat API. Typical placement: application startup (`Program.cs`, `Startup.cs`, or a DI bootstrap).

### From a file

```csharp
var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");
```

If you pass only the file name, the loader searches:

1. The folder of the Aspose.LLM assembly.
2. The folder of the calling assembly.
3. The folder of the entry assembly.
4. The current working directory.

You can also pass a full path.

### From a stream

```csharp
using var stream = new FileStream("Aspose.LLM.lic", FileMode.Open, FileAccess.Read);
var license = new Aspose.LLM.License();
license.SetLicense(stream);
```

Use a stream when the license is not a plain file — for example, when it comes from an HTTP response, a secret store, or a resource decrypted at runtime.

### From an embedded resource

Embed the `.lic` file in your assembly, then call `SetLicense` with the resource name.

1. Add the `.lic` file to your project.
2. In the project file, mark it as an embedded resource:

   ```xml
   <ItemGroup>
     <EmbeddedResource Include="Aspose.LLM.lic" />
   </ItemGroup>
   ```

3. Call `SetLicense` with the file name:

   ```csharp
   var license = new Aspose.LLM.License();
   license.SetLicense("Aspose.LLM.lic");
   ```

   The `License` class searches embedded resources of the calling assembly automatically.

## Check license status

```csharp
bool isLicensed = Aspose.LLM.License.IsLicensed;
```

Use this to verify that a license has been applied — for example, to log at startup or to guard optional features that require a commercial license.

## Temporary license

A temporary license is a full-functionality license valid for a limited period — typically 30 days — issued free of charge for evaluation, proof of concept, or pre-purchase testing. While active, it removes evaluation-mode restrictions exactly as a commercial license does.

### When to request a temporary license

- You want to evaluate Aspose.LLM for .NET end-to-end before purchasing.
- You are building a proof of concept and need unrestricted access to chat APIs.
- You are benchmarking the SDK against alternatives.
- Your commercial license has expired while a renewal is in progress.

Aspose.LLM does not run inference in evaluation mode, so a license — commercial or temporary — is required to send any chat messages.

### Request a temporary license

1. Go to [purchase.aspose.com/temporary-license](https://purchase.aspose.com/temporary-license).
2. Fill in the request form. You need an Aspose account; create one at no cost if you do not have it.
3. Describe your evaluation goal in a few sentences.
4. Submit the form.
5. Aspose emails you a `.lic` file. Delivery is usually within a business day.

### Apply a temporary license

Temporary licenses use the same `License.SetLicense` call as commercial ones. No code changes are needed when you switch between temporary and commercial.

```csharp
var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.Temporary.lic");
```

All three methods from the [Apply a license](#apply-a-license) section work — file path, stream, or embedded resource. Pick whatever fits your deployment.

### Limitations

- **Time-limited.** The license embeds an expiration date. After expiry, chat APIs throw the same `Not licensed for this method` exception as with no license. Request a new temporary license or purchase a commercial one before the current one expires.
- **Evaluation scope.** Temporary licenses are for evaluation and internal testing, not for production use. Consult the [Aspose End User License Agreement](https://company.aspose.com/legal/eula) for the exact terms.
- **Same API functionality.** No feature is held back in a temporary license — the API behaves identically to a commercial license during the validity period.

### Renew or replace

- To extend an evaluation, request a new temporary license via the same form. Replace the old `.lic` file with the new one and restart your application (the license is applied once per process).
- To move to production, purchase a commercial license at [purchase.aspose.com](https://purchase.aspose.com/). Swap the `.lic` file — code does not change.

## What's next

- [Hello, world!](/net/hello-world/) — run the first licensed example.
- [License reference](/net/developer-reference/license/) — the `License` class in the developer reference.
- [Installation](/net/installation/) — if you have not yet installed the package.

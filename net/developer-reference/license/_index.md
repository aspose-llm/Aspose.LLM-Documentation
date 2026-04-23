---
weight: 40
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/license/
feedback: LLMNET
version: 26.5.0
title: License class
description: API reference for the Aspose.LLM.License class — SetLicense methods and IsLicensed property.
keywords:
- license
- License class
- SetLicense
- IsLicensed
- API
---

`Aspose.LLM.License` is the class used to apply and check the product license at runtime. This page covers the API surface; for the process of obtaining and deploying a license (including the free temporary license), see [Licensing](/net/licensing/).

## Class reference

```csharp
namespace Aspose.LLM;

public class License
{
    public License();
    public void SetLicense(string licensePath);
    public void SetLicense(Stream licenseStream);
    public static bool IsLicensed { get; }
}
```

## Apply a license

Apply the license once per process, before the first inference call.

### From a file

```csharp
var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");
```

If you pass a file name without a directory, the loader searches:

1. The folder of the Aspose.LLM assembly.
2. The folder of the calling assembly.
3. The folder of the entry assembly.
4. The current working directory.
5. Embedded resources in the calling assembly.

Pass a full path to bypass this search.

### From a stream

```csharp
using var stream = new FileStream("Aspose.LLM.lic", FileMode.Open, FileAccess.Read);
var license = new Aspose.LLM.License();
license.SetLicense(stream);
```

Use a stream for licenses coming from secret stores, HTTP responses, or encrypted storage.

## Check license status

```csharp
bool isLicensed = Aspose.LLM.License.IsLicensed;
```

`IsLicensed` is a static property. It returns `true` when a valid license is applied to the current process and `false` otherwise (including when the license has expired).

## Effect of licensing on the API

- **Without a license** — `StartNewChatAsync`, `SendMessageAsync`, and `SendMessageToSessionAsync` throw `Exception("Not licensed for this method")`.
- **With a valid license** — all methods work normally.
- **With an expired temporary license** — same behavior as without a license.

Non-inference operations (`AsposeLLMApi.Create`, `Dispose`, `GetDefaultPreset`) work regardless of license state.

## What's next

- [Licensing](/net/licensing/) — how to obtain and deploy a license, including embedded resources and temporary licenses.
- [Hello, world!](/net/hello-world/) — first runnable example with licensing.

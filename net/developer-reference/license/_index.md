---
weight: 40
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/developer-reference/license/
feedback: LLMNET
title: License
description: Using the License class in Aspose.LLM for .NET.
keywords:
- license
- SetLicense
- IsLicensed
---

The `Aspose.LLM.License` class is used to apply and check the product license.

## Applying a license

Apply the license once per process before using `AsposeLLMApi`:

```csharp
var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");
```

You can pass a file path or a stream. See [Licensing](/net/licensing/) in Getting started for file, stream, and embedded-resource options.

## Checking license status

```csharp
bool isLicensed = Aspose.LLM.License.IsLicensed;
```

Use this to verify that a license has been applied (e.g. for logging or conditional behavior). The API will throw if you call methods that require a license when none is set; see the product documentation for evaluation limitations.

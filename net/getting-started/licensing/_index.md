---
weight: 30
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/licensing/
feedback: LLMNET
aliases:
- /net/evaluate-aspose-llm/
title: Licensing
description: Apply Aspose.LLM for .NET license and use evaluation mode.
keywords:
- license
- evaluation
- trial
---

Aspose.LLM for .NET is a commercially licensed product. You can run it in evaluation (trial) mode or apply a license to remove restrictions.

## Evaluation mode

Without a license, the component runs in evaluation mode. Functionality may be limited (e.g. watermarks or usage caps). Apply a license to use the product without these limitations.

## Applying a license

Apply the license once per application process, before calling the Aspose.LLM API (e.g. in startup or before `AsposeLLMApi.Create`).

### From file

```csharp
Aspose.LLM.License license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");
```

If you pass only the file name, the loader looks for the file in the folder of the component, the calling assembly, and the entry assembly. You can also pass a full path.

### From stream

```csharp
using (var stream = new FileStream("Aspose.LLM.lic", FileMode.Open, FileAccess.Read))
{
    var license = new Aspose.LLM.License();
    license.SetLicense(stream);
}
```

### Embedded resource

Embed the `.lic` file as an embedded resource in your assembly, then call `SetLicense` with the file name. The license class will search embedded resources of the calling assembly.

## Checking license status

```csharp
bool isLicensed = Aspose.LLM.License.IsLicensed;
```

## Temporary license

You can request a [temporary license](https://purchase.aspose.com/temporary-license) for evaluation. Purchase a full license when you are ready to deploy.

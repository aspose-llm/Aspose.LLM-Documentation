---
weight: 20
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/installation/
feedback: LLMNET
title: Installation
description: Add Aspose.LLM for .NET to your project.
keywords:
- install
- NuGet
- package
---

**Aspose.LLM for .NET** is distributed as a NuGet package. You also need the **Aspose.LLM.Abstractions** package for presets and parameter types (it is usually pulled in as a dependency of Aspose.LLM).

## NuGet

1. Open your solution in Visual Studio (or your IDE).
2. Add the **Aspose.LLM** package to the project:
   - **Package Manager UI**: Browse for "Aspose.LLM" and install.
   - **Package Manager Console**: run  
     `Install-Package Aspose.LLM`
3. Restore packages. The Aspose.LLM and Aspose.LLM.Abstractions assemblies will be available to your project.

To update to a newer version, use the NuGet UI or run `Update-Package Aspose.LLM` in the Package Manager Console.

## After installation

- Reference `Aspose.LLM` and `Aspose.LLM.Abstractions` in your code.
- Apply a [license](/net/licensing/) before using the API (or run in evaluation mode).
- Use a [preset](/net/product-overview/supported-presets/) to create an `AsposeLLMApi` instance and run the [Hello, world!](/net/hello-world/) example.

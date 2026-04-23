---
weight: 60
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/use-cases/batch-processing/
feedback: LLMNET
version: 26.5.0
title: Batch processing
description: Run many prompts through one loaded Aspose.LLM model — amortize the load cost, pick session-per-prompt vs shared-session patterns.
keywords:
- batch processing
- bulk
- many prompts
- amortize
- throughput
---

Batch processing runs many prompts through the same `AsposeLLMApi` instance without paying the model-load cost for each one. This is the right pattern for classification, tagging, summarization, or any bulk task over a fixed set of inputs.

## When to use this pattern

- Process a folder of documents with the same prompt template.
- Score a dataset with the same classifier.
- Run regression evaluations over a test set.
- Periodic offline batches (nightly job, weekly report).

## Prerequisites

- [Install the NuGet package](/net/installation/).
- [Apply a license](/net/licensing/).

## Two patterns

### Pattern A — fresh session per prompt

Each prompt starts in isolation, no history from earlier prompts. Good for **independent** tasks like classification or per-item summarization.

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Parameters.Presets;

var license = new Aspose.LLM.License();
license.SetLicense("Aspose.LLM.lic");

var preset = new Qwen25Preset();
preset.ChatParameters.SystemPrompt =
    "Classify the email sentiment. Return exactly one word: positive, negative, or neutral.";
preset.ChatParameters.MaxTokens = 10;
preset.SamplerParameters.Temperature = 0.1f;

using var api = AsposeLLMApi.Create(preset);

string[] emails = ReadEmails();
var results = new List<(string Email, string Label)>();

foreach (string email in emails)
{
    // Fresh session per prompt — no history carryover.
    string sessionId = await api.StartNewChatAsync();
    string label = await api.SendMessageToSessionAsync(sessionId, email);
    results.Add((email, label.Trim()));
}
```

### Pattern B — reuse one session with forced cleanup

Process many prompts through a single session, clearing its KV cache between prompts. Avoids the session-creation overhead; useful when you have very many short prompts.

```csharp
using Aspose.LLM.Abstractions.Models;

using var api = AsposeLLMApi.Create(preset);
string sessionId = await api.StartNewChatAsync();

foreach (string email in emails)
{
    string label = await api.SendMessageToSessionAsync(sessionId, email);
    results.Add((email, label.Trim()));

    // Reset cache — each prompt sees only the system prompt.
    api.ForceCacheCleanup(CacheCleanupStrategy.KeepSystemPromptOnly);
}
```

{{% alert color="primary" %}}
Pattern B is an optimization over Pattern A for very short prompts. For long prompts or cases where session boundaries matter, stick with Pattern A.
{{% /alert %}}

## Progress and cancellation

For long batches, wire a progress reporter and a cancellation token:

```csharp
using var cts = new CancellationTokenSource();
Console.CancelKeyPress += (_, e) => { e.Cancel = true; cts.Cancel(); };

int total = emails.Length;
int done = 0;

foreach (string email in emails)
{
    if (cts.IsCancellationRequested) break;

    string sessionId = await api.StartNewChatAsync();
    try
    {
        string label = await api.SendMessageToSessionAsync(
            sessionId, email, cancellationToken: cts.Token);
        results.Add((email, label.Trim()));
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("Cancelled.");
        break;
    }

    done++;
    if (done % 10 == 0)
        Console.WriteLine($"Progress: {done}/{total}");
}
```

## Memory across a long batch

Long batches accumulate KV cache across sessions. Force cleanup periodically:

```csharp
int processedSinceCleanup = 0;

foreach (string email in emails)
{
    // ... process
    processedSinceCleanup++;

    if (processedSinceCleanup >= 100)
    {
        api.ForceCacheCleanup(CacheCleanupStrategy.KeepSystemPromptOnly);
        processedSinceCleanup = 0;
    }
}
```

Or restart the `AsposeLLMApi` every N prompts for aggressive cleanup — dispose and recreate inside a loop, but budget 5-15 seconds for the restart.

## Parallelism caveats

`AsposeLLMApi` is **single-instance per process** and serialization at the native layer means concurrent inference calls on the same instance are not safe. Two ways to parallelize:

- **Multi-process**: split the batch across processes, one `AsposeLLMApi` per process. Each process loads the model once and works on its slice.
- **Pipeline queue**: one worker thread for inference, multiple producers feeding prompts. The worker serves prompts sequentially.

Native `llama.cpp` can internally batch multiple sequences in one forward pass, but Aspose.LLM's public chat API dispatches serialized requests — the gain from manual parallelism is mostly in host-side work (I/O, post-processing), not in inference itself.

## Full example

```csharp
using Aspose.LLM;
using Aspose.LLM.Abstractions.Models;
using Aspose.LLM.Abstractions.Parameters.Presets;

internal class BatchDemo
{
    public static async Task Main()
    {
        var license = new Aspose.LLM.License();
        license.SetLicense("Aspose.LLM.lic");

        var preset = new Qwen25Preset();
        preset.ChatParameters.SystemPrompt =
            "Summarize the paragraph in one short sentence.";
        preset.ChatParameters.MaxTokens = 64;
        preset.SamplerParameters.Temperature = 0.3f;

        using var api = AsposeLLMApi.Create(preset);
        string sessionId = await api.StartNewChatAsync();

        string[] paragraphs = ReadParagraphsFromFile("input.txt");
        using var writer = new StreamWriter("summaries.tsv");

        int done = 0;
        var sw = System.Diagnostics.Stopwatch.StartNew();

        foreach (string paragraph in paragraphs)
        {
            string summary = await api.SendMessageToSessionAsync(sessionId, paragraph);
            writer.WriteLine($"{done}\t{summary.Replace('\t', ' ').Trim()}");
            api.ForceCacheCleanup(CacheCleanupStrategy.KeepSystemPromptOnly);
            done++;

            if (done % 50 == 0)
            {
                double rate = done / sw.Elapsed.TotalSeconds;
                Console.WriteLine($"Progress: {done}/{paragraphs.Length} ({rate:F1}/s)");
            }
        }

        Console.WriteLine($"Done in {sw.Elapsed}.");
    }

    private static string[] ReadParagraphsFromFile(string path) =>
        File.ReadAllText(path).Split("\n\n", StringSplitOptions.RemoveEmptyEntries);
}
```

## Tuning for throughput

- **Small, task-specific system prompts** — long system prompts multiply the per-call fixed cost.
- **Low `MaxTokens`** for short outputs — faster and cheaper.
- **Deterministic sampling** (`Temperature = 0.0` or low, fixed `Seed`) — useful for eval runs.
- **GPU offload** — batch jobs benefit the most from GPU; a 20× speed-up is common versus CPU.

## What's next

- [Multiple concurrent sessions](/net/use-cases/multiple-concurrent-sessions/) — when sessions represent separate users.
- [Cache management](/net/developer-reference/cache-management/) — strategy details.
- [Chat parameters](/net/developer-reference/parameters/chat/) — `MaxTokens` and `CacheCleanupStrategy`.

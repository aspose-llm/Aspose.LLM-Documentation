---
weight: 50
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/sampler/
feedback: LLMNET
version: 26.5.0
title: Sampler parameters
description: Configure token sampling behavior in Aspose.LLM for .NET — temperature, top-p, top-k, min-p, repetition and presence penalties, DRY, XTC, dynamic temperature, Mirostat, seed, and logit bias.
keywords:
- SamplerParameters
- temperature
- top-p
- top-k
- min-p
- repetition penalty
- DRY
- XTC
- Mirostat
- dynatemp
- seed
- logit bias
---

`SamplerParameters` controls how the engine picks each next token during generation. It exposes the full sampler surface of `llama.cpp` common parameters: the classic temperature and nucleus sampling knobs, repetition penalties, specialized filters (DRY, XTC, dynatemp, Mirostat), reproducibility controls, and fine-grained logit bias.

Every built-in preset sets sensible defaults for its model family. Override specific fields before `AsposeLLMApi.Create` to tune behavior.

## Class reference

```csharp
namespace Aspose.LLM.Abstractions.Parameters;

public class SamplerParameters
{
    // Core sampling
    public float Temperature { get; set; } = 0.7f;
    public float TopP { get; set; } = 0.9f;
    public float TopK { get; set; } = 40;
    public float MinP { get; set; } = 0.05f;

    // Reproducibility
    public int Seed { get; set; } = unchecked((int)0xFFFFFFFF); // time-based
    public int MinKeep { get; set; } = 1;

    // Repetition controls
    public int PenaltyContextSize { get; set; } = -1; // -1 = full context
    public float RepetitionPenalty { get; set; } = 1.1f;
    public float PresencePenalty { get; set; } = 0f;
    public float FrequencyPenalty { get; set; } = 0f;

    // Advanced filters
    public float TypicalP { get; set; } = -1.0f;   // disabled if <= 0
    public float TopNSigma { get; set; } = -1.0f;  // disabled if <= 0

    // Dynamic temperature
    public float DynatempRange { get; set; } = 0.0f; // 0 disables
    public float DynatempExponent { get; set; } = 1.0f;

    // XTC (Exclude Top Choices)
    public float XtcProbability { get; set; } = -1.0f; // disabled if <= 0
    public float XtcThreshold { get; set; } = 0.0f;

    // DRY (Don't Repeat Yourself)
    public float DryMultiplier { get; set; } = -1.0f; // disabled if <= 0
    public float DryBase { get; set; } = 1.75f;
    public int DryAllowedLength { get; set; } = 2;
    public int DryPenaltyLastN { get; set; } = 0;
    public List<string> DrySequenceBreakers { get; set; } = new() { "\n", ":", "\"", "*" };

    // Mirostat
    public int Mirostat { get; set; } = 0; // 0=off, 1=Mirostat 1.0, 2=Mirostat 2.0
    public float MirostatTau { get; set; } = 5.0f;
    public float MirostatEta { get; set; } = 0.1f;

    // Fine-grained controls
    public Dictionary<int, float> LogitBias { get; set; } = new();
    public bool EnableInfill { get; set; } = false;
}
```

## Detailed field reference

Each field has a dedicated page with full defaults, scenario tables, code examples, and interactions. The rest of this page is an inline overview of the same content; follow the links for the deeper treatment.

**Core sampling**: [Temperature](/net/developer-reference/parameters/sampler/temperature/), [TopP](/net/developer-reference/parameters/sampler/top-p/), [TopK](/net/developer-reference/parameters/sampler/top-k/), [MinP](/net/developer-reference/parameters/sampler/min-p/).

**Reproducibility**: [Seed](/net/developer-reference/parameters/sampler/seed/), [MinKeep](/net/developer-reference/parameters/sampler/min-keep/).

**Repetition controls**: [PenaltyContextSize](/net/developer-reference/parameters/sampler/penalty-context-size/), [RepetitionPenalty](/net/developer-reference/parameters/sampler/repetition-penalty/), [PresencePenalty](/net/developer-reference/parameters/sampler/presence-penalty/), [FrequencyPenalty](/net/developer-reference/parameters/sampler/frequency-penalty/).

**Advanced filters**: [TypicalP](/net/developer-reference/parameters/sampler/typical-p/), [TopNSigma](/net/developer-reference/parameters/sampler/top-n-sigma/).

**Dynamic temperature**: [DynatempRange](/net/developer-reference/parameters/sampler/dynatemp-range/), [DynatempExponent](/net/developer-reference/parameters/sampler/dynatemp-exponent/).

**XTC (Exclude Top Choices)**: [XtcProbability](/net/developer-reference/parameters/sampler/xtc-probability/), [XtcThreshold](/net/developer-reference/parameters/sampler/xtc-threshold/).

**DRY (Don't Repeat Yourself)**: [DryMultiplier](/net/developer-reference/parameters/sampler/dry-multiplier/), [DryBase](/net/developer-reference/parameters/sampler/dry-base/), [DryAllowedLength](/net/developer-reference/parameters/sampler/dry-allowed-length/), [DryPenaltyLastN](/net/developer-reference/parameters/sampler/dry-penalty-last-n/), [DrySequenceBreakers](/net/developer-reference/parameters/sampler/dry-sequence-breakers/).

**Mirostat**: [Mirostat](/net/developer-reference/parameters/sampler/mirostat/), [MirostatTau](/net/developer-reference/parameters/sampler/mirostat-tau/), [MirostatEta](/net/developer-reference/parameters/sampler/mirostat-eta/).

**Fine-grained controls**: [LogitBias](/net/developer-reference/parameters/sampler/logit-bias/), [EnableInfill](/net/developer-reference/parameters/sampler/enable-infill/).

## Core sampling

These four fields govern the baseline sampling pipeline. Start with these; rarely do you need the advanced filters.

### `Temperature`

Default `0.7`. Scales the logits before sampling. Higher values flatten the probability distribution (more random output); lower values sharpen it (more deterministic).

| Value | Effect |
|---|---|
| `0.0` | Greedy sampling — always pick the most likely token. Fully deterministic. |
| `0.1 - 0.3` | Precise, low-creativity output. Good for code, structured data, classification. |
| `0.7` (default) | General-purpose balance of accuracy and variety. |
| `0.8 - 1.0` | More creative, more varied output. |
| `> 1.0` | High randomness. Useful for brainstorming; risks incoherent output. |

### `TopP`

Default `0.9`. Nucleus sampling threshold. Only the smallest set of tokens whose cumulative probability exceeds `TopP` is considered for sampling. Lower `TopP` is more conservative.

- `0.9` (default) — balanced; a small tail of unlikely tokens is kept.
- `0.7 - 0.8` — more conservative; drops more of the tail.
- `1.0` — disabled; all tokens are candidates.

### `TopK`

Default `40`. Consider only the top `K` most likely tokens per step. Works alongside `TopP`.

- `40` (default) — reasonable upper bound for most models.
- `20 - 30` — more conservative.
- `0` or a very large number — disabled; `TopP` alone filters.

### `MinP`

Default `0.05`. Minimum probability relative to the top token. A token is kept only if its probability is at least `MinP × p(top)`. Useful when the tail distribution has very low-probability tokens you never want to sample.

- `0.05` (default) — reasonable.
- `0.1` — stricter; drops more tail.
- `0.0` — disabled.

## Reproducibility

### `Seed`

Default `0xFFFFFFFF` — a sentinel that `llama.cpp` maps to a time-based (non-deterministic) seed. For reproducible output, set a specific integer.

```csharp
preset.SamplerParameters.Seed = 42;
```

Two runs with the same seed, the same prompt, and identical parameters produce the same output.

### `MinKeep`

Default `1`. Minimum number of candidate tokens kept after the filters (`TopK`, `TopP`, `MinP`, `TypicalP`) are applied. Guarantees the sampler always has at least `MinKeep` tokens to choose from, preventing empty-candidate edge cases.

Leave at `1` unless you have a specific reason.

## Repetition controls

The engine penalizes tokens that appeared recently to avoid loops and verbatim repetition.

### `PenaltyContextSize`

Default `-1` (= full context). Number of recent tokens considered for repetition penalties.

- `-1` — use the model's full context size.
- A positive integer — only the last N tokens contribute.

Smaller windows make penalties more local; larger windows spread them across the whole conversation.

### `RepetitionPenalty`

Default `1.1`. Multiplicative penalty applied to recently-seen tokens. Values `> 1` make repeats less likely; `1.0` disables repetition penalty.

- `1.0` — no penalty.
- `1.05 - 1.15` (default range) — gentle anti-repetition.
- `1.2 - 1.3` — aggressive; risks under-generating common words like "the" or "and".

### `PresencePenalty`

Default `0` (disabled). Additive penalty for tokens that have appeared at least once in the penalty window. Biases the model toward new tokens not yet used.

```csharp
preset.SamplerParameters.PresencePenalty = 0.6f;
```

### `FrequencyPenalty`

Default `0` (disabled). Additive penalty proportional to how many times a token has appeared. Stronger version of presence penalty.

```csharp
preset.SamplerParameters.FrequencyPenalty = 0.3f;
```

Typical ranges for both penalties: `0.0 - 1.0`. Combine with `RepetitionPenalty` to shape repetition behavior; tune one at a time.

## Advanced filters

### `TypicalP`

Default `-1` (disabled). Locally-typical sampling — keeps tokens whose log-probability is close to the expected entropy. Alternative to nucleus sampling; rarely needed when `TopP` is set.

Enable with a value in `(0, 1]`, e.g. `0.95`.

### `TopNSigma`

Default `-1` (disabled). Keeps tokens within `N` standard deviations of the top logit. Experimental filter from recent `llama.cpp` versions.

### Dynamic temperature (`DynatempRange`, `DynatempExponent`)

Dynamically adjusts temperature per step based on token entropy. When entropy is low (the model is confident), temperature drops; when entropy is high (the model is uncertain), temperature rises.

- `DynatempRange` default `0` — dynatemp disabled.
- Set `DynatempRange > 0` to enable. Typical values `0.2 - 0.5`.
- `DynatempExponent` default `1.0` controls the shape of the entropy-to-temperature curve.

```csharp
preset.SamplerParameters.Temperature = 0.8f;
preset.SamplerParameters.DynatempRange = 0.3f;
// Effective temperature varies in [0.5, 1.1] based on per-step entropy.
```

### XTC — Exclude Top Choices

XTC randomly excludes the top tokens during sampling at a configurable probability. Useful for injecting diversity without raising overall temperature.

- `XtcProbability` default `-1` (disabled). Probability of applying XTC at each step.
- `XtcThreshold` — minimum probability below which tokens are not excluded.

### DRY — Don't Repeat Yourself

DRY detects and penalizes verbatim string repetition (word-for-word copies from earlier in the context). Useful for creative writing; often too aggressive for code.

| Field | Default | Purpose |
|---|---|---|
| `DryMultiplier` | `-1` (disabled) | Strength of the penalty. Enable with a value like `0.8`. |
| `DryBase` | `1.75` | Base of the exponential penalty for consecutive repeated tokens. |
| `DryAllowedLength` | `2` | Minimum repeat length before DRY engages. |
| `DryPenaltyLastN` | `0` | Number of trailing tokens checked (0 = all). |
| `DrySequenceBreakers` | `["\n", ":", "\"", "*"]` | Tokens that reset the repeat detector. |

```csharp
preset.SamplerParameters.DryMultiplier = 0.8f;
preset.SamplerParameters.DryBase = 1.75f;
preset.SamplerParameters.DryAllowedLength = 3;
```

### Mirostat

Adaptive sampler that targets a specific output entropy (perplexity). Alternative to temperature + nucleus sampling.

- `Mirostat = 0` (default) — disabled.
- `Mirostat = 1` — Mirostat 1.0.
- `Mirostat = 2` — Mirostat 2.0 (usually preferred).
- `MirostatTau = 5.0` — target entropy. Lower = more deterministic.
- `MirostatEta = 0.1` — learning rate.

When Mirostat is on, `TopP`, `TopK`, and `MinP` are effectively bypassed — Mirostat manages the full distribution itself.

```csharp
preset.SamplerParameters.Mirostat = 2;
preset.SamplerParameters.MirostatTau = 4.5f;
```

## Fine-grained controls

### `LogitBias`

Per-token logit adjustment. Keys are token IDs, values are additive biases applied before sampling.

```csharp
// Strongly bias against token 1234:
preset.SamplerParameters.LogitBias[1234] = -100f;
// Slightly favor token 5678:
preset.SamplerParameters.LogitBias[5678] = +2f;
```

A bias of `-100` (or lower) effectively bans a token. A bias of `+2` to `+5` noticeably favors it. Obtaining token IDs requires the model's tokenizer — not exposed in this parameter bag but available via the API reference.

### `EnableInfill`

Default `false`. Enables the INFILL sampler variant used for fill-in-the-middle code completion (models that support it). Leave off for normal chat.

## Typical recipes

### Deterministic output (for tests and reproducible demos)

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.Temperature = 0.0f; // greedy
preset.SamplerParameters.Seed = 42;
```

### Balanced chat assistant

```csharp
var preset = new Qwen25Preset();
// Defaults are already tuned:
// Temperature 0.7, TopP 0.9, TopK 40, MinP 0.05, RepetitionPenalty 1.1
```

### Creative writing

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.Temperature = 0.9f;
preset.SamplerParameters.TopP = 0.95f;
preset.SamplerParameters.MinP = 0.02f;
preset.SamplerParameters.DryMultiplier = 0.8f; // discourage verbatim repetition
```

### Tight technical output (code, structured data)

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.Temperature = 0.2f;
preset.SamplerParameters.TopP = 0.95f;
preset.SamplerParameters.RepetitionPenalty = 1.05f; // gentler for code patterns
```

### Precision via Mirostat

```csharp
var preset = new Qwen25Preset();
preset.SamplerParameters.Mirostat = 2;
preset.SamplerParameters.MirostatTau = 5.0f;
preset.SamplerParameters.MirostatEta = 0.1f;
// Temperature, TopP, TopK are now ignored.
```

## What's next

- [Chat parameters](/net/developer-reference/parameters/chat/) — max tokens and system prompt shape what the sampler generates into.
- [Context parameters](/net/developer-reference/parameters/context/) — the context window the sampler reads penalties from.
- [Custom preset](/net/use-cases/custom-preset/) — full customization patterns.

---
family: gpt-oss
scope: api
versions:
  - openai/gpt-oss-20b
  - openai/gpt-oss-120b
  - openai/gpt-oss-safeguard-20b
  - openai/gpt-oss-safeguard-120b
retrieved: 2026-07-19
primary_sources:
  - https://developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide
  - https://developers.openai.com/cookbook/articles/openai-harmony
  - https://github.com/openai/harmony
  - https://github.com/openai/gpt-oss
  - https://huggingface.co/openai/gpt-oss-20b
  - https://huggingface.co/openai/gpt-oss-20b/raw/main/config.json
  - https://huggingface.co/openai/gpt-oss-safeguard-20b
  - https://huggingface.co/openai/gpt-oss-safeguard-120b
  - https://console.groq.com/docs/model/openai/gpt-oss-safeguard-20b
  - https://console.groq.com/docs/rate-limits
maturity_note: |
  Open-weight family. "API" here means the Harmony chat-template contract plus
  the self-host / host-served inference surface, not a single OpenAI-hosted
  HTTP endpoint. Covers base gpt-oss (20b / 120b) and gpt-oss-safeguard
  (20b / 120b). The dominant contrast with the closed GPT-5.x API: reasoning
  effort is set INSIDE the system message (Harmony directive), not as a
  top-level `reasoning.effort` request parameter. Base gpt-oss + Harmony
  launched 2025-08-05; safeguard is dated 2025-10-29. Sources re-verified
  2026-07-19. Groq rows are Tier-2 host facts (host = Groq), verified against
  console.groq.com on 2026-07-19, not OpenAI primaries.
---

# gpt-oss (open weight) — API-Layer Reference

Inference-layer detail for OpenAI's open-weight line: base **gpt-oss** and
**gpt-oss-safeguard**. Portable prompt-layer content (model selection, policy
structure, output-instruction reinforcement, token budgeting) lives in
`gpt-oss-prompt.md`.

**Not this file:** the closed GPT-5.x Responses API. Its `reasoning.effort`
top-level parameter, `text.format` structured outputs, and prompt-caching
surface are unrelated to this open-weight, Harmony-format family — route to
`openai-prompt-api.md`.

## 1. API Surface

No single OpenAI-hosted endpoint. The family is consumed as open weights through
a self-host stack or a third-party host. The **verified** self-host / local
targets are:

- **vLLM** — dedicated GPUs.
- **HuggingFace Transformers.**
- **Google Colab.**
- **Ollama** — `gpt-oss-safeguard:20b` and `:120b`.
- **LM Studio** — `openai/gpt-oss-safeguard-20b` / `-120b`.

Both safeguard variants are documented for self-host. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

vLLM, Ollama, and LM Studio all expose an OpenAI-shaped Chat-Completions
endpoint over these weights, so an OpenAI-compatible client body works at the
wire level; the reasoning-effort mechanism (§4) is still a Harmony system-message
directive, not a top-level request field.

### Host-served (Groq) — Tier-2 host facts

Groq serves `openai/gpt-oss-safeguard-20b` (model page status: Preview). These
are host facts (host = Groq), verified against console.groq.com on 2026-07-19,
not OpenAI primaries.

| Item | Value |
|------|-------|
| Context window / max output | 131,072 / 65,536 tokens |
| Pricing (per 1M) | input $0.075 / cached input $0.037 / output $0.30 |
| Free-Plan rate limits | 30 RPM / 1,000 RPD / 8,000 TPM / 200,000 TPD (over-limit → HTTP 429) |
| Host quantization | "Groq's TruePoint Numerics, which reduces precision only in areas that don't affect accuracy" (verbatim) |

[source: console.groq.com/docs/model/openai/gpt-oss-safeguard-20b, retrieved 2026-07-19]
[source: console.groq.com/docs/rate-limits, retrieved 2026-07-19]

**Caution — safeguard-120b is NOT served on Groq.** There is no
`gpt-oss-safeguard-120b` on Groq: it is absent from the rate-limits model table
and its console model page returns 404. This is independently corroborated by the
HF card, which shows "This model isn't deployed by any Inference Provider" for
safeguard-120b as of 2026-07-19. **Do not conflate** the absent safeguard-120b
with the general (non-safeguard) `openai/gpt-oss-120b`, which **is** served on
Groq (30 RPM / 1K RPD / 8K TPM / 200K TPD). [source: console.groq.com/docs/model/openai/gpt-oss-safeguard-120b (404), retrieved 2026-07-19]
[source: console.groq.com/docs/rate-limits, retrieved 2026-07-19]
[source: huggingface.co/openai/gpt-oss-safeguard-120b, retrieved 2026-07-19]

## 2. Chat Template / Message Structure

Both lines require the **harmony response format**; a non-Harmony prompt "will
not work correctly." If you drive the model through vLLM / Ollama / HF /
LM Studio, the server renders Harmony for you and you do not hand-build tokens.
Hand-assemble only if you are building your own inference loop. [source: github.com/openai/harmony, retrieved 2026-07-19]

### Special tokens (base-gpt-oss / Harmony-general)

Special tokens follow the format `<|type|>`; under tiktoken they are the
`o200k_harmony` encoding. [source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]

| Token | Purpose (verbatim) | Token ID |
|-------|--------------------|----------|
| `<\|start\|>` | "Indicates the beginning of a message. Followed by the 'header' ... starting with the role" | 200006 |
| `<\|end\|>` | "Indicates the end of a message" | 200007 |
| `<\|message\|>` | "Transition from the message 'header' to the actual content" | 200008 |
| `<\|channel\|>` | "Transition to the channel information of the header" | 200005 |
| `<\|constrain\|>` | "Transition to the data type definition in a tool call" | 200003 |
| `<\|return\|>` | "Model is done sampling. A valid stop token" | 200002 |
| `<\|call\|>` | "Model wants to call a tool. A valid stop token" | 200012 |

Message form: `<|start|>{header}<|message|>{content}<|end|>`, where `{header}`
carries the role. A completed message ends with `<|end|>`; the model may instead
stop on `<|call|>` (tool call) or `<|return|>` (done). [source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]

### Roles and precedence (base-gpt-oss / Harmony-general)

Five roles, with a fixed conflict-resolution hierarchy:

`system` > `developer` > `user` > `assistant` > `tool`

- `system` — reasoning effort, meta info (knowledge cutoff, built-in tools).
- `developer` — the instructions normally called the "system prompt," plus
  function-tool declarations.
- `user` — input to the model.
- `assistant` — model output; may carry a channel tag.
- `tool` — output of a tool call (the tool name is used as the role).

[source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]

**Safeguard scope note.** The safeguard classifier flow uses only `system`
(policy + reasoning effort) + `user` (content) and never invokes `developer`.
The five-role hierarchy above is base-gpt-oss / Harmony-general; both are correct
within their own scope — do not merge them. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

### System-message contract (base line)

The base-line system message defines: model identity ("You are ChatGPT, a large
language model trained by OpenAI" — change identity via the `developer` message,
not here), meta dates (`Knowledge cutoff:`, `Current date:`), reasoning effort,
available channels (`analysis`, `commentary`, `final`), and built-in tools
(`python`, `browser`). [source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]

### Worked example (basic chat turn, base line, verbatim)

Input:

```
<|start|>user<|message|>What is 2 + 2?<|end|>
<|start|>assistant
```

Output:

```
<|channel|>analysis<|message|>User asks: "What is 2 + 2?" Simple arithmetic. Provide answer.<|end|>
<|start|>assistant<|channel|>final<|message|>2 + 2 = 4.<|return|>
```

[source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]

### Multi-turn history normalization (base line)

Two load-bearing rules when you hand-roll Harmony history rather than using the
`openai-harmony` renderer:

- **`<|return|>` is a decode-time stop token only.** When appending the
  assistant reply to history for the next turn, replace the trailing
  `<|return|>` with `<|end|>` so stored messages are fully formed. Prior messages
  in prompts end with `<|end|>`. [source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]
- **Drop prior CoT on subsequent turns** when the previous assistant turn ended
  in a `final`-channel message. The exception is tool/function calling: the model
  calls tools as part of its chain-of-thought, so pass the prior CoT back in for
  subsequent sampling within a tool loop. [source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]

## 3. Sampling Parameters

Base-line recommendation: **`temperature=1.0` and `top_p=1.0`.** This appears on
the reference-implementation repo (github.com/openai/gpt-oss), not on the HF
model card, and the repo's vLLM code sample sets `temperature=1` explicitly. It
is stated for base gpt-oss (both 20b and 120b, undifferentiated). [source: github.com/openai/gpt-oss, README, retrieved 2026-07-19]

**No safeguard-specific override exists.** Neither safeguard card nor the
safeguard guide restates temperature / top_p for classification. Absent an
override, treat T=1.0 / top_p=1.0 as the standing family default; whether
classification wants a lower, more deterministic setting is undocumented (see
Gaps). [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

## 4. Reasoning / Thinking Control

### `reasoning_effort` is set in the system message

Accepted levels: **`low` / `medium` / `high`**, **default `medium`** if unset.
It is set **inside the `system` message**, not as a top-level API parameter. In
raw Harmony this is the directive `Reasoning: high` in the system-message body;
through a renderer/host it is passed as `reasoning_effort` on the system turn. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]
[source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]

**Contrast with GPT-5.x.** The closed GPT-5.x API takes a top-level
`reasoning.effort` request field; gpt-oss / safeguard instead carries the
directive in the system message. The name is similar; the placement is not
portable. Base line and safeguard fine-tune share the identical mechanism
(default medium), confirming it is Harmony-general, not safeguard-specific. [source: huggingface.co/openai/gpt-oss-20b, retrieved 2026-07-19]

### Channels

- **Safeguard flow: two channels** — a reasoning channel and an output channel.
  `commentary` sits dormant in a pure text-classification flow (no tool calls),
  so the safeguard guide describes it as effectively two-channel. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]
- **Base gpt-oss / Harmony-general: three channels** — `analysis` (chain of
  thought; does **not** meet final-channel safety standards — avoid showing to
  end users), `commentary` (typically function-tool calls, occasionally a
  multi-tool preamble), and `final` (the end-user-facing response). [source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]

### Bounding reasoning time

To bound reasoning, **lower `reasoning_effort`** — do not cap max output tokens.
OpenAI's guidance is to "ideally not cap the maximum output tokens to give the
model enough room to reason." Capping tokens starves the reasoning trace; lower
the effort level instead. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

## 5. Tool Use / Function Calling

**Safeguard:** no first-party tool-use contract. Safeguard is a text-only safety
classifier; for tool use / agentic work, use base gpt-oss. Aggregator claims
that safeguard supports web search or function calling conflate it with the base
line. [source: huggingface.co/openai/gpt-oss-safeguard-20b, retrieved 2026-07-19]

**Base gpt-oss:** function calls route to the `commentary` channel; built-in
tools (`python`, `browser`) normally route to `analysis`. Function-tool
declarations live in the `developer` message. The Harmony worked example
declares tools as a TypeScript-style `namespace functions { ... }` block inside
the `developer` message and stops generation on `<|call|>`. [source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]
[source: github.com/openai/harmony, retrieved 2026-07-19]

## 6. Structured Outputs

For safeguard, the "structured output" is the classification contract itself,
enforced by the policy prompt, not a separate JSON-mode toggle. Three documented
output tiers (bare `0`/`1`; policy-referencing JSON with `violation` /
`policy_category`; rationale JSON with `rule_ids`, `confidence`, and a short
2-4-bullet rationale) are specified in the policy prompt and reinforced by
stating the output instruction twice. Full authoring guidance is in
`gpt-oss-prompt.md`. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

No dedicated grammar / JSON-Schema constraint mode is documented on the safeguard
guide or HF cards for these models; schema enforcement, if needed, is an
inference-stack feature (vLLM guided decoding, etc.), independent of the model
and not documented on the family's own primaries this pass.

## 7. Caching, Batch, Streaming

Open-weight models — caching, batching, and streaming are properties of the
serving stack (vLLM prefix caching, continuous batching, SSE streaming over an
OpenAI-compatible wrapper), not of the model. The one host data point verified
this pass is Groq's cached-input price ($0.037 per 1M vs $0.075 uncached input)
for safeguard-20b — a host billing fact, not a model capability. [source: console.groq.com/docs/model/openai/gpt-oss-safeguard-20b, retrieved 2026-07-19]

## 8. Deployment Flags

- **Memory footprints hold because of native MXFP4.** Both lines are
  post-trained with MXFP4 quantization of the MoE weights; "all evals were
  performed with the same MXFP4 quantization." This lets 120b run on a single
  80GB GPU and 20b within 16GB. MXFP4 is the eval-time precision, not a host
  add-on — word any host-quantization caveat accordingly. [source: huggingface.co/openai/gpt-oss-20b, retrieved 2026-07-19]
- **Context length 131,072** via YaRN RoPE scaling from a 4,096-token trained
  base (`config.json`: `max_position_embeddings: 131072`, `rope_type: yarn`,
  `factor: 32.0`). Not stated in card prose — from the raw repo config. [source: huggingface.co/openai/gpt-oss-20b/raw/main/config.json, retrieved 2026-07-19]
- **Ollama IDs:** `gpt-oss-safeguard:20b`, `gpt-oss-safeguard:120b`. **LM Studio
  IDs:** `openai/gpt-oss-safeguard-20b`, `-120b`. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

## 9. Deprecations and Breaking Changes

No deprecations apply to this open-weight family as of 2026-07-19; the weights
are Apache-2.0 and self-hostable, so there is no provider-driven sunset path of
the kind the closed GPT-5.x line carries. The only availability delta worth
tracking is host-side: `gpt-oss-safeguard-120b` is not served by any HF Inference
Provider (Groq included) as of 2026-07-19 (§1). [source: huggingface.co/openai/gpt-oss-safeguard-120b, retrieved 2026-07-19]

## 10. Gaps

- **Harmony token-level template — captured; renderer preferred.** The special
  tokens, role hierarchy, channel model, and worked examples above are the
  canonical spec (developers.openai.com/cookbook/articles/openai-harmony). The
  `openai-harmony` package (`docs/python.md`, `docs/rust.md` in
  github.com/openai/harmony) is the intended way to render these — hand-building
  is error-prone. No token-level facts differ across the four variants; the
  encoding (`o200k_harmony`) is architecture-shared.
- **No safeguard-specific sampling guidance.** T=1.0 / top_p=1.0 is the base
  default (§3); no classification-specific override is published, checked
  2026-07-19.
- **No numeric multi-policy cap.** Multi-policy degrades accuracy with no stated
  maximum. Not documented at the safeguard guide, checked 2026-07-19.
- **No vendor multilingual classification metrics.** Not documented at the
  safeguard guide, checked 2026-07-19.
- **No primary quantization-vs-threshold-stability data.** MXFP4 is confirmed
  native/eval-time, but no data quantifies how further host-side requantization
  shifts classification boundaries. Not documented, checked 2026-07-19.
- **safeguard-120b license not re-confirmed from its own front matter.**
  Apache-2.0 is inherited-by-statement from the base card / collection, not
  independently re-scraped from the 120b card's raw README this pass.

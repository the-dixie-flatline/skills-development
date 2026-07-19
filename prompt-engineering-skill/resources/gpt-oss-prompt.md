---
family: gpt-oss
scope: prompt
versions:
  - openai/gpt-oss-20b
  - openai/gpt-oss-120b
  - openai/gpt-oss-safeguard-20b
  - openai/gpt-oss-safeguard-120b
retrieved: 2026-07-19
primary_sources:
  - https://developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide
  - https://developers.openai.com/cookbook/articles/openai-harmony
  - https://huggingface.co/openai/gpt-oss-20b
  - https://huggingface.co/openai/gpt-oss-safeguard-20b
  - https://huggingface.co/openai/gpt-oss-safeguard-120b
  - https://github.com/openai/gpt-oss
  - https://github.com/openai/harmony
maturity_note: |
  This file covers OpenAI's open-weight line: base gpt-oss (20b / 120b) and the
  gpt-oss-safeguard classifier fine-tunes (20b / 120b). All four are
  Harmony-format, self-hosted or host-served, Apache-2.0 open weights — a
  distinct surface from the closed GPT-5.x Responses API (covered in
  `openai-prompt.md` / `openai-prompt-api.md`). Base gpt-oss and the Harmony
  format launched 2025-08-05; the safeguard line is dated 2025-10-29 across the
  HuggingFace cards and the safeguard cookbook guide. Sources re-verified
  2026-07-19. The safeguard-specific policy-as-prompt discipline (four-section
  policy, token budgeting, output-instruction reinforcement) is documented by
  OpenAI for classification tasks; the general policy-authoring method
  generalizes and is a `prompt-engineering-architect` concern, not a
  model-specific one.
---

# gpt-oss (open weight) — Prompt-Layer Reference

Portable prompting guidance for OpenAI's open-weight line: base **gpt-oss**
(general instruction model) and **gpt-oss-safeguard** (a fine-tune specialized
for policy-based safety classification). API-layer detail (Harmony token
template, `reasoning_effort` mechanics, self-host stack, Groq host facts) lives
in `gpt-oss-prompt-api.md`.

**Scope / not this file.** This is the open-weight, Harmony-format, self-hosted
family. It is not the closed GPT-5.x API. If the task is the GPT-5.x Responses
API (`instructions` field, top-level `reasoning.effort` {none..xhigh},
`text.format` structured outputs, prompt caching, hosted deep-research), route
to `openai-prompt.md`. Nothing here transfers to that surface and vice versa.

## 1. Model Selection

Two model lines, two sizes each. Base gpt-oss is the general-purpose instruction
model; gpt-oss-safeguard is a fine-tune of it that classifies content against a
policy supplied at inference time.

| Model | Total / Active params | Memory footprint | License |
|-------|----------------------|------------------|---------|
| `openai/gpt-oss-20b` | 21B / 3.6B | runs within 16GB | Apache 2.0 |
| `openai/gpt-oss-120b` | 117B / 5.1B | single 80GB GPU (H100 / MI300X) | Apache 2.0 |
| `openai/gpt-oss-safeguard-20b` | 21B / 3.6B | fits a 16GB-VRAM GPU | Apache 2.0 |
| `openai/gpt-oss-safeguard-120b` | 117B / 5.1B | single H100 (80GB) | Apache 2.0 (see note) |

[source: huggingface.co/openai/gpt-oss-safeguard-20b, retrieved 2026-07-19]
[source: huggingface.co/openai/gpt-oss-safeguard-120b, retrieved 2026-07-19]
[source: huggingface.co/openai/gpt-oss-20b, retrieved 2026-07-19]

Notes:

- The 16GB / 80GB footprints hold because both lines are post-trained with
  **MXFP4 quantization of the MoE weights** — this is the native, eval-time
  precision, not a host-side lossy re-quantization. [source: huggingface.co/openai/gpt-oss-20b, retrieved 2026-07-19]
- Both safeguard variants are fine-tunes of the correspondingly-sized base
  gpt-oss. The HF "Model tree" lists `openai/gpt-oss-120b` as the base of
  gpt-oss-safeguard-120b. [source: huggingface.co/openai/gpt-oss-safeguard-120b, retrieved 2026-07-19]
- **Apache-2.0 on safeguard-120b is inherited-by-statement**, not independently
  re-confirmed against that card's own front matter this pass. The base 20b
  card states Apache 2.0 explicitly ("Build freely without copyleft
  restrictions or patent risk"). [source: huggingface.co/openai/gpt-oss-20b, README raw, retrieved 2026-07-19]

**Selection rule (safeguard vs base).** Use gpt-oss-safeguard **only** for
safety-classification use cases — labeling content against a written policy. For
anything else (agentic work, tool use, general generation, coding), OpenAI
directs users to base gpt-oss. Do not reach for safeguard as a general chat or
tool-calling model. [source: huggingface.co/openai/gpt-oss-safeguard-20b, retrieved 2026-07-19]
[source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

## 2. Prompt Structure Conventions

### Harmony is mandatory

Both lines were trained on the **harmony response format** and "should only be
used with the harmony format as it will not work correctly otherwise." This is
not a soft preference: a raw prompt that is not Harmony-structured produces
malformed behavior. If you drive the model through an inference provider or a
server (vLLM, Ollama, HuggingFace, LM Studio), the server renders Harmony for
you; if you hand-assemble prompts, you must emit the format yourself (token
template in `gpt-oss-prompt-api.md`). [source: huggingface.co/openai/gpt-oss-20b, retrieved 2026-07-19]
[source: github.com/openai/harmony, retrieved 2026-07-19]

### Safeguard message placement (policy vs content)

For the safeguard classification flow the placement is decisive and simple:

- **Policy prompt → `system` message.**
- **Content to classify → `user` message.**
- `reasoning_effort` is **also** set in the `system` message (see
  `gpt-oss-prompt-api.md`).

The worked curl example in the safeguard guide is exactly `role: system` =
policy, `role: user` = content. The safeguard flow uses only `system` + `user`
and never invokes a `developer` role. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

The `developer` role and the full five-role Harmony hierarchy
(`system` > `developer` > `user` > `assistant` > `tool`) are **base-gpt-oss /
Harmony-general** semantics, where the `developer` message carries what is
normally called the "system prompt." They are not part of the safeguard
classifier contract — do not port the base-line role hierarchy into a safeguard
policy prompt. [source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]

### Safeguard policy structure (four sections)

Structure the policy in the `system` message as four labeled sections, using
headers rather than prose:

1. **Instruction** — the task and the output instruction.
2. **Definitions** — key terms.
3. **Criteria** — what VIOLATES (label `1`) vs what is SAFE (label `0`).
4. **Examples** — 4-6 short boundary examples, each labeled `0` or `1`.

[source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

## 3. Instruction Patterns

### Reinforce the output instruction (safeguard)

State the output instruction at least twice: once near the top (in the
INSTRUCTION section) and again near the bottom, before the EXAMPLES. The single
most common cause of malformed classifier output is stating the output contract
only once. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

### Policy length heuristic (safeguard)

The model tolerates policies up to roughly 10,000 tokens, but early testing
suggests the **optimal range is 400-600 tokens** per policy. This is an explicit
soft heuristic — OpenAI states there is "no one-size-fits-all." Longer is not
better; a tight 400-600-token policy outperforms a sprawling one. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

### Multi-policy: pre-compress, expect degradation (safeguard)

Multiple policies can be evaluated in a single call if all of them are in the
prompt. Two rules:

- **Pre-compress** each sub-policy to **300-600 tokens** (definitions +
  categories + 1-2 examples each) before combining.
- **Expect accuracy loss.** "Additional policies lead to small but meaningful
  degradations in accuracy." There is **no documented numeric cap** on how many
  policies you may combine — you trade breadth for per-policy accuracy with no
  vendor-stated ceiling (see Gaps).

[source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

### Output contracts (safeguard)

Three documented output tiers, in increasing verbosity:

1. **Bare binary** — `0` or `1`.
2. **Policy-referencing JSON** — `{"violation": 0|1, "policy_category": ...}`.
3. **Rationale JSON** — adds `rule_ids`, `confidence`, and a **short**
   non-step-by-step rationale (2-4 bullets or 1-2 sentences).

The rationale tier is deliberately short and is **not** the model's full
chain-of-thought — it is a compact justification, distinct from the raw
reasoning trace (see §6). [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

### Ambiguity and precedence rules (safeguard)

- **Avoid vague qualifiers** like "generally" or "usually" in the policy — they
  produce inconsistent boundaries.
- **Add an explicit escalation path** for ambiguous, regional, or
  cross-language cases (e.g., "if unclear, label `1` and set low confidence").
- **State which policy wins on conflict.** When combining policies, name the
  dominant-policy rule so the model has a deterministic tiebreak.

[source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

### Off-the-shelf policy packs

OpenAI ships `openai/teen-safety-policy-pack` (6 policy domains) as ready-to-use
safeguard prompts. Start from a pack rather than authoring from scratch when the
domain overlaps. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]

## 4. Context Window Practical Guidance

Native context is **131,072 tokens**, reached via YaRN RoPE scaling from a
4,096-token trained base. This figure comes from the base gpt-oss-20b repo
`config.json` (`max_position_embeddings: 131072`); it is not stated in the HF
card prose. [source: huggingface.co/openai/gpt-oss-20b/raw/main/config.json, retrieved 2026-07-19]

For safeguard specifically, the practical budget is dominated by policy length,
not raw window: keep each policy near the 400-600-token heuristic and reserve
room for the model to reason (see the max-output guidance in
`gpt-oss-prompt-api.md`). The 131K window is ample for single-policy
classification; multi-policy prompts stress accuracy (§3) long before they
stress the context window.

## 5. Multimodal Conventions

Not applicable. Base gpt-oss and gpt-oss-safeguard are text models; safeguard is
a text-only safety classifier. No image, audio, or video input path is
documented for either line. [source: huggingface.co/openai/gpt-oss-safeguard-20b, retrieved 2026-07-19]

## 6. Behavioral Quirks

- **Raw chain-of-thought is not end-user-safe.** The reasoning trace is intended
  for developers and safety practitioners only, "not intended for exposure to
  general users." At the Harmony level this is formalized: analysis-channel
  (CoT) messages "do not adhere to the same safety standards as final messages."
  Surface the final output (or the short rationale tier), never the raw trace,
  to end users. [source: huggingface.co/openai/gpt-oss-safeguard-20b, retrieved 2026-07-19]
  [source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]
- **Multi-policy accuracy degrades gracefully but really.** Combining policies
  works, but each added policy costs accuracy (§3). Treat single-policy calls as
  the accuracy baseline. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]
- **Safeguard is a classifier, not an agent.** It has no first-party tool-use
  contract in the safeguard flow. Aggregator claims that safeguard supports web
  search / function calling conflate it with base gpt-oss — for tool use, use
  base gpt-oss. [source: huggingface.co/openai/gpt-oss-safeguard-20b, retrieved 2026-07-19]

## 7. Anti-Patterns

- **Do not use safeguard for anything but safety classification.** Agentic work,
  tool use, coding, or general generation route to base gpt-oss. [source: huggingface.co/openai/gpt-oss-safeguard-20b, retrieved 2026-07-19]
- **Do not put the policy in the `user` message.** Policy goes in `system`,
  content-to-classify goes in `user`. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]
- **Do not use a `developer` role in the safeguard flow.** That role and the
  five-role hierarchy are base-gpt-oss / Harmony-general, not the safeguard
  contract. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]
  [source: developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]
- **Do not run either line without Harmony.** Non-Harmony prompts "will not work
  correctly." [source: huggingface.co/openai/gpt-oss-20b, retrieved 2026-07-19]
- **Do not use vague qualifiers in a policy.** "Generally" / "usually" blur the
  0/1 boundary. State criteria and an explicit escalation path. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]
- **Do not over-length the policy.** Past the 400-600-token sweet spot, extra
  policy text tends to hurt, not help. [source: developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]
- **Do not expose the raw reasoning trace to end users.** [source: huggingface.co/openai/gpt-oss-safeguard-20b, retrieved 2026-07-19]

## 8. Gaps

- **No numeric cap on concurrent policies.** Multi-policy is supported and warned
  to degrade accuracy, but OpenAI publishes no maximum policy count. Not
  documented at developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide,
  checked 2026-07-19.
- **No vendor multilingual classification metrics.** The safeguard guide gives no
  multilingual F1 or per-language classification accuracy. A Tier-2 multilingual
  F1 figure (aclanthology) exists for cross-lingual behavior but is not
  vendor-grade and is not relayed here as fact. Not documented at the safeguard
  guide, checked 2026-07-19. [community-reported]
- **No primary quantization-vs-threshold-stability data.** MXFP4 is confirmed as
  the native eval-time precision, but neither the guide nor the cards quantify
  how further (host-side) requantization shifts classification boundaries. Not
  documented, checked 2026-07-19.
- **No safeguard-specific sampling recommendation.** The T=1.0 / top_p=1.0 base
  default (see `gpt-oss-prompt-api.md`) is not restated or overridden for the
  classification task on any safeguard page, checked 2026-07-19. Whether
  classification wants a lower, more deterministic temperature is undocumented.

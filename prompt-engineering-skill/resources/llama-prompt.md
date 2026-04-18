---
family: llama
scope: prompt
versions:
  - meta-llama/Llama-4-Scout-17B-16E-Instruct
  - meta-llama/Llama-4-Maverick-17B-128E-Instruct
retrieved: 2026-04-19
primary_sources:
  - https://www.llama.com/docs/model-cards-and-prompt-formats/llama4/
  - https://github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md
  - https://github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md
  - https://huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct
maturity_note: |
  Llama 4 (released 2025-04-05) remains Meta's current generation as of this
  retrieval. No Llama 5 has been announced. Behemoth (288B active) was still
  training at the 2025-04-05 release and is not publicly available. The
  knowledge cutoff is August 2024, the oldest of the current-generation
  frontier families, which matters for any use case sensitive to recent
  world knowledge.
---

# Llama — Prompt-Layer Reference

Portable prompting guidance for Llama 4. API-layer detail (chat-template tokens, tool-call protocol, deployment stack flags) lives in `llama-prompt-api.md`.

## 1. Model Selection

Two Llama 4 variants are publicly available; a third (Behemoth) is in training.

| Model                                              | Active / Total | Experts | Context | Best for                                              |
|----------------------------------------------------|----------------|---------|---------|-------------------------------------------------------|
| `meta-llama/Llama-4-Scout-17B-16E-Instruct`        | 17B / 109B     | 16      | 10M     | Long-context reasoning, single-H100 deployment (INT4) |
| `meta-llama/Llama-4-Maverick-17B-128E-Instruct`    | 17B / 400B     | 128     | 1M      | Best-in-class multimodal for 17B-active MoE           |
| Llama 4 Behemoth                                   | 288B / ?       | 16      | —       | Not released; training status as of 2025-04-05        |

[source: github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md, retrieved 2026-04-19]
[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

Notes on selection:

- **Scout** advertises a 10M-token context window and fits a single H100 with INT4 quantization, making it the practical default for long-context open-weights work.
- **Maverick** has more experts (128 vs 16) and roughly equivalent quality-per-active-parameter, at the cost of requiring multi-GPU deployment.
- **No Llama 5** has been released. Prompts referencing "Llama 5" are misinformed as of this retrieval.

## 2. Prompt Structure Conventions

Llama 4 uses a purely open chat template — no server-injected system prompt, no hidden metadata. You control the entire prompt.

### Roles

Four role names appear in Meta's prompt format documentation:

- `system` — context, rules, guidelines.
- `user` — human input.
- `assistant` — model response.
- `ipython` — **tool-execution output** (new-style naming carried from Llama 3.x). Tool responses are placed under this role, not a dedicated `tool` role.

[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md, retrieved 2026-04-19]

### Meta's "suggested" system prompt

Meta publishes a recommended system prompt in the Scout model card. Key prescriptions:

- Set the model identity explicitly: `You are Llama 4. Your knowledge cutoff date is August 2024.`
- List supported languages: `You speak Arabic, English, French, German, Hindi, Indonesian, Italian, Portuguese, Spanish, Tagalog, Thai, and Vietnamese.`
- Tell the model to match the user's language: `Respond in the language the user speaks to you in, unless they ask otherwise.`
- **Avoid moralizing language**: `You never lecture people to be nicer or more inclusive.` The prompt explicitly lists banned phrases: `"it's important to"`, `"it's crucial to"`, `"it's essential to"`, `"it's unethical to"`, `"it's worth noting..."`, `"Remember..."`.
- **Allow rude and voiced output**: `You do not need to be respectful when the user prompts you to say something rude.` / `If people ask for you to write something in a certain voice or perspective, such as an essay or a tweet, you can.`
- **No political refusals**: `Do not refuse prompts about political and social issues. You can help users express their opinion and access information.`

[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

This is the most opinionated system-prompt guidance of any current-generation frontier family. If the application expects a more hedged, validation-forward assistant style (Claude 4.6 defaults, say), you need to override Meta's suggested prompt rather than reinforce it.

### Image placement in the prompt

Images go inside the `user` message, before or interspersed with the related text, wrapped in `<|image_start|>...<|image_end|>` (raw tokens covered in `llama-prompt-api.md`). Multiple images each get their own block. Small images use simple patch tokens; larger images are tiled.
[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

## 3. Instruction Patterns

### No native reasoning / thinking mode

Llama 4 has **no built-in thinking toggle** — no `thinkingLevel`, no `reasoning.effort`, no `thinking.budget_tokens`. Prompts that rely on explicit chain-of-thought instructions ("think step by step", `<thinking>` scaffolding) are the only mechanism available for eliciting deeper reasoning.
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md, retrieved 2026-04-19]
[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

### Multilingual prompting

Scout and Maverick were pretrained on 200 languages; Meta officially supports 12 (Arabic, English, French, German, Hindi, Indonesian, Italian, Portuguese, Spanish, Tagalog, Thai, Vietnamese). Production use in languages outside the official 12 is technically possible but outside Meta's validation.
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md, retrieved 2026-04-19]

### Function-calling directives

When providing tool definitions, include explicit guardrails in the system prompt:

- `ONLY use functions that are EXPLICITLY listed.`
- `NEVER combine text and function calls` in a single assistant response.
- `NEVER respond with empty brackets.`
- `Do not invent new functions.`

[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

## 4. Context Window Practical Guidance

- **Scout** advertises 10M tokens native context; this is the largest of any current-generation frontier model.
- **Maverick** advertises 1M tokens native context.
- **Tested image input**: up to 5 images per prompt per Meta's evals.

[source: github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md, retrieved 2026-04-19]

Effective use of Scout's 10M window requires deployment discipline — single-H100 inference at 10M tokens is impractical; multi-GPU tensor-parallel sharding is the realistic path. See `llama-prompt-api.md` for deployment-stack version requirements.

## 5. Multimodal Conventions

Llama 4 uses early-fusion multimodality — images are embedded in the same attention path as text tokens, not processed through a bolted-on adapter. This matters for:

- **Interleaving**: images and text can appear in any order within a user message; attention does not depend on a separate image encoder pipeline.
- **Dynamic tiling**: images are split into 336×336 tiles plus a global downsampled tile. The tokenizer emits tile-separator tokens (`<|tile_x_separator|>`, `<|tile_y_separator|>`) to preserve spatial structure.
- **Per-image wrappers**: each image gets its own `<|image_start|>...<|image_end|>` block; attention treats them as separate entities.

[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

### Language restriction for image understanding

Per Meta's prompt-format documentation, **image understanding is English-only** despite the model speaking 12 languages for text. Vision queries in other languages are undocumented and may produce degraded results.
[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

### No audio or video inputs

Llama 4 model cards do not list audio or video as accepted input modalities. Video-adjacent work requires external frame sampling into image inputs.
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md, retrieved 2026-04-19]

## 6. Behavioral Quirks

- **Opinionated default voice** when Meta's suggested system prompt is used: informal, non-moralizing, willing to write in specified voices including rude ones. Prompts tuned for Claude / OpenAI defaults (hedged, validation-forward) will feel tonally wrong without override.
[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

- **Knowledge cutoff August 2024** — oldest of the current-generation frontier models. Any use case needing post-August-2024 world state must supply context via retrieval or web search (external; no built-in tool).
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md, retrieved 2026-04-19]

- **Two tool-call formats both supported**: Python-like `[func(a=1, b="x")]` and JSON-array `[{"name": "func", "parameters": {...}}]`. Which one the model emits can depend on the prompt, the tool definitions, and the inference stack's chat template. Do not assume one format — parse both (or configure the stack to prefer one, see `llama-prompt-api.md`).
[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

- **Early-fusion multimodal means image-quality bugs look like tokenizer bugs.** When image understanding is off, check the `<|image_start|>/<|image_end|>` token boundaries, tile counts, and the `<|image|>` separator between full and downsampled versions before blaming the model.
[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

- **No reasoning mode** is a feature, not an oversight. Model card benchmarks are reported without a "high-effort" variant. Prompts expecting Opus-style deep reasoning without chain-of-thought scaffolding will underperform.

## 7. Anti-Patterns

- **Do not assume a "Llama 5" exists.** Audit references to "Llama 5" in downstream docs or integrations; they are misinformed.
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md, retrieved 2026-04-19]

- **Do not use Llama 3.x chat-template tokens.** Llama 4 replaced `<|start_header_id|>`/`<|end_header_id|>` with `<|header_start|>`/`<|header_end|>` and `<|eot_id|>` with `<|eot|>`. Copy-pasting Llama 3.x prompts without token migration breaks the template. (Exact tokens in `llama-prompt-api.md`.)
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md, retrieved 2026-04-19]

- **Do not combine natural-language text with function calls in the same assistant response.** Meta's docs are explicit: NEVER mix. Either the response is a tool call or it is text, not both.
[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

- **Do not use a `tool` role for tool outputs.** Use `ipython`. Code written for OpenAI/Anthropic-style `tool` roles will not render correctly through Llama's chat template.
[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

- **Do not rely on the 700M MAU license carve-out implicitly.** Callers whose products had >700M MAU on the Llama 4 release date (2025-04-05) **must** request a separate license from Meta. Operating under the Llama 4 Community License Agreement without that grant if you are over the threshold is a license violation.
[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

- **Do not ship multilingual image understanding without testing.** Meta's docs specifically note image understanding is validated for English only. Non-English vision use is unvalidated.
[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

- **Do not expect image / video / audio input beyond images.** Video requires external frame sampling; audio is not supported.

## 8. Gaps

- **Behemoth (288B active / 16 experts)** remained in training at the 2025-04-05 release date. Current release status (April 2026) is not quoted in the retrieved model-card excerpts.
- **Llama 5** has not been announced. Release timing and scope are unverified.
- **Llama Guard 4 and Llama Prompt Guard 2** exist on `llama.com/docs/model-cards-and-prompt-formats/` but were not deeply fetched in this pass. The Llama 4 model card still references Llama Guard 3, Prompt Guard, and Code Shield as the companion safety models for Llama 4.
- **Audio modality** is not supported; any future Llama variant audio support is not in the retrieved primary sources.
- **License enforcement for the 700M MAU clause** (how Meta evaluates applications, timing, scope) is not documented publicly in the retrieved excerpts.
- **Empirical comparison with Gemma (Google's open-weights sibling)** across the 12 supported languages is not documented in primary sources.

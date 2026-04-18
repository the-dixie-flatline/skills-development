---
family: llama
scope: api
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
  Llama 4 is open-weights. "API" here means the chat-template contract plus
  the inference-stack surface (transformers, vLLM, SGLang), not a
  provider-hosted HTTP endpoint. Sampling-parameter defaults are not
  explicitly published by Meta for Llama 4; this file does not fabricate them.
  vLLM/SGLang version floors beyond `transformers>=4.51.0` were not pulled
  in this retrieval pass and appear in Gaps.
---

# Llama — API-Layer Reference

Inference-layer detail for Llama 4. Prompt-layer content (selection, voice, anti-patterns) lives in `llama-prompt.md`.

## 1. API Surface

Llama 4 has no official Meta-hosted API. Deployment paths:

- **HuggingFace `transformers`** (≥ 4.51.0) with `Llama4ForConditionalGeneration`.
- **vLLM** via `LLM(model="meta-llama/Llama-4-Scout-17B-16E-Instruct", ...)`. Exact minimum version not quoted in the retrieved excerpts (see Gaps).
- **SGLang** via `sglang.launch_server --model-path ...`. Exact minimum version not quoted in the retrieved excerpts.
- **llama.cpp / GGUF** via community quantizations on HuggingFace.

[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

### Model IDs (HuggingFace)

```
meta-llama/Llama-4-Scout-17B-16E-Instruct
meta-llama/Llama-4-Maverick-17B-128E-Instruct
```

The `17B-16E` / `17B-128E` encoding is `{active params}B-{expert count}E`. The `Instruct` suffix is the chat-tuned variant.
[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

### Inference-class usage

```python
from transformers import AutoProcessor, Llama4ForConditionalGeneration
import torch

model_id = "meta-llama/Llama-4-Scout-17B-16E-Instruct"
processor = AutoProcessor.from_pretrained(model_id)
model = Llama4ForConditionalGeneration.from_pretrained(
    model_id,
    attn_implementation="flex_attention",
    device_map="auto",
    torch_dtype=torch.bfloat16,
)
```

[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

## 2. Chat Template / Message Structure

Llama 4 uses a special-token chat template. Exact tokens:

| Token                            | Purpose                                                       |
|----------------------------------|---------------------------------------------------------------|
| `<|begin_of_text|>`              | Start of prompt                                               |
| `<|end_of_text|>`                | End-of-generation (pretrained base only)                      |
| `<|header_start|>` / `<|header_end|>` | Role name delimiters                                     |
| `<|eot|>`                        | End of turn (chat-tuned models)                               |
| `<|image_start|>` / `<|image_end|>` | Image-block delimiters                                     |
| `<|image|>`                      | Separator between full-resolution and downsampled image       |
| `<|patch|>`                      | Individual image-tile patch token                             |
| `<|tile_x_separator|>`           | Horizontal tile boundary                                      |
| `<|tile_y_separator|>`           | Vertical tile boundary                                        |

[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md, retrieved 2026-04-19]

### Basic chat template

```
<|begin_of_text|><|header_start|>system<|header_end|>

{system prompt text}<|eot|><|header_start|>user<|header_end|>

{user message}<|eot|><|header_start|>assistant<|header_end|>

{assistant response}<|eot|>
```

[source: github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md, retrieved 2026-04-19]

### Roles

`system`, `user`, `assistant`, `ipython`. `ipython` is the role for **tool-execution output** — do not use a `tool` role.
[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md, retrieved 2026-04-19]

### Tool-response rendering

```
<|header_start|>ipython<|header_end|>

{tool response content}<|eot|>
```

[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]
[testable: id=llama.ipython-role-tool-response.v1, expected=tool responses rendered with the ipython role, not a tool role, when using the canonical chat template]

### Migration from Llama 3.x

[applies-to: meta-llama/Llama-4-Scout-17B-16E-Instruct, meta-llama/Llama-4-Maverick-17B-128E-Instruct]

| Llama 3.x token / role      | Llama 4 equivalent         |
|-----------------------------|----------------------------|
| `<|start_header_id|>`       | `<|header_start|>`         |
| `<|end_header_id|>`         | `<|header_end|>`           |
| `<|eot_id|>`                | `<|eot|>`                  |
| `tool` role (in some wrappers) | `ipython` role           |

Copy-paste of Llama 3.x chat templates into Llama 4 breaks the template silently — the model will treat the old tokens as ordinary text.
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md, retrieved 2026-04-19]

## 3. Sampling Parameters

Meta does not publish recommended sampling-parameter defaults or bounds for Llama 4 in the retrieved primary sources. The HuggingFace Scout model card shows only `max_new_tokens=256` in its inference example, with no temperature/top_p/top_k guidance.

[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

Treat this as a Gap. Callers should validate sampling on their own workload and not import defaults from the Llama 3.x era blindly.

## 4. Reasoning / Thinking Control

**No native reasoning or thinking mode.** Llama 4 has no equivalent of `thinking.budget_tokens`, `thinkingLevel`, or `reasoning.effort`. Deep-reasoning behavior must be elicited through prompting (explicit chain-of-thought, `<thinking>` scaffolding) and evaluated on workload metrics.
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md, retrieved 2026-04-19]
[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

## 5. Tool Use / Function Calling

### Two supported emission formats

Llama 4 can emit tool calls in either Python-like syntax or JSON-array form. Which it emits is driven by prompt shape, tool-definition shape, and inference-stack chat template.

**Python-like (native format):**

```
[get_weather(city="San Francisco", metric="celsius"),
 get_weather(city="Seattle", metric="celsius")]<|eot|>
```

**JSON-array:**

```json
[
  {
    "name": "get_weather",
    "parameters": { "city": "San Francisco", "metric": "celsius" }
  }
]
```

**Custom XML-ish alternative (documented):**

```
<function=get_weather>{"city": "San Francisco"}</function><|eot|>
```

[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md, retrieved 2026-04-19]

### Tool-definition shape

Functions are declared as JSON inside the system or user message:

```json
{
  "name": "get_weather",
  "description": "Get weather info for a city.",
  "parameters": {
    "type": "dict",
    "required": ["city"],
    "properties": {
      "city": { "type": "string", "description": "City name" },
      "metric": { "type": "string", "default": "celsius" }
    }
  }
}
```

[source: github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md, retrieved 2026-04-19]

### Constraints (per Meta's docs)

- Assistant responses that emit function calls **must not** include natural-language text in the same response.
- Only use functions explicitly listed in the declarations.
- Never respond with empty brackets `[]`.
- Never invent functions not in the list.

[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

### Inference-stack parser flags

vLLM ships a Llama 4 JSON tool-call chat template at `vllm/examples/tool_chat_template_llama4_json.jinja`. Pass this template at server startup to force JSON-format tool emissions. Exact CLI flag (`--chat-template ...`) wiring is stack-specific and not quoted in the retrieved primary excerpts.
[source: github.com/vllm-project/vllm/blob/main/examples/tool_chat_template_llama4_json.jinja, retrieved 2026-04-19]

### Parallel tool calls

Multiple tool calls in a single assistant response are supported by the native Python-like format (a list of calls):

```
[get_weather(city="San Francisco"), get_weather(city="Seattle")]<|eot|>
```

[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

## 6. Structured Outputs

**No native structured-output mode.** Schema-constrained JSON requires inference-stack guided decoding:

- vLLM: grammar constraints, JSON Schema, or regex via the `guided_decoding` parameter.
- SGLang: grammar-based constrained decoding.
- llama.cpp: GBNF grammars.

These are stack features that work with Llama 4 at the inference layer — not a model-level JSON mode. In-prompt "respond only in JSON" does not guarantee structure.

## 7. Caching, Batch, Streaming

Llama 4 is open-weights; caching, batching, and streaming behavior are properties of the inference stack, not the model:

- **Prefix caching / KV reuse** — vLLM and SGLang both support automatic prefix caching, useful for shared system prompts.
- **Batching** — vLLM continuous batching, SGLang's scheduler; both support throughput-oriented batch inference.
- **Streaming** — SSE streaming is available from all major stacks via standard OpenAI-compatible server wrappers.

Specific parameter flags are stack-specific and not covered in the Llama-model primary sources.

## 8. Deployment Flags

### transformers

- **Minimum version**: `transformers>=4.51.0` (Llama 4 added).
- **Inference class**: `Llama4ForConditionalGeneration`.
- **Recommended attention implementation**: `attn_implementation="flex_attention"`.
- **Dtype**: BF16 native (`torch.bfloat16`).
- **INT4 quantization**: supported on-the-fly; **enables single-H100 deployment for Scout**.

```python
model = Llama4ForConditionalGeneration.from_pretrained(
    model_id,
    attn_implementation="flex_attention",
    device_map="auto",
    torch_dtype=torch.bfloat16,
)
```

[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

### vLLM and SGLang

Both support Llama 4 as of current versions. Exact minimum versions were not quoted in the retrieved primary sources. vLLM ships a dedicated JSON tool-call chat template for Llama 4 (`vllm/examples/tool_chat_template_llama4_json.jinja`) — use it for JSON-format tool-call emission.
[source: github.com/vllm-project/vllm/blob/main/examples/tool_chat_template_llama4_json.jinja, retrieved 2026-04-19]

### Context-window tuning

- **Scout native**: 10M tokens. Practical long-context deployment requires multi-GPU tensor-parallel sharding; single-H100 INT4 is not viable at the full 10M.
- **Maverick native**: 1M tokens. Requires multi-GPU.

[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]
[source: github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md, retrieved 2026-04-19]

### Safety companions

Meta's recommended safety stack for Llama 4:

- **Llama Guard 3** — input/output content filtering.
- **Prompt Guard** — adversarial prompt detection.
- **Code Shield** — code-generation safety.

Reference implementation at `github.com/meta-llama/llama-agentic-system`.
[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

Newer safety models (Llama Guard 4, Llama Prompt Guard 2) exist per the Llama model-cards index but have not been re-verified in this pass.

## 9. Deprecations and Breaking Changes

### Chat-template token migration (Llama 3.x → Llama 4)

[applies-to: meta-llama/Llama-4-Scout-17B-16E-Instruct, meta-llama/Llama-4-Maverick-17B-128E-Instruct]

- `<|start_header_id|>` / `<|end_header_id|>` → `<|header_start|>` / `<|header_end|>`
- `<|eot_id|>` → `<|eot|>`
- Image-related tokens (`<|image_start|>`, `<|image_end|>`, `<|image|>`, `<|patch|>`, `<|tile_x_separator|>`, `<|tile_y_separator|>`) are new in Llama 4 (multimodal was not present in Llama 3 base).

[source: github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md, retrieved 2026-04-19]
[source: www.llama.com/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-04-19]

### License clauses

- **Llama 4 Community License Agreement** (custom commercial license). Not OSI-approved.
- **700M MAU clause**: callers whose products had > 700M monthly active users on 2025-04-05 (the Llama 4 release date) must obtain a separate license from Meta.
- **Attribution**: "Built with Llama" required on website/UI/documentation. Derivative models must start with "Llama" in their name.
- **Jurisdiction**: Meta Platforms Ireland (EEA/Switzerland) and Meta Platforms Inc. (elsewhere). California law governs.
- **Acceptable Use Policy**: https://www.llama.com/llama4/use-policy.

[source: huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct, retrieved 2026-04-19]

## 10. Gaps

- **Sampling parameter defaults** for Llama 4 are not published by Meta in the retrieved primary sources.
- **vLLM and SGLang minimum versions** for Llama 4 support were not quoted in this retrieval pass.
- **Behemoth release status** as of 2026-04-19: the 2025-04-05 model card said "still training"; no quoted update in the current retrieval.
- **Llama 5** has not been announced as of this retrieval.
- **Llama Guard 4 / Prompt Guard 2 detailed integration** with the Llama 4 stack was not fetched; the Llama 4 model card still references Guard 3 / Prompt Guard / Code Shield.
- **Exact tool-response format for built-in tools** (if any — Meta's docs for Llama 4 did not surface `brave_search` / `wolfram_alpha` / `code_interpreter` built-ins as in Llama 3.x).
- **Environment declarations** (`Environment: ipython` headers known from Llama 3.x) are not documented in the retrieved Llama 4 prompt-format primary.
- **Multi-GPU tensor-parallel configuration guidance** for long-context Scout deployment (10M tokens) is stack-specific and not pulled here.
- **Guided-decoding JSON mode integration examples** with vLLM / SGLang are not covered in depth.

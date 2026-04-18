---
family: qwen
scope: api
versions:
  - qwen3.6-plus
  - qwen3.6-plus-2026-04-02
  - Qwen/Qwen3.6-35B-A3B
  - Qwen/Qwen3.6-35B-A3B-FP8
retrieved: 2026-04-18
primary_sources:
  - https://huggingface.co/Qwen/Qwen3.6-35B-A3B
  - https://huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json
  - https://github.com/QwenLM/Qwen3.6
  - https://www.alibabacloud.com/blog/qwen3-6-plus-towards-real-world-agents_603005
  - https://www.alibabacloud.com/help/en/model-studio/models
maturity_note: |
  Qwen3.6 is current as of this retrieval. Some of the deepest API-layer
  guidance (structured output grammar specifics, full caching/batch surface,
  exact Alibaba Cloud Model Studio endpoint URLs) is not fully present in the
  retrieved primary sources and appears in the "Gaps" section below rather
  than being paraphrased from Tier 2/3 sources.
---

# Qwen — API-Layer Reference

API-call-level detail for the Qwen3.6 generation. Portable prompt-layer content (model selection rules, role conventions, behavioral quirks) lives in `qwen-prompt.md`.

## 1. API Surface

### Open-weights deployments

Supported inference stacks, with minimum versions recommended by the Qwen team:

- **Hugging Face `transformers`** with the `serving` extra. Invoke as `transformers serve --port 8000 --continuous-batching` and `transformers chat Qwen/Qwen3.6-35B-A3B`.
- **vLLM ≥ 0.19.0**. Install with `uv pip install vllm --torch-backend=auto`.
- **SGLang ≥ 0.5.10**. Install with `uv pip install sglang[all]`.
- **KTransformers** via the Qwen3.5 deployment guide (see upstream for 3.6).

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
[source: github.com/QwenLM/Qwen3.6, README, retrieved 2026-04-18]

### Closed — Alibaba Cloud Model Studio

Model Studio exposes Qwen through three concurrently-available API surfaces:

- OpenAI-compatible chat completions
- Anthropic-compatible protocol
- Native DashScope

All three accept the same model IDs (e.g. `qwen3.6-plus`, `qwen3.6-plus-2026-04-02`). Regional endpoints depend on account region:

| Region              | Data center                                 |
|---------------------|---------------------------------------------|
| International       | Singapore                                   |
| Global              | US (Virginia) or Germany (Frankfurt)        |
| US                  | US (Virginia)                               |
| Chinese Mainland    | Beijing                                     |
| Hong Kong           | Hong Kong                                   |
| EU                  | Germany (Frankfurt)                         |

[source: www.alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]
[source: www.alibabacloud.com/blog/qwen3-6-plus-towards-real-world-agents_603005, retrieved 2026-04-18]

Exact endpoint URL paths per compatibility mode are covered in Gaps (§10) — they are not quoted verbatim in the retrieved primary excerpts. Resolve against the current Model Studio documentation before shipping integration code.

### Model IDs

Dated versions are available alongside the rolling ID:

```
qwen3.6-plus                 # rolling
qwen3.6-plus-2026-04-02      # pinned

Qwen/Qwen3.6-35B-A3B         # open weights, BF16
Qwen/Qwen3.6-35B-A3B-FP8     # open weights, FP8
```

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, retrieved 2026-04-18]
[source: www.alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]

## 2. Chat Template / Message Structure

Qwen3.6 uses a ChatML-derived template with a `Qwen2Tokenizer`. BOS is not set (`add_bos_token: false`); EOS is `<|im_end|>`; pad is `<|endoftext|>`. Native context in the tokenizer config is 262,144.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json, retrieved 2026-04-18]

### Special tokens

| Token                  | ID     | Purpose                                              |
|------------------------|--------|------------------------------------------------------|
| `<\|endoftext\|>`      | 248044 | Pad / end-of-text                                    |
| `<\|im_start\|>`       | 248045 | Message delimiter (open)                             |
| `<\|im_end\|>`         | 248046 | Message delimiter (close); also EOS                  |
| `<think>` / `</think>` | 248068 / 248069 | Reasoning block wrapper                      |
| `<tool_call>` / `</tool_call>` | 248058 / 248059 | Tool-call wrapper (JSON payload inside)   |
| `<\|vision_start\|>` / `<\|vision_end\|>` | 248053 / — | Vision content delimiters            |
| `<\|image_pad\|>`      | 248056 | Image placeholder                                    |
| `<\|video_pad\|>`      | —      | Video placeholder                                    |
| `<\|vision_pad\|>`     | —      | General vision-content placeholder                   |
| `<\|audio_start\|>`    | 248070 | Audio content delimiter (open)                       |
| `<\|audio_pad\|>`      | —      | Audio placeholder                                    |

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json, retrieved 2026-04-18]

### Role semantics

The template recognizes four roles with strict positional rules:

- `system` — at most one; must be the first message if present; cannot contain image or video content.
- `user` — user query; may contain text and multimodal content.
- `assistant` — model output; may contain `<think>...</think>` blocks and `<tool_call>...</tool_call>` blocks.
- `tool` — tool-execution output; rendered as `<tool_response>...</tool_response>` inside the surrounding user-turn context, not as a standalone role block.

The template requires at least one `user` message (it rejects conversations that are entirely `tool` responses).
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json, retrieved 2026-04-18]

### Generation prompt

When the template is rendered with `add_generation_prompt=true`, it appends `<|im_start|>assistant\n<think>` (the opening thinking block) unless `enable_thinking` is explicitly `false`. This is the mechanism that makes thinking on-by-default.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json, retrieved 2026-04-18]

### Template kwargs

The template references these optional kwargs; none are hardcoded defaults, they must be passed at render time:

- `enable_thinking` — suppress the opening `<think>` block when `false`.
- `preserve_thinking` — keep `<think>` blocks from prior-turn assistant messages when `true`; otherwise prior reasoning is stripped before re-tokenization.
- `add_vision_id` — optional "Picture N:" labelling for multi-image inputs.
- `add_generation_prompt` — standard HF kwarg; controls whether the assistant header is appended.

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json, retrieved 2026-04-18]

## 3. Sampling Parameters

The Qwen team publishes four distinct sampling-parameter sets on the model card. Task and mode both matter; picking by mode alone misses a material axis.

### Thinking mode — general tasks

```
temperature=1.0, top_p=0.95, top_k=20, min_p=0.0,
presence_penalty=1.5, repetition_penalty=1.0
```

### Thinking mode — precise coding (e.g. WebDev)

```
temperature=0.6, top_p=0.95, top_k=20, min_p=0.0,
presence_penalty=0.0, repetition_penalty=1.0
```

### Instruct (non-thinking) — general tasks

```
temperature=0.7, top_p=0.8, top_k=20, min_p=0.0,
presence_penalty=1.5, repetition_penalty=1.0
```

### Instruct (non-thinking) — reasoning tasks

```
temperature=1.0, top_p=1.0, top_k=40, min_p=0.0,
presence_penalty=2.0, repetition_penalty=1.0
```

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Output length

- Default budget: **32,768** tokens.
- Highly complex tasks (competition math, programming contests): **81,920** tokens.

Under-budgeting with thinking enabled is the primary cause of truncated final responses. Allocate for thinking + answer together.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Framework variance

Support for individual sampling parameters varies by inference stack. The Qwen team explicitly flags that `min_p`, `presence_penalty`, and some combinations behave differently across vLLM, SGLang, Transformers, and DashScope. Validate sampling behavior against the specific stack in use.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Greedy decoding

Pure greedy decoding (`temperature=0` or equivalent) is a documented failure mode: the model falls into repetition loops. Always sample.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
[testable: id=qwen.greedy-repetition.v1, expected=temperature=0 top_p=1.0 top_k=1 on a non-trivial prompt yields repeating n-grams within the first 512 output tokens]

## 4. Reasoning / Thinking Control

Thinking is enabled by default in Qwen3.6. There is no single global switch — the parameter name and placement vary by deployment.

### Open-weights (transformers / vLLM / SGLang)

Pass through `chat_template_kwargs` at rendering time:

```python
tokenizer.apply_chat_template(
    messages,
    add_generation_prompt=True,
    chat_template_kwargs={
        "enable_thinking": False,       # suppress <think> block
        "preserve_thinking": True,      # retain prior-turn <think> for multi-turn
    },
)
```

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Alibaba Cloud Model Studio (OpenAI-compatible)

Both parameters live at the top level of `extra_body`, not nested under `chat_template_kwargs`:

```python
client.chat.completions.create(
    model="qwen3.6-plus",
    messages=[...],
    extra_body={
        "enable_thinking": False,
        "preserve_thinking": True,
    },
)
```

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Defaults

[applies-to: Qwen/Qwen3.6-35B-A3B, qwen3.6-plus]

- `enable_thinking`: defaults to `True` (thinking on).
- `preserve_thinking`: defaults to `False` (prior-turn `<think>` blocks stripped). This is a change from the implicit behavior of the earlier Qwen3 ecosystem.

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
[testable: id=qwen.preserve-thinking-default-false.v1, expected=multi-turn conversation with prior-turn assistant <think> blocks renders without those blocks in the next input unless preserve_thinking=True is passed]

### Soft switches removed

[applies-to: Qwen/Qwen3.6-35B-A3B, qwen3.6-plus]

`/think` and `/nothink` in-message tokens — which toggled thinking in Qwen3 — **do not work in Qwen3.6**. The model card is explicit:

> Qwen3.6 does not officially support the soft switch of Qwen3, i.e., `/think` and `/nothink`.

Use `enable_thinking` instead.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Server-side thinking-block parsing

When running vLLM or SGLang, pass `--reasoning-parser qwen3` to have the server separate the `<think>` content from the final response on the wire. Without this flag the thinking content is included inline in the response string.
[source: github.com/QwenLM/Qwen3.6, README, retrieved 2026-04-18]

## 5. Tool Use / Function Calling

### Wire format

Tool calls are emitted as JSON payloads inside `<tool_call>...</tool_call>` blocks within the assistant's message. Tool-execution results come back as `tool`-role messages, which the chat template rewrites to `<tool_response>...</tool_response>` inline within the following user-turn context.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json, retrieved 2026-04-18]

### Parser flags

vLLM:

```
--enable-auto-tool-choice --tool-call-parser qwen3_coder
```

SGLang:

```
--tool-call-parser qwen3_coder
```

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

Note the reasoning parser and tool-call parser have different names: `qwen3` for thinking-block separation, `qwen3_coder` for tool calls. Passing `--tool-call-parser qwen3` will not work.
[source: github.com/QwenLM/Qwen3.6, README, retrieved 2026-04-18]
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Orchestration: Qwen-Agent

The upstream `Qwen-Agent` project ([github.com/QwenLM/Qwen-Agent](https://github.com/QwenLM/Qwen-Agent)) is the reference orchestration layer. Tools can be registered as MCP server configs:

```python
tools = [{
    "mcpServers": {
        "filesystem": {
            "command": "npx",
            "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"],
        },
    },
}]
bot = Assistant(llm=llm_cfg, function_list=tools)
```

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Parallel tool calls

Parallel tool-call behavior in Qwen3.6 is not explicitly documented in the retrieved primary sources. Treat parallelism as unverified until confirmed against current docs.
[unverified] Qwen3.6 supports emitting multiple `<tool_call>` blocks in a single assistant turn.

## 6. Structured Outputs

The retrieved primary sources do not give a definitive structured-output recipe for Qwen3.6 beyond the below. Treat this section as partial until re-verified.

- Alibaba Cloud Model Studio exposes a `result_format` parameter on the native DashScope API; exact semantics and supported schemas are not quoted in the retrieved excerpts.
[source: www.alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]
- For open-weights deployments, vLLM and SGLang both support generic grammar-constrained decoding (JSON schema, regex, EBNF) independent of the model. These work with Qwen3.6 at the inference-stack level.
[unverified] A Qwen-specific `json_object` or equivalent strict-JSON flag distinct from the generic vLLM/SGLang constraint is not documented in the retrieved sources.

For production JSON use: prefer grammar-constrained decoding via the inference server over in-prompt "respond only in JSON" instructions, which do not guarantee structure. Benchmark against a known-good schema; Qwen does not emit a model-level JSON-mode guarantee.

## 7. Caching, Batch, Streaming

- **Context caching**, Alibaba Cloud Model Studio: `qwen3-coder-plus` lists a 20% discount for implicit cache hits and 10% for explicit cache hits in its pricing table. Similar caching on other tiers is not confirmed in the retrieved excerpts.
[source: www.alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]

- **Streaming**: not quoted verbatim in the retrieved excerpts; OpenAI-compatible and Anthropic-compatible endpoints conventionally support SSE streaming and the compatibility claim implies it.
[unverified] SSE streaming works identically on Qwen's OpenAI-compatible endpoint as on the OpenAI reference.

- **Batch**: Alibaba Cloud Model Studio's batch API surface is not documented in the retrieved excerpts. See §10.

## 8. Deployment Flags

### vLLM

```
vllm serve Qwen/Qwen3.6-35B-A3B \
    --port 8000 \
    --tensor-parallel-size 4 \
    --max-model-len 262144 \
    --reasoning-parser qwen3 \
    --enable-auto-tool-choice \
    --tool-call-parser qwen3_coder
```

### SGLang

```
python -m sglang.launch_server \
    --model-path Qwen/Qwen3.6-35B-A3B \
    --port 8000 \
    --tp-size 4 \
    --context-length 262144 \
    --reasoning-parser qwen3 \
    --tool-call-parser qwen3_coder
```

[source: github.com/QwenLM/Qwen3.6, README, retrieved 2026-04-18]
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Context-window adjustments

- Reducing below the 262K default to save memory: safe down to **~128K**; below that threshold thinking quality degrades.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

- Extending above 262K (up to ~1M): enable YaRN scaling via `rope_parameters` in `config.json` (e.g. `factor: 2.0` gives ~524K) and set the environment variable appropriate to the server:
  - vLLM: `VLLM_ALLOW_LONG_MAX_MODEL_LEN=1`
  - SGLang: `SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN=1`

Static YaRN harms short-context performance; only enable when long-context use is actually required.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Long-video preprocessing

For long-video understanding with the 35B-A3B open-weights model, raise `longest_edge` in `video_preprocessor_config.json` from the conservative default to `469762048` (≈224K video tokens).
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

## 9. Deprecations and Breaking Changes

[applies-to: Qwen/Qwen3.6-35B-A3B, qwen3.6-plus]

- **`/think` and `/nothink` removed.** In-message soft switches no longer toggle thinking. Migrate to `enable_thinking`.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

- **`preserve_thinking` added with default `False`.** Callers that ran Qwen3 multi-turn agent loops and relied on implicit thinking carryover must now set `preserve_thinking=True` explicitly or accept lost reasoning context across turns.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
[source: www.alibabacloud.com/blog/qwen3-6-plus-towards-real-world-agents_603005, retrieved 2026-04-18]

- **`qwen-turbo` deprecated** in favor of `qwen-flash` on Alibaba Cloud Model Studio.
[source: www.alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]

- **Tool-call parser name differs from reasoning-parser name.** `--tool-call-parser qwen3_coder` vs. `--reasoning-parser qwen3`. A single-word parser flag for both operations is not available.
[source: github.com/QwenLM/Qwen3.6, README, retrieved 2026-04-18]
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

## 10. Gaps

- **Exact Alibaba Cloud Model Studio endpoint URL paths** for the OpenAI-compatible, Anthropic-compatible, and native DashScope surfaces are not quoted in the retrieved excerpts. Integration code must resolve these against current Model Studio documentation at authoring time.
- **Structured-output mechanics** (JSON-mode equivalent, strict schema validation, grammar hooks beyond the generic vLLM/SGLang facilities) are not documented in the retrieved sources for Qwen3.6.
- **Batch API** surface on Alibaba Cloud Model Studio is not covered in the retrieved excerpts.
- **Parallel tool-call emission** behavior is not explicitly documented for Qwen3.6; whether the model reliably emits multiple `<tool_call>` blocks in one turn is unverified.
- **SSE streaming details** for the OpenAI-compatible and Anthropic-compatible endpoints (event shape, `content_block_delta` vs OpenAI `delta` semantics on the Anthropic-compatible path) are not quoted.
- **Context-caching** beyond `qwen3-coder-plus` is not documented per-tier in the retrieved pricing excerpt.
- **Upstream user-facing docs (`qwen.readthedocs.io`)** were not reachable at retrieval time; several API-level behaviors that would normally be covered there (fine-grained error codes, rate-limit shapes, field-by-field OpenAI-compat deviations) are consequently gaps in this reference.

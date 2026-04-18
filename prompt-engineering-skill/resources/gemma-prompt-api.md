---
family: gemma
scope: api
versions:
  - google/gemma-4-E2B
  - google/gemma-4-E2B-it
  - google/gemma-4-E4B
  - google/gemma-4-E4B-it
  - google/gemma-4-26B-A4B
  - google/gemma-4-26B-A4B-it
  - google/gemma-4-31B
  - google/gemma-4-31B-it
retrieved: 2026-04-19
primary_sources:
  - https://ai.google.dev/gemma/docs/core/model_card_4
  - https://ai.google.dev/gemma/docs/core/prompt-formatting-gemma4
  - https://ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4
  - https://huggingface.co/google/gemma-4-E4B-it
maturity_note: |
  Gemma 4 is open-weights. "API" here means the chat-template contract plus
  the inference-stack surface (transformers, vLLM, SGLang, LiteRT), not a
  provider-hosted HTTP endpoint. Two breaking changes from Gemma 3 dominate
  this file: the new `<|turn>` / `<turn|>` control tokens replace
  `<start_of_turn>` / `<end_of_turn>`, and native function calling is added
  with its own token protocol. Exact vLLM / SGLang minimum versions for
  Gemma 4 chat-template support were not quoted in the retrieved primary
  sources and appear in Gaps.
---

# Gemma — API-Layer Reference

Inference-layer detail for Gemma 4. Portable prompt-layer content (selection, multimodal placement, thinking-mode handling, anti-patterns) lives in `gemma-prompt.md`.

## 1. API Surface

Gemma 4 has no official Google-hosted API surface for direct use (in contrast to Gemini). Deployment paths:

- **HuggingFace `transformers`** via `AutoProcessor`, `AutoModelForCausalLM` (text-only), `AutoModelForMultimodalLM` (multimodal).
- **vLLM** — supported; exact minimum version not quoted in retrieved primary sources.
- **SGLang** — supported; exact minimum version not quoted.
- **Ollama** — model weights available through the Ollama registry.
- **Kaggle** — official model-weight hosting.
- **LiteRT** (Google's on-device inference runtime) — E-series variants (`litert-community/gemma-4-E2B-it-litert-lm`) target mobile / edge.
- **llama.cpp / GGUF** — community quantizations available (`unsloth/gemma-4-E2B-it-GGUF` etc.).

[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

### Model IDs (HuggingFace)

```
google/gemma-4-E2B         google/gemma-4-E2B-it
google/gemma-4-E4B         google/gemma-4-E4B-it
google/gemma-4-26B-A4B     google/gemma-4-26B-A4B-it
google/gemma-4-31B         google/gemma-4-31B-it
```

The `-it` suffix is the instruction-tuned variant. The `A4B` in `26B-A4B` is "4B active" — consistent with the MoE expert configuration (8 of 128 experts active + 1 shared).
[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]

### Inference-class usage

Text-only:

```python
from transformers import AutoProcessor, AutoModelForCausalLM

MODEL_ID = "google/gemma-4-E4B-it"
processor = AutoProcessor.from_pretrained(MODEL_ID)
model = AutoModelForCausalLM.from_pretrained(
    MODEL_ID,
    dtype="auto",
    device_map="auto",
)
```

Multimodal (image, audio, video, text):

```python
from transformers import AutoProcessor, AutoModelForMultimodalLM

processor = AutoProcessor.from_pretrained(MODEL_ID)
model = AutoModelForMultimodalLM.from_pretrained(
    MODEL_ID,
    dtype="auto",
    device_map="auto",
)
```

Install extras for multimodal:

```bash
pip install -U transformers torch torchvision librosa accelerate
```

[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

## 2. Chat Template / Message Structure

Gemma 4 uses a special-token chat template with asymmetric turn delimiters.

### Complete control-token table

| Token                           | Purpose                                                            |
|---------------------------------|--------------------------------------------------------------------|
| `<|turn>`                       | Open a dialogue turn (pipe after `<`)                              |
| `<turn|>`                       | Close a dialogue turn (pipe before `>`)                            |
| `<|image|>`                     | Image-embedding placeholder                                        |
| `<|audio|>`                     | Audio-embedding placeholder                                        |
| `<|think|>`                     | System-prompt flag to enable thinking mode                         |
| `<|channel>` / `<channel|>`     | Wrap internal thought block (always followed by `thought`)         |
| `<|tool>` / `<tool|>`           | Wrap tool-declaration block                                        |
| `<|tool_call>` / `<tool_call|>` | Wrap model's tool-call emission                                    |
| `<|tool_response>` / `<tool_response|>` | Wrap tool-response injection                               |
| `<|"|>`                         | String-value delimiter inside structured tool data                 |

[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

The asymmetric `<|turn>` / `<turn|>` pattern (pipe position flips) applies consistently across all paired tokens above. Preserve the exact form — tokenizers are strict.

### Roles

`system`, `user`, `model`. **Use `model`, not `assistant`.**
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]
[testable: id=gemma.role-is-model.v1, expected=chat template rendered with role "assistant" produces a different token sequence than with role "model"]

### Basic template

```
<|turn>system
{system content}<turn|>
<|turn>user
{user message}<turn|>
<|turn>model
{model response}<turn|>
```

[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

### Migration from Gemma 3

[applies-to: google/gemma-4-E2B-it, google/gemma-4-E4B-it, google/gemma-4-26B-A4B-it, google/gemma-4-31B-it]

| Gemma 3 token            | Gemma 4 equivalent         |
|--------------------------|----------------------------|
| `<start_of_turn>`        | `<|turn>`                  |
| `<end_of_turn>`          | `<turn|>`                  |
| (no native tool tokens)  | `<|tool_call>` family      |
| (no native thinking)     | `<|think|>` + channel tokens |

Callers who constructed raw Gemma 3 strings directly must either migrate to `<|turn>` / `<turn|>` or switch to `AutoProcessor.apply_chat_template`.
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

## 3. Sampling Parameters

Gemma 4's standardized sampling configuration, identical across text, multimodal, thinking, and non-thinking usage:

```
temperature = 1.0
top_p       = 0.95
top_k       = 64
```

[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

The uniform recommendation is a sharp simplification compared to Qwen (four task-scoped tables) or OpenAI (per-model defaults). No model-specific override is published for the 26B MoE or 31B dense variants.

## 4. Reasoning / Thinking Control

### Enabling thinking

Two equivalent mechanisms:

- **Prompt-level**: include `<|think|>` in the system-turn content.
- **Processor-level**: pass `enable_thinking=True` to `apply_chat_template`.

[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

### Thought-block format

When thinking is enabled, the model wraps internal reasoning in a channel block before the final answer:

```
<|channel>thought
{internal reasoning}<channel|>
{final answer}<turn|>
```

When thinking is disabled, the channel block may still appear as an empty stabilizer on larger variants:

```
<|channel>thought
<channel|>{final answer}<turn|>
```

[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

### Multi-turn history handling

Strip thoughts from prior-turn `model` messages before sending the next request. Retain thoughts only within a single tool-call sequence (thought → tool call → tool response → final answer). This is the **opposite** of Claude's rule, where thinking blocks must be preserved in tool loops across turns.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]
[testable: id=gemma.strip-historical-thoughts.v1, expected=multi-turn conversations with retained prior-turn thoughts produce degraded responses vs stripped history]

### No token budget control

There is no `thinkingBudget` / `thinkingLevel` / `effort` parameter in Gemma 4. Thinking is binary (on or off). Modulate depth via prompt content, not a parameter.
[source: ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4, retrieved 2026-04-19]

## 5. Tool Use / Function Calling

### Tool declarations

Two declaration styles:

- **JSON Schema** — explicit dictionary with `name`, `description`, typed parameters, `required` fields.
- **Python function with type hints** — the processor auto-generates JSON Schema from Google-style docstrings.

Declarations are placed inside the system turn, wrapped in `<|tool>...<tool|>` blocks, alongside any `<|think|>` token.

[source: ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4, retrieved 2026-04-19]

### Model's tool-call emission

```
<|tool_call>call:FUNCTION_NAME{param_name:<|"|>value<|"|>}<tool_call|>
```

Example:

```
<|tool_call>call:get_current_weather{location:<|"|>Tokyo, JP<|"|>}<tool_call|>
```

[source: ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4, retrieved 2026-04-19]

### Tool-response injection

```
<|tool_response>response:FUNCTION_NAME{field:value,string_field:<|"|>text<|"|>}<tool_response|>
```

At the message-dict level, tool calls and responses are encoded on the `assistant` role dict via `tool_calls` and `tool_responses` arrays, which the processor renders to the token form above.
[source: ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4, retrieved 2026-04-19]

### String escaping

Every string value inside tool-call and tool-response blocks is wrapped with `<|"|>`. Standard JSON escaping does not apply. Parsers must handle this delimiter explicitly.
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

### Full conversation flow

A complete weather-query trace:

```
<|turn>system
<|think|>You are a helpful assistant.<|tool>declaration:get_current_weather{...}<tool|><turn|>
<|turn>user
What's the temperature in London?<turn|>
<|turn>model
<|channel>thought
...<channel|><|tool_call>call:get_current_weather{location:<|"|>London<|"|>}<tool_call|><|tool_response>response:get_current_weather{temperature:15,weather:<|"|>sunny<|"|>}<tool_response|>
The temperature in London is 15 degrees and it is sunny.<turn|>
```

Note that the `<|tool_response>` block is appended by the **caller's code** before re-sending, not emitted by the model. The model resumes generation after the tool response closes.
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

### Parallel calls

Multiple tool calls can appear in a single model turn (expressed as multiple `tool_calls` entries on the message dict). Responses are appended similarly.
[source: ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4, retrieved 2026-04-19]

### FunctionGemma specialization

A separate `google/functiongemma-270m-it` variant exists as a dedicated small-footprint function-calling model. Gemma 4 proper handles function calling natively without requiring FunctionGemma; reserve FunctionGemma for extreme latency / memory constraints.
[source: ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4, retrieved 2026-04-19]

## 6. Structured Outputs

No dedicated structured-output mode is documented for Gemma 4 in the retrieved primary sources. For schema-constrained JSON output:

- **vLLM** — grammar / JSON-Schema / regex constraints via `guided_decoding`.
- **SGLang** — grammar-constrained decoding.
- **llama.cpp** — GBNF grammars.

These are inference-stack features independent of the model. In-prompt "respond only in JSON" does not guarantee structure.

The native tool-calling protocol (§5) offers stronger output-shape guarantees than free-form JSON for structured data extraction — define the extraction task as a tool and require invocation.

## 7. Caching, Batch, Streaming

Open-weights model — caching, batching, and streaming are inference-stack properties:

- **Prefix caching / KV reuse** — vLLM and SGLang both support automatic prefix caching for shared system prompts.
- **Continuous batching** — vLLM and SGLang schedulers support throughput-oriented batch inference.
- **SSE streaming** — standard OpenAI-compatible server wrappers over vLLM / SGLang.
- **LiteRT on-device** — for mobile / edge deployment; separate performance model.

No provider-hosted caching or batch discount applies. Specific flag configurations are stack-specific and not covered in Gemma-model primary sources.

## 8. Deployment Flags

### transformers

- **Install**: `pip install -U transformers torch accelerate` (text); add `torchvision librosa` for multimodal.
- **Classes**: `AutoProcessor`, `AutoModelForCausalLM` (text-only), `AutoModelForMultimodalLM` (multimodal).
- **Entry point**: `processor.apply_chat_template(messages, tokenize=True, return_dict=True, return_tensors="pt", add_generation_prompt=True, enable_thinking=<bool>)`.
- **Response parsing**: `processor.parse_response(decoded_text)` handles thought-channel extraction and tool-call parsing.

[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

### Context-window and memory

- Sliding window sizes: **512 tokens** (E-series) / **1024 tokens** (26B/31B). Interleaved with global-attention layers (final layer always global), delivering lightweight memory at long context.
- Per-Layer Embeddings (PLE) on E-series reduce active memory — the "effective" parameter number is what matters for deployment sizing.

[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

### Modality-specific deployment notes

- **Audio**: E2B / E4B only. Audio encoder ~300M params on E4B.
- **Vision**: all variants. Vision encoder ~150M params on E4B.
- **Image resolution budgets**: `70`, `140`, `280`, `560`, `1120` tokens per image — trade speed vs detail.
- **Audio max length**: 30s per segment.
- **Video max length**: 60s at 1 fps.

[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

### On-device path (LiteRT)

`litert-community/gemma-4-E2B-it-litert-lm` packages Gemma 4 E2B for Google's LiteRT runtime, intended for mobile / edge deployment. API contract is the same chat template; packaging differs.

## 9. Deprecations and Breaking Changes

### Chat-template migration (Gemma 3 → Gemma 4)

[applies-to: google/gemma-4-E2B-it, google/gemma-4-E4B-it, google/gemma-4-26B-A4B-it, google/gemma-4-31B-it]

- `<start_of_turn>` → `<|turn>`
- `<end_of_turn>` → `<turn|>`
- Native thinking mode added (`<|think|>`, `<|channel>`/`<channel|>`).
- Native function-calling tokens added (`<|tool>`, `<|tool_call>`, `<|tool_response>`, `<|"|>`).
- Tool-calling no longer requires community-built chat templates — Gemma 4 ships its own first-party protocol.

[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]
[source: ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4, retrieved 2026-04-19]

### License

**Apache 2.0.** No 700M MAU clause, no regional exclusions, no attribution requirement beyond what Apache 2.0 mandates. A separate Gemma Prohibited Use Policy covers content categories; use of weights is not gated by organization size.
[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

## 10. Gaps

- **Exact Gemma 4 release date** is not quoted in the retrieved primary-source excerpts. The ai.google.dev model card was "updated" 2026-04-17 but this is a page-update date, not the initial release date.
- **vLLM / SGLang minimum versions** for Gemma 4 chat-template and native tool-call parsing support were not captured.
- **BOS / EOS token specification** is absent from the prompt-format primary.
- **MoE expert-routing behavior** for the 26B-A4B variant beyond "8 active + 1 shared of 128" was not fetched in depth.
- **Full set of multimodal token-budget trade-offs** (quality impact per resolution tier) is not quantitatively documented in retrieved sources.
- **FunctionGemma 270M-it** operational differences (latency, memory, quality envelope vs Gemma 4 E2B with function calling) were not pulled.
- **Training token count** is not quoted numerically.
- **Fine-tuning recipes** for instruction-tuning continued from Gemma 4-it are out of scope here.

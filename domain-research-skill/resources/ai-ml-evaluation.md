---
name: ai-ml-evaluation
description: Source taxonomy for AI/ML model and hardware evaluation — published benchmarks with harness and hardware specificity, model cards, spec sheets, MLPerf submissions. Distinct from vendor marketing.
domain: ai-ml-evaluation
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://mlcommons.org/
  - https://mlcommons.org/2026/04/mlperf-inference-v6-0-results/
  - https://huggingface.co/
  - https://paperswithcode.com/
---

# AI/ML Model and Hardware Evaluation — Source Taxonomy

Layer for evaluation of AI/ML models and the hardware and inference stacks that run them. Activates on benchmark claims, model-capability comparisons, accelerator selection, and inference-cost modeling. Vendor-produced marketing numbers are accepted only with the reproduction recipe they cite.

## Activation Signals

- User is comparing models on benchmark scores (MMLU-Pro, MMMU, SWE-Bench Verified, ARC-AGI-2, HumanEval, MATH, GPQA, HELM, AgentBench, etc.).
- User is evaluating accelerator options (NVIDIA H200 / B200 / GB200, AMD MI300X / MI325X / MI355, Google TPU v5e / v5p / v6e, AWS Trainium / Inferentia).
- User is benchmarking inference throughput / latency against a specific stack (vLLM, SGLang, TensorRT-LLM, TGI, llama.cpp).
- User is sizing infrastructure against cost / throughput / TCO estimates.

<Source_Domain role="ai-ml-evaluation">

<Tier_1_Primary>
- MLPerf submitted results (mlcommons.org/benchmarks) — cited with the benchmark (MLPerf Training, MLPerf Inference Datacenter / Edge), round, submission ID, hardware SKU, software stack version, harness (LoadGen++ as of Inference v6.0), and accuracy target. Current Inference round: **v6.0**, released 2026-04-01, introducing LoadGen++ and adding or updating GPT-OSS 120B, DeepSeek-R1 (with speculative-decoding interactive scenario), DLRMv3 (first sequential recommendation test), Text-to-Video Generation (the suite's first), VLM (multimodal product-data-to-metadata), and YOLOv11 Large (edge). [source: https://mlcommons.org/2026/04/mlperf-inference-v6-0-results/, retrieved 2026-04-19]
- Eval-harness results accompanied by the harness commit hash:
  - LM Evaluation Harness (github.com/EleutherAI/lm-evaluation-harness), cited with harness commit SHA and task name.
  - BIG-bench and BIG-bench Hard (github.com/google/BIG-bench) with commit.
  - OpenAI simple-evals (github.com/openai/simple-evals) with commit.
  - Inspect AI (inspect.aisi.org.uk) with version.
- Benchmark-authoritative paper and leaderboard:
  - MMLU-Pro (paper + Hugging Face leaderboard with dataset version).
  - MMMU (mmmu-benchmark.github.io) with dataset version.
  - SWE-Bench / SWE-Bench Verified / SWE-Bench Multimodal (swebench.com) with task split and evaluation harness.
  - ARC-AGI-2 (arcprize.org) with season and submission date.
  - GPQA (arxiv.org/abs/2311.12022) with dataset version (Diamond, Main, Extended).
  - HumanEval (github.com/openai/human-eval) with subset.
  - MATH (github.com/hendrycks/math) with subset and temperature.
  - HELM (crfm.stanford.edu/helm/) with scenario and version.
  - Chatbot Arena (chat.lmsys.org) Elo rankings with the arena snapshot date and sample size.
- Provider model cards on HuggingFace or the provider's own domain, pinned to commit / published revision. Model-card weights-license statements are primary for licensing claims.
- Papers-with-Code entries (paperswithcode.com) tied back to the original paper (primary for the paper; the leaderboard is a secondary aggregator).
- Provider technical reports and research papers (arxiv preprint with version, or conference proceedings).
- Accelerator and inference-stack documentation:
  - NVIDIA datasheets (nvidia.com/en-us/data-center/) with part numbers (H100 SXM5 80GB, H200 SXM5 141GB, B100, B200, GB200 NVL72, etc.).
  - AMD Instinct datasheets (amd.com/en/products/accelerators).
  - Google Cloud TPU documentation (cloud.google.com/tpu/docs) with TPU generation.
  - AWS Trainium / Inferentia documentation (aws.amazon.com/machine-learning/trainium/).
  - vLLM documentation and release notes (github.com/vllm-project/vllm) with release tag.
  - SGLang (github.com/sgl-project/sglang), TensorRT-LLM (github.com/NVIDIA/TensorRT-LLM), TGI (github.com/huggingface/text-generation-inference), llama.cpp (github.com/ggerganov/llama.cpp).
- CUDA / ROCm / XLA / OpenVINO release notes tied to version.
- Public audits and reproductions of published benchmarks where the reproducer published full methodology.
</Tier_1_Primary>

<Tier_2_Contextual>
- Named independent reviewer writeups: Chips and Cheese, ServeTheHome, SemiAnalysis (research posts; distinct from their paywalled subscriptions), TechTechPotato.
- Independent practitioner posts on evaluation: Simon Willison (simonwillison.net), Hamel Husain, Thomas Capelle (on Weights & Biases), Nathan Lambert (Interconnects), Sebastian Raschka (Ahead of AI).
- Reproducible community leaderboards with full harness + seed metadata (e.g., Open LLM Leaderboard v2 on Hugging Face).
- Discord / forum reports tagged as community-observed.
- Conference talk recordings (NeurIPS, ICLR, MLSys, SC).
</Tier_2_Contextual>

<Disqualified>
- "Best AI models 2024/2025" SEO listicles.
- AI-generated comparison content without harness or provenance.
- Vendor benchmark claims without harness or reproduction recipe.
- Crypto / GPU-mining hype aggregators cross-posting AI training numbers.
- Undated benchmark claims.
- Charts without axis labels, hardware specs, or precision (FP8 / FP16 / BF16 / INT8 / FP4) callouts.
- "AI agent" leaderboards that change task definitions between entries.
- "Beats GPT-4" claims on unspecified tasks.
- Reddit / Twitter vibes-based rankings.
</Disqualified>

<Evidence_Threshold>
- Benchmark claim: harness name + commit SHA or version (for MLPerf: LoadGen++ beginning Inference v6.0; LoadGen for earlier rounds), dataset version, date of run, hardware SKU, quantization / precision, seeds (where non-deterministic), prompt or prompt-template version, and reported variance or confidence interval where the benchmark provides one. MLPerf claims must pin the round (e.g., `Inference v6.0`) and the submission ID.
- Hardware-performance claim: datasheet citation with part number and revision, or MLPerf submission ID. Single-number claims without precision and workload are rejected.
- Inference-throughput / latency claim: tokens/sec (input vs output separately), batch size, sequence length, KV-cache / PagedAttention / prefix-caching settings, and the stack version (vLLM X.Y.Z, SGLang A.B, TRT-LLM C).
- Cost / TCO claim: hourly accelerator cost basis with cloud provider / on-prem-depreciation-model, utilization assumption, and throughput measurement cited separately.
- Model-capability claim: provider model card, paper, or benchmark result with the model's public release version and date.
- License claim: model card license field quoted, or the accompanying LICENSE file at the specific commit.
- Currency requirement: benchmarks update frequently; Hugging Face Open LLM Leaderboard retired v1 and now runs v2; SWE-Bench has had multiple refinements including Verified; ARC-AGI-2 runs in seasons; MLPerf Inference rounds are roughly quarterly (v6.0 released 2026-04-01). Cite the specific leaderboard / round / season version. Accelerator generations ship fast; re-verify against the current generation on the vendor's product page.
</Evidence_Threshold>

</Source_Domain>

## Gaps

- Many recent evals on agentic coding and long-horizon planning lack widely-adopted harnesses; claims on those tasks will be flagged as preliminary until a reproducible harness exists.
- Cost-per-token comparisons across closed APIs are provider-published; use them with the date and pricing-tier, and recognize that provider list price differs from negotiated enterprise price.
- Training-cost claims are often vendor-reported without reproduction recipe; flag them as vendor-reported rather than benchmarked.
- Benchmark contamination (train / test overlap) is a structural concern; when citing a benchmark number, note any published contamination analyses or treat the number with the contamination caveat.

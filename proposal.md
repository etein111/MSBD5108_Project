# Initial Project Scope Summary

| Surname | Given name | Student ID |
| --- | --- | --- |
| GONG | Ruiting | 21319234 |
| Ye | Chuan | 21317834 |
| Zheng | Zhiqi  | 21329954 |

## Topic introduction

Jev a specialized System One decision model trained via Reinforcement Learning for Calibrated Decisions has emerged as a dedicated alternative to using general-purpose LLMs (e.g., **Qwen3-direct**) as safety judges. Unlike generative architectures that rely on verbose, token-by-token autoregressive reasoning, Jev directly predicts discrete decision categories paired with native probability distributions, targeting key LLM bottlenecks in inference latency, logit calibration, and adversarial vulnerability.

## Core Research Question

**Core Question:**

**Sub-questions:**

- 
- 
- 

## Project Boundaries

This project focuses on evaluating Jev's effectiveness in typed content-safety decisions, covering spam detection, phishing detection, and toxicity assessment. The tasks include binary classification (safe/unsafe) and ordinal severity scoring (1–5), using publicly available datasets such as Enron, SMS Spam, Jigsaw, and ToxiGen.

The evaluation is limited to two primary systems: Jev (`JEV-27B`) and its proposed general-purpose LLM baseline (`Qwen3.8-27B`). Their performance will be compared under consistent evaluation conditions across four dimensions: **task accuracy, probability calibration, inference cost, and latency**. Robustness will also be examined through controlled adversarial perturbations, including prompt injection, option renaming, abstention removal, and option-order permutation.

The project emphasizes inference-time evaluation rather than model development. It excludes model pretraining, fine-tuning, architectural modifications, and general-purpose text generation. Post-hoc probability calibration is included, but the development of new calibration algorithms is outside the scope. The study aims to identify the practical strengths and limitations of typed decision models compared with conventional LLMs.

## Planned Methodologies and Deliverable

**Method.** Compare JEV-27B against its frozen backbone Qwen3.8-27B on identical inputs and prompt schema [3][4]; the Qwen3.8-27B base model is prompted (few-shot) with **constrained decoding** to emit the same `noul`/`score` decisions, with no chain-of-thought in the fair pairing. **Metrics:** accuracy (exact / macro-F1 / QWK-MAE) [5]; calibration (ECE + Brier, using Qwen3's yes/no token **logits** rather than verbalized confidence, with cross-fitted Platt scaling) [6][7]; cost per 1,000 decisions; p50/p95 latency. **Robustness:** re-evaluate under the adversarial probes (prompt injection, option renaming, abstain removal, option-order permutation) to test whether Jev's known weaknesses erase its cost/latency edge [1][2], with fixed seeds, repeated sampling, and 95% bootstrap CIs.

**Deliverables.** Reproducible codebase (JevBench-compatible loader, both adapters, probe generator); experiment report (accuracy/cost/latency tables, calibration plots, robustness comparison); live demo with an adversarial case.

### References

[1] Check Point, *A First Look at Jev*. [link](https://blog.checkpoint.com/ai-security/jev-is-not-a-language-model-but-it-breaks-like-one-prompt-injection-against-a-typed-decision-model/)
[2] *Jev Calibration Audit*. [GitHub](https://github.com/jujumilk3/jev-calibration-audit)
[3] AutoTrust, *JEV-27B*. [Hugging Face](https://huggingface.co/autotrust/JEV-27B)
[4] *Qwen3.8-27B*. [GitHub](https://github.com/AlibabaCloud-Official/Qwen3.8-27B)
[5] *JevBench*. [GitHub](https://github.com/model-collapse/jev-bench)
[6] Guo et al., *On Calibration of Modern Neural Networks*. [arXiv](https://arxiv.org/abs/1706.04599)
[7] *Decision Without Generation*. [Zenodo](https://zenodo.org/records/22885291)

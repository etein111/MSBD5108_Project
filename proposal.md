# Initial Project Scope Summary

| GONG | Ruiting | 21319234 |
| --- | --- | --- |
| Ye | Chuan | 21317834 |
|  |  |  |

## Topic introduction

**This project will introduce what Jev is and its overarching goal in Natural Language Processing/AI. Define the problem setup explicitly (e.g., given X input data, the task is to predict/evaluate Y). Moreover, summarize the key difficulties in current approaches when handling Jev, e.g., Agents are still inherently untrustworthy, establishing why a novel methodology is needed.**

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

**Goal.** Evaluate Jev — TypeSafe AI's "System One" typed-decision model — on a content-safety NLP suite (spam, phishing, and toxicity detection), and benchmark it head-to-head against an open-weight LLM baseline (Qwen3) along four axes: **task success rate, cost, latency, and calibration**. The core question is whether Jev's fast, calibrated, typed decisions remain the better choice than a general-purpose LLM, including under adversarial inputs that specifically target decision models. All comparisons follow an explicit fair-comparison protocol (below) rather than ad-hoc prompting.

**1. Evaluation set.** A benchmark spanning Jev's decision types, grounded in content safety:

- *Noul (yes/no):* spam detection (Enron / SMS Spam), phishing detection, and binary toxicity classification (Jigsaw).
- *Score (ordered rating):* toxicity severity and safety-risk severity (1–5) on ToxiGen [8].

All tasks reuse public datasets and a fixed prompt schema so Jev and Qwen3 see identical inputs. Data follow the model-agnostic **JevBench JSONL convention** (`noul` / `choice` / `score`), so we reuse its harness and metrics instead of re-implementing evaluation [9]; the composite **JevBench Score** (intelligence / calibration / speed / cost) gives a ready rubric for the axes below [10].

**2. Systems compared.**

- *Jev:* open-weights `autotrust/JEV-27B` served locally via vLLM [3]; the official (closed) Jev API is treated only as an optional upper bound if a key becomes available.
- *LLM baseline:* `Qwen/Qwen3.8-27B` — the frozen backbone of JEV-27B (27B dense, Apache 2.0) [7] — prompted to emit the same typed decision via constrained decoding. Since the backbone is the *base* (non-instruct) model, it uses fixed few-shot in-context examples, whereas Jev answers natively zero-shot; this asymmetry is documented and held constant.

*Decision extraction (removing the format confound).* Jev returns a typed decision natively. Qwen3 is made comparable by emitting the identical decision through **constrained decoding / structured JSON output** (same schema, no free-text parsing); the decision — and therefore accuracy, cost, and latency — is compared on this structured path.

*Chain-of-thought control.* The primary comparison is **Jev vs Qwen3-direct**, i.e. both in "System 1" mode with no reasoning, which is the only fair pairing. `Qwen3-CoT` is reported separately as the "System 2" variant (higher accuracy, higher cost and latency).

**3. Metrics.**

- *Success rate:* exact accuracy for `noul`; precision / recall / macro-F1 on the minority class (spam / phishing / toxicity), since the data are skewed; `score` uses pooled **QWK + MAE** [9].
- *Calibration (first-class):* ECE (15-bin) [13] + Brier score + reliability diagrams, computed with an off-the-shelf calibration harness [14]. Jev's per-option probability is native; Qwen3's is taken from the **yes/no token logits** (fine-grained), with verbalized confidence kept only as an ablation — this "quantisation control" is required for a fair ECE comparison [11]. Both models are additionally **recalibrated with cross-fitted Platt / temperature scaling on a held-out set**, and the post-recalibration ECE is the primary calibration result, with a preset acceptance threshold of ECE ≤ 0.05 following the pre-registered protocol in [12].
- *Cost:* cost per 1,000 decisions — tokens processed × a reference per-token price, normalized to GPU-time for locally served models so Jev and Qwen3 share one currency [10].
- *Latency:* p50 / p95 wall-clock per decision, reported for both single requests and batched serving (batching affects the two systems differently).

**4. Statistical protocol.** A fixed item set shared across both systems, fixed seeds, and repeated sampling; results carry 95% bootstrap confidence intervals, and head-to-head claims (e.g., "Jev is cheaper at equal accuracy") are backed by a significance test rather than point estimates alone [12].

**5. Boundary / adversarial probes.** We mutate the same inputs to stress decision-specific failure modes and compare the robustness of the two systems:

- *Prompt injection* — inject an instruction that tries to override the verdict; Check Point broke Jev on all 9 tested injection variants [1].
- *Option renaming* — relabel the decision options, a known manipulation that flips verdicts in decision models [2].
- *Abstain removal* — drop the "unsure / abstain" option, which independent audits show collapses Jev's accuracy (0.95 → 0.0) and calibration (ECE 0.023 → 0.793) [2].
- *Option-order permutation* — shuffle option order to test position bias [2].

This design directly tests whether Jev's known weaknesses (injection, calibration sensitivity to abstain) erase its cost/latency advantage against the LLM [4][5][6].

**6. Deliverables.**

- A reproducible codebase: a JevBench-compatible benchmark loader, Jev and Qwen3 inference adapters (structured decoding for Qwen3), and the adversarial-probe generator.
- An experiment report with success/cost/latency tables, calibration/reliability plots, and a robustness comparison under the probe suite.
- A live demo on representative spam/phishing/toxicity examples, including at least one adversarial case.

### References

[1] Check Point Research, *A Decision Model Breaks Like Any Other Language Model: A First Look at Jev* — [link](https://blog.checkpoint.com/ai-security/jev-is-not-a-language-model-but-it-breaks-like-one-prompt-injection-against-a-typed-decision-model/)

[2] *Jev Calibration Audit* (independent API-only audit) — [GitHub](https://github.com/jujumilk3/jev-calibration-audit)

[3] AutoTrust AI, *JEV-27B: An Open Decision Model for Self-Hosted AI Agents* — [Hugging Face](https://huggingface.co/autotrust/JEV-27B)

[4] LangChain, *Jev-as-a-Judge for Agent Evals* — [link](https://www.langchain.com/blog/jev-agent-evals-langsmith)

[5] Archestra, *We tested Jev on 100 real agent calls* — [link](https://archestra.ai/blog/we-tested-jev-on-100-real-agent-calls)

[6] *Just Ask Jev: Reinforcement Learning for Calibrated Decisions as a Zero-Shot Detector of AI Alignment Failures* — [arXiv](https://arxiv.org/html/2609.29429)

[7] *Qwen3.8-27B* (27B dense, Apache 2.0) — [GitHub](https://github.com/AlibabaCloud-Official/Qwen3.8-27B)

[8] Hartvigsen et al., *ToxiGen: A Large-Scale Machine-Generated Dataset for Adversarial and Implicit Hate Speech Detection* — [arXiv](https://arxiv.org/abs/2203.09509)

[9] *JevBench: an open, model-agnostic benchmark for typed decision models* — [GitHub](https://github.com/model-collapse/jev-bench)

[10] *JevBench leaderboard / composite score (intelligence, calibration, speed, cost)* — [GitHub](https://github.com/fstandhartinger/jevbench)

[11] *Decision Without Generation: Benchmarking a Calibrated Non-Generative Decision Model Against LLM Judges on Moderation, Hallucination, NLI and Prompt-Injection Tasks* — [Zenodo](https://zenodo.org/records/22885291)

[12] *Do Decision Models Know When They're Wrong? A Pre-Registered Test of Calibration and Accuracy of Seven Decision Models* — [Zenodo](https://zenodo.org/records/23293732)

[13] Guo et al., *On Calibration of Modern Neural Networks* — [arXiv](https://arxiv.org/abs/1706.04599)

[14] *laya-evals: calibration-first evaluation (ECE, Brier, reliability, threshold advisor)* — [PyPI](https://pypi.org/project/laya-evals/)

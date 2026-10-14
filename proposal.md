# Initial Project Scope Summary

| GONG | Ruiting | 21319234 |
| --- | --- | --- |
|  |  |  |
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

## Planned Methodologies and Deliverable

**Goal.** Evaluate Jev — TypeSafe AI's "System One" typed-decision model — on a content-safety NLP suite (spam, phishing, and toxicity detection), and benchmark it head-to-head against an open-weight LLM baseline (Qwen3) along three axes: **task success rate, cost, and latency**. The core question is whether Jev's fast, calibrated, typed decisions remain the better choice than a general-purpose LLM, including under adversarial inputs that specifically target decision models.

**1. Evaluation set.** A benchmark spanning Jev's decision types, grounded in content safety:

- *Noul (yes/no):* spam detection (Enron / SMS Spam), phishing detection, and binary toxicity classification (Jigsaw).
- *Score (ordered rating):* toxicity severity and safety-risk severity (1–5) on ToxiGen [8], evaluated by ranking/calibration and a thresholded F1.

All tasks reuse public datasets and a fixed prompt schema so Jev and Qwen3 see identical inputs.

**2. Systems compared.**

- *Jev:* open-weights JEV-27B served locally via vLLM [3]; the official Jev API is treated only as an optional upper bound if a key becomes available.
- *LLM baseline:* Qwen3 (open-weight, local) [7], prompted to emit the same structured decision (constrained decoding / JSON) for a fair comparison.

**3. Metrics.**

- *Success rate:* accuracy for binary tasks; precision / recall / macro-F1 on the minority class (spam / phishing / toxicity), since the data are skewed.
- *Cost:* tokens processed × a reference per-token price, normalized to GPU-time for locally served models so Jev and Qwen3 share one currency.
- *Latency:* mean wall-clock time per decision.

**4. Boundary / adversarial probes.** We mutate the same inputs to stress decision-specific failure modes and compare the robustness of the two systems:

- *Prompt injection* — inject an instruction that tries to override the verdict; Check Point broke Jev on all 9 tested injection variants [1].
- *Option renaming* — relabel the decision options, a known manipulation that flips verdicts in decision models [2].
- *Abstain removal* — drop the "unsure / abstain" option, which independent audits show collapses Jev's accuracy (0.95 → 0.0) and calibration (ECE 0.023 → 0.793) [2].
- *Option-order permutation* — shuffle option order to test position bias [2].

This design directly tests whether Jev's known weaknesses (injection, calibration sensitivity to abstain) erase its cost/latency advantage against the LLM [4][5][6].

**5. Deliverables.**

- A reproducible codebase: benchmark loader, Jev and Qwen3 inference adapters, and the adversarial-probe generator.
- An experiment report with success/cost/latency tables and a robustness comparison under the probe suite.
- A live demo on representative spam/phishing/toxicity examples, including at least one adversarial case.

### References

[1] Check Point Research, *A Decision Model Breaks Like Any Other Language Model: A First Look at Jev* — [link](https://blog.checkpoint.com/ai-security/jev-is-not-a-language-model-but-it-breaks-like-one-prompt-injection-against-a-typed-decision-model/)

[2] *Jev Calibration Audit* (independent API-only audit) — [GitHub](https://github.com/jujumilk3/jev-calibration-audit)

[3] AutoTrust AI, *JEV-27B: An Open Decision Model for Self-Hosted AI Agents* — [link](https://www.tmcnet.com/usubmit/2026/09/28/10452979.htm)

[4] LangChain, *Jev-as-a-Judge for Agent Evals* — [link](https://www.langchain.com/blog/jev-agent-evals-langsmith)

[5] Archestra, *We tested Jev on 100 real agent calls* — [link](https://archestra.ai/blog/we-tested-jev-on-100-real-agent-calls)

[6] *Just Ask Jev: Reinforcement Learning for Calibrated Decisions as a Zero-Shot Detector of AI Alignment Failures* — [arXiv](https://arxiv.org/html/2609.29429)

[7] *Qwen3 Technical Report* — [arXiv](https://arxiv.org/abs/2505.09388)

[8] Hartvigsen et al., *ToxiGen: A Large-Scale Machine-Generated Dataset for Adversarial and Implicit Hate Speech Detection* — [arXiv](https://arxiv.org/abs/2203.09509)

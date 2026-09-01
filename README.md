# Breaking the Assumptions: Auditing Input-Side Jailbreak Defenses Against Semantic Attacks

---

## Overview

This repository contains the released experimental data for our paper:

> **"Breaking the Assumptions: Auditing Input-Side Jailbreak Defenses Against Semantic Attacks"**

Locally deployed LLMs (e.g., via Ollama) run without the moderation and abuse detection present in API-served models, so their safety depends on the input-side defenses chosen at server setup. This work **audits six such defenses** under *semantic* jailbreak attacks. Some defenses publish formal guarantees (SmoothLLM, Erase-and-Check, Sequential Monitors); others rely on empirical detection results (Semantic Smoothing, Self-Denoised Smoothing, Perplexity Filtering).

Instead of merely observing that defenses fail, we trace each failure back to the **specific assumption** the defense was designed around: for every defense we (1) extract the condition it relies on, (2) derive the empirical signature a violation should produce, and (3) test that prediction on **six open-weight models (14B–35B parameters)** with a corpus of **100 jailbreak prompts** drawn from **40+ public sources**, totalling **13,800 evaluation records**.

---

## Research Questions

1. **RQ1 — Effectiveness.** Do existing defenses reduce the attack success rate (ASR) of semantic jailbreaks on locally deployed LLMs?
2. **RQ2 — Diagnosis.** For each defense, which specific design assumption fails, and what does that failure look like in the data?
3. **RQ3 — Attribution.** How much of the observed safety comes from the defense, and how much from the model's own intrinsic alignment?

---

## Setup at a Glance

| Component | Details |
|---|---|
| **Models** | Gemma 4:31b (Google DeepMind), DeepSeek-r1:32b (DeepSeek), Phi-4:14b (Microsoft), Granite 4.1:30b (IBM), OLMo-3.1:32b (Allen Institute), Qwen 3.6:35b (Alibaba) — 14B to 35B parameters |
| **Defenses** | SmoothLLM, Erase-and-Check, Self-Denoised Smoothing, Semantic Smoothing, Sequential Monitors, Perplexity Filtering |
| **Prompt corpus** | 100 prompts from 40+ public sources (Reddit, GitHub, Discord, Horselock templates, HuggingFace datasets, academic publications + self-designed); 6 attack families: role-play/persona injection, semantic paraphrasing, hypothetical scenarios, instruction-prefix modification, multi-turn decomposition |
| **Environment** | Local workstation (NVIDIA DGX Spark), Ollama, default settings, no cloud component |
| **Judge** | Llama-Guard-3:8b binary judge; human-verified subset (3 annotators, majority vote) |
| **Scale** | **13,800 evaluation records** |

### Evaluation scale (records per condition)

| Condition | Conv./model | Records | Configuration |
|---|---|---|---|
| Baseline (no defense) | 100 | 600 | single generation |
| SmoothLLM | 100 | 600 | N=10 perturbed copies, majority vote |
| Erase-and-Check | 100 | 1,800 | 3 erasure depths, filter per erasure |
| Self-Denoised Smoothing | 100 | 600 | N=7 regenerations, vote |
| Semantic Smoothing | 200 | 1,200 | M=7 rewrites, vote |
| Sequential Monitors | 1,000 | 6,000 | K=4 turns, monitor per turn |
| Perplexity Filtering | 100 | 3,000 | 5 thresholds, offline scoring |
| **Total** | — | **13,800** | — |

---

## Key Findings at a Glance

Baseline (no defense) mean ASR is **13.33%** (Phi-4: 24%, Granite 4.1: 56%; the other four models refuse everything).

| Defense | Assumption under test | Observed result | Failure type |
|---|---|---|---|
| **SmoothLLM** | Adversarial content is suffix-localized and *κ*-unstable (κ ≤ M) | Mean ASR **15.17%** (+1.84); Phi-4 jumps 24→**38%** (+14, *p*≈0.03); outcomes track model alignment, not perturbation | **Locality violation** |
| **Erase-and-Check** | Payload is a contiguous suffix of length ≤ d | ASR flat at **13.16%** across all erasure depths (ΔASR/Δm = 0); 0 intermediate outcomes; miss rate **32.16%** (95% CI 27.0–37.8%) | **Locality violation** |
| **Self-Denoised Smoothing** | Regeneration washes out adversarial content | Mean **10.66%** (lowest of all defenses); Granite 56→28% (sig.), but Phi-4 24→36% (+12, n.s.); denoiser *rebuilds* persona prompts | **Semantic preservation** |
| **Semantic Smoothing** | Meaning-preserving rewrites break attacks | Mean **16.00%** (+2.67); 192/1,200 jailbroken; certification arithmetically reachable (M=7) yet votes stay bimodal | **Semantic preservation** |
| **Sequential Monitors** | Harmful turns inject positive drift (CUSUM optimality premise) | **42.10%** jailbroken; recall **12.11%**, precision 36.9%, FP 15.1%; **87.89%** of jailbreaks never flagged | **Distributional separation failure** |
| **Perplexity Filtering** | Attacks sit in a high-perplexity tail | **0/3,000 detections** at all 5 thresholds; **100% bypass** (346 jailbreaks) | **Distributional inversion** |

### Full ASR matrix (%, per model × defense)

| Model | Baseline | SmoothLLM | E&C | Self-Den. | Sem. Smooth. | Seq. Mon.‡ | PPL Filt. |
|---|---|---|---|---|---|---|---|
| Gemma 4:31b | 0.00 | 0.00 | 4.00 | 0.00 | 0.00 | 21.0 | 5.00 |
| DeepSeek-r1:32b | 0.00 | 4.00 | 11.00 | 0.00 | 1.50 | 75.7 | 1.20 |
| Phi-4:14b | 24.00 | 38.00 | 20.00 | 36.00 | 32.00 | 56.1 | 11.00 |
| Granite 4.1:30b | 56.00 | 49.00 | 43.00 | 28.00 | 62.50 | 59.90 | 52.00 |
| OLMo-3.1:32b | 0.00 | 0.00 | 1.00 | 0.00 | 0.00 | 5.0 | 0.00 |
| Qwen 3.6:35b | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 34.9 | 0.00 |
| **Mean** | **13.33** | **15.17** | **13.16** | **10.66** | **16.00** | **42.10** | **11.53** |
| **Δ vs baseline** | — | +1.84 | −0.17 | −2.67 | +2.67 | +28.8 | −1.8 |

‡ Sequential Monitors is measured under multi-turn decomposition attacks (K=4 turns), so its absolute values are not directly comparable to the single-turn columns.

### Where defenses backfire

**11 of the 30** single-turn model–defense cells sit *above* the model's own baseline — defenses sometimes make models less safe:

| Defense | Cells above baseline | Models affected | Largest increase |
|---|---|---|---|
| SmoothLLM | 2 of 6 | DeepSeek, Phi-4 | **+14.0** |
| Erase-and-Check | 3 of 6 | Gemma, DeepSeek, OLMo | +11 |
| Self-Denoised Sm. | 1 of 6 | Phi-4 | +12 |
| Semantic Smoothing | 3 of 6 | DeepSeek, Phi-4, Granite | +8.0 |
| PPL Filtering | 2 of 6 | Gemma, DeepSeek | +5.0 |
| **Total** | **11 of 30** | all except Qwen | — |

The largest single-turn effect in the entire study is defense-induced harm: **SmoothLLM pushes Phi-4 from 24% to 38%**.

---

## The Unified Root Cause → A Taxonomy of Assumption Failures

The six defenses do not fail randomly. Each failure traces to a specific design assumption, and the six cluster into three pairs:

1. **Locality violations** — *SmoothLLM, Erase-and-Check.* Both certificates assume adversarial content is confined to a bounded suffix region. Semantic attacks spread intent across the whole prompt, so perturbation or erasure never touches the payload.
2. **Semantic preservation** — *Semantic Smoothing, Self-Denoised Smoothing.* These defenses reword or reconstruct the input, which is logically incompatible with attacks whose payload *is* the meaning. Self-Denoised Smoothing makes this plain: its denoiser is told to restore a fluent completion, which amounts to rebuilding the malicious prompt from its fragments.
3. **Distributional failures** — *Perplexity Filtering, Sequential Monitors.* Detectors need the attack to live in an identifiable region of a score distribution (document-level or turn-level). Fluent semantic text sits in the same region as benign text — or the opposite one entirely.

---

## Four Case Studies of Assumption Failure

1. **Perturbation that weakens alignment.** Under SmoothLLM's default config, Phi-4's ASR rises 24→38% — a statistically significant (+14, *p*≈0.03) *degradation caused by the defense*. Character noise doesn't touch the semantic payload but roughs up the surface text the model's refusal was tuned on, tipping borderline requests past the refusal threshold.
2. **Depth-invariant erasure.** Erase-and-Check's ASR does not respond to erasure depth (13.16% at m = 0.1/0.3/0.5). Soundness holds (93.84% of conversations have zero false positives) but completeness fails exactly where the theorem's precondition stops applying.
3. **The inverted detection boundary.** Perplexity filtering fires **0 of 3,000** times at five thresholds; every one of 346 successful jailbreaks walks through at a 100% bypass rate. No threshold adjustment can restore detection without discarding most benign traffic.
4. **The silent monitor.** Sequential Monitors were *built* for decomposition attacks and still lose. 87.89% of successful jailbreaks finish without an alarm; the alarm rate is anti-correlated with model vulnerability (Gemma, 21% jailbroken, has the highest alarm rate at 26.8%; DeepSeek, 75.7% jailbroken, is flagged less than half as often). The per-turn drift premise — the condition that makes CUSUM minimax-optimal — is absent.

---

## Deployment Guidance (RQ3)

- **Within a single attack surface, model choice dominates defense choice.** Models fall into a resistant cluster (Gemma 4, OLMo-3.1, Qwen 3.6, DeepSeek-r1) and a vulnerable cluster (Granite 4.1, Phi-4), and cluster membership does not change when the defense changes. A strongly aligned model with *no* defense beats a weakly aligned model with any of these wrappers in every single-turn condition tested.
- **That conclusion does not transfer across surfaces.** Under multi-turn decomposition, the ranking reshuffles: DeepSeek-r1 (0% single-turn) reaches **75.7%**, Qwen rises 0→34.9%, Gemma 0→21.0%. Only OLMo-3.1 resists both surfaces (5.0% multi-turn), and even it is not clean.

---

## Repository Structure

```
.
├── csv_results_defence/            # Per-defense result CSVs
│   ├── smooth_llm_results.csv
│   ├── erase_and_check_results.csv
│   ├── Self_denoised_results.csv
│   ├── semantic_smoothing_results.csv
│   ├── sequential_monitor_results.csv
│   └── perplexity_filtering_results_analyzed.csv
├── without_defence/                # Baseline (no defense)
│   ├── LEADERBOARD_SUMMARY.csv
│   ├── FULL_REPORT_WITH_RESPONSES.json
│   └── <model>_FULL_RESPONSES.json  # per-model raw responses
└── README.md
```

The paper's anonymous artifact (paper source + analysis code) is available at: **https://anonymous.4open.science/r/defence-testing-EDB4**

---

## Responsible Disclosure

Following standard practice in security measurement, the exact attack prompts are provided for documentation and reproducibility, but **harmful model completions are excluded** from this release to prevent misuse.

---

## License

This repository is dual-licensed to separate code from data:

- **Code** — [MIT License](LICENSE.md).
- **Data** (the result CSVs under `csv_results_defence/` and the JSON reports under `without_defence/`) — [Creative Commons Attribution 4.0 International (CC BY 4.0)](DATA_LICENSE.md).

---

## Citation

```bibtex
@article{anonymous2026breaking,
  title   = {Breaking the Assumptions: Auditing Input-Side Jailbreak Defenses Against Semantic Attacks},
  author  = {Anonymous},
  year    = {2026}
}
```

*Keywords: Large Language Models, AI Safety, Semantic Jailbreaks, Certified Robustness, Adversarial Robustness, Defense Auditing.*

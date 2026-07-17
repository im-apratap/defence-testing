# Mathematical Analysis of Jailbreak Defense Failures in Local Large Language Models: A Certified Robustness Perspective

---

## Overview

This repository contains the complete experimental infrastructure, raw data, analysis code, and paper source for our AAAI submission:

> **"Mathematical Analysis of Jailbreak Defense Failures in Local Large Language Models: A Certified Robustness Perspective"**

We do not merely show that jailbreak defenses fail — a finding already well-known in the empirical literature. Instead, for each of six mathematically grounded defenses, we provide **formal mathematical proofs** identifying precisely **which assumption** embedded in each defense's theoretical guarantee is violated by the class of semantic jailbreak prompts we employ. Every violation is confirmed via rigorous statistical hypothesis testing with Wilson confidence intervals and p-values.

---

## The Core Research Question

> *When a mathematically certified jailbreak defense fails, which specific mathematical assumption in its formal guarantee was violated, and why does that assumption fail for semantic jailbreak attacks?*

---

## Key Findings at a Glance

| Defense | Mathematical Claim | Observed ASR | Violation |
|---|---|---|---|
| **SmoothLLM** | $p \leq (1-q)^L = 7.41\times10^{-19}$ | **94.39%** | Ratio $8.30\times10^{16}\times$ |
| **Semantic Smoothing** | $\underline{p}_A > 0.5 \Rightarrow R_s > 0$ | **16.17%** | $\underline{p}_A^{\max} = 0.4782 < 0.5$ always; 0% certified |
| **Self-Denoised Smoothing** | $R_d = \sigma\Phi^{-1}(\underline{p}_A) - \tau > 0$ | **9.57%** | $R_d^{\max} = -0.3006 < 0$ always; 0% certified |
| **Sequential Monitors** | Detection rate $\geq 0.95$; FP $\leq e^{-h}$ | **42.10% JB rate** | 12.11% detection; Wald violated $1{,}744\times$ at $h=10$ |
| **Erase-and-Check** | $P(\text{detect}) = 1$ for suffix $\leq m \cdot n$ | **13.04%** | $\Delta\text{ASR}/\Delta m = 0$; 15.57% complete failure |
| **Perplexity Filtering** | $\text{PPL}(x_\text{attack}) \gg \mu + k\sigma$ | **11.39%** | 0/345 jailbreaks detected; $z = -2.61$ (wrong direction) |

**Grand total: 13,272 experimental conversations across 6 models and 6 defenses.**

---

## The Unified Root Cause

All six defenses share a single structural failure: they were designed and certified against **token-space attacks** (primarily GCG-style adversarial suffixes). Semantic jailbreaks — natural language role-play, persona injection, and multi-turn decomposition — violate every structural assumption these defenses embed:
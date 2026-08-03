# Knowledge-Distillation Evidence Audit — Check Result

*Updated 2026-08-02 for the general research wiki.*

An **evidence audit** checks whether published conclusions are supported by
measurements strong enough to separate a real difference from random change.

**Later outcome:** the Stage 1 study produced by this gate was **SUSPENDED on
2026-07-26**. See [[KD-Noise-Floor-Stage1]]. Its measurements and design were
sound, but the field already knew the reporting problem and had no incentive
to change. An audit of many papers also creates reviewer conflicts of interest.

Historical gate status: **Gated 2026-07-25. SURVIVES WITH A REQUIRED CHANGE OF
DIRECTION.** It was the first of eleven ideas to pass a gate. The check found a
stronger piece of evidence than the original idea and corrected three mistakes
in the brief.

## Corrections to the original brief

These errors cannot remain in a paper whose purpose is to audit evidence.

1. [DistiLLM-2](https://arxiv.org/abs/2503.07067) is an **ICML 2025 Oral (top 1%)**, not a Spotlight.
2. The AVG column mixes **two judges**. GPT-4o judges AlpacaEval and
   Evol-Instruct, while **GPT-4o-mini** judges UltraFeedback. Calling the whole
   column a "GPT-4o win rate" is wrong.
3. [DistiLLM-1](https://arxiv.org/abs/2402.03898)'s five seeds change **evaluation-time decoding**, not training.

Also, **do not claim position bias.** DistiLLM-2 already controls for it by
averaging results after switching the order of the two responses being
compared.

## The main comparison, checked directly

Table 2 is byte-for-byte the same in paper versions v1 and v2. For a
Qwen2-7B-Instruct teacher and Qwen2-1.5B student, the AVG scores are
**[GKD](https://arxiv.org/abs/2306.13649) 56.14, DistiLLM 56.35, and [DistiLLM-2](https://arxiv.org/abs/2503.07067) 58.69.** None of the 19 pages,
including the appendices, reports variation across runs. Yet both sides of the
evaluation are random: the student samples with temperature 0.8 and top-p 0.95,
and the judge samples with temperature 0.7. The result comes from one run with
no stated seed.

## The strongest evidence: the earlier paper already disproves the certainty

This needs zero new computing. **[DistiLLM-1](https://arxiv.org/abs/2402.03898), from the same research group,
published the needed evidence in 2024.**

It reports means and standard deviations over five decoding seeds for **GPT-4
Eval**, not only for ROUGE-L. We extracted all 156 judge-score standard
deviations from Tables 11–13:

> **minimum 0.02, median 0.46, mean 0.55, maximum 1.83 — and 75.6% are larger
> than the 0.21-point difference that [DistiLLM-2](https://arxiv.org/abs/2503.07067) uses to put [GKD](https://arxiv.org/abs/2306.13649) below
> DistiLLM.**

The median `0.46` is a **lower bound** on all relevant randomness. These five
runs change only the decoding seed. They hold the training seed, judge model,
judge sampling, and temperature fixed.

The same group measured a median judge standard deviation of 0.46 in 2024 and
then published a 0.21-point ordering in 2025. A general paper about judge
reliability cannot tell this exact story. The result costs zero GPU-hours to
establish.

## Why the original framing fails

Three of its four planned sources of variation are already covered:

- **CyclicJudge** ([`2603.01865`](https://arxiv.org/abs/2603.01865)) already separates benchmark-score variation into scenario,
  generation, judge, and leftover parts. That jointly covers three of the four
  planned factors.
- **The Coin Flip Judge?** ([`2606.13685`](https://arxiv.org/abs/2606.13685)) measures repeat-run reliability,
  temperature, prompt sensitivity, response-order bias, and sampling seed.
  Pairwise choices flip **13.6%** of the time on average. It takes 11 repeated
  trials to recover the answer given by a 50-trial reference.
- **Reliability without Validity** ([`2606.19544`](https://arxiv.org/abs/2606.19544)) covers changes between judge models: 21
  judges, about 541,000 judgments, and rankings that move by as many as 14
  places.

The overall type of study also appeared at LLM scale two months earlier.
[`2605.20798`](https://arxiv.org/abs/2605.20798) audits 20 transformer changes with equal compute, several seeds, a
noise floor, and a Bonferroni correction for many tests. It finds that **only 2
of 20 changes still show a reliable gain at 1.2B parameters**. That has the
same shape as this paper, but for model architecture instead of distillation.

## The required new direction

1. **Lead with the training seed, not judge noise.** It is the only factor that
   changes the actual trained model rather than only how that model is measured.
   **Zero cs.CL papers on arXiv jointly match "training seed" and "judge".**
2. **Lead with the earlier-paper contradiction above.** It is unique, free to
   measure, and unavailable to a general judge-reliability paper.
3. **Report rankings that reverse, not only percentages below a noise floor.**
   "These K published method orderings reverse when we resample" is a concrete
   result. "N% of score gaps are below the floor" is weaker.
4. **Keep the scope honest.** One teacher-student pair, about 4 methods, and 5
   training seeds means about 20 runs on 4x A100 GPUs. Estimate the part of
   variation caused by the training seed; do not promise every combination of
   every factor.

## What is still open in KD reproducibility

- **No study found has audited reproducibility, controlled comparisons, or
  score variation for modern LLM knowledge-distillation methods.**
- **EasyOPD ([`2607.11012`](https://arxiv.org/abs/2607.11012)) is only a library.** It provides modular settings,
  runnable YAML files, and three on-policy setups. It provides **no audit,
  seeds, variance report, or comparison between published and reproduced
  results.** It does not take this idea, though it competes with any claim about
  infrastructure.
- Four surveys ([`2402.13116`](https://arxiv.org/abs/2402.13116), [`2407.01885`](https://arxiv.org/abs/2407.01885), [`2503.12067`](https://arxiv.org/abs/2503.12067) TMLR 102 pages, and
  [`2504.14772`](https://arxiv.org/abs/2504.14772)) run no controlled comparison with variance and do not measure whether
  results are comparable.

## Main practical problem: the expensive part is the new part

**DistiLLM-2 releases no official student checkpoints.** Available third-party
copies reproduce the DistiLLM-1 GPT-2/Llama-2 Dolly setup, not the DistiLLM-2
Qwen2 setup. Every training-seed cell therefore needs a full training run using
2–4 A100 GPUs.

This creates an uncomfortable tradeoff: **the only new source of variation is
expensive to measure, while the three cheap ones are already published.** That
is why the plan below has stages.

## Table concern — be careful and do not publish it first

In Table 2, [GKD](https://arxiv.org/abs/2306.13649) scores **57.74** on UltraFeedback in both the Qwen2 and Mistral
groups. DistiLLM scores **58.18** in both groups. These are two exact two-decimal
matches across different teacher-student pairs. The AVG cells are consistent
with those values, so this is not simply an averaging typo. Separately, the
DistiLLM Qwen2 AVG recalculates to `56.36`, while the table prints `56.35`.

**This is weak evidence by itself.** Win rates are bounded and often close
together. When many cells are compared, some exact matches can happen by
chance. The pattern is worth checking, but it proves nothing.

Before mentioning this anywhere, independently recompute the values from the
PDF, check whether v1 and the final camera-ready paper differ, and **contact the
authors privately first.** Suggesting an integrity problem in a named ICML Oral
can seriously harm real people. It must not become a headline or a persuasive
trick. The paper must discuss a **field-wide** issue, using DistiLLM only as one
case study. It must never attack one group.

## Staged plan

### Stage 1 — no computing, several days

Extract DistiLLM-1's 156 published standard deviations. Describe their
distribution. Compare score differences published across the KD literature
with that distribution, and report which method orderings cannot be separated
from the field's own measured randomness. Any reader can verify this from the
published PDFs. This can stand alone as a short or workshop paper and has no
risk of a failed reproduction.

### Stage 2 — only if Stage 1 is useful, about 20 runs on 4x A100

Measure the training-seed part for one teacher-student pair and four methods.
Report which rankings reverse.

### Stage 3 — only if Stages 1 and 2 work

Release the reproduction and variance-checking process as a usable tool. This
competes with EasyOPD by adding the audit that EasyOPD lacks.

## Work still needed

1. **OpenReview reviews are not checked.** Forum [`rc65N9xIrY`](https://openreview.net/forum?id=rc65N9xIrY) is behind a
   Cloudflare challenge, and ICML does not publicly release reviews. We do not
   know whether reviewers asked for error bars, and that matters.
2. The claim that nobody else is doing this—gate 6 in the numbered gate list in
   the svib repo wiki—comes only from arXiv API searches because the WebSearch
   budget ran out. This negative result is weaker than the others.
3. Independently verify the table concern before using it at all.

## Related

[[LLM-KD-Direction-Gates]] — the survey that led to this idea and the lesson
about how to find research questions.
[[KD-Noise-Floor-Stage1]] — the final form of Stage 1, expanded to two research
areas and later suspended.

# KD Evidence Audit — Gate Result

*Updated 2026-08-02 for the general research wiki.*

Status: **Gated 2026-07-25. SURVIVES WITH A REQUIRED PIVOT.** First candidate in
eleven to clear a gate. The gate also found a stronger asset than the one
proposed, and corrected three factual errors in the brief.

## Corrections to my brief — an audit paper cannot carry these

1. DistiLLM-2 is an **ICML 2025 Oral** (top 1%), not a Spotlight.
2. The AVG column mixes **two judges** — GPT-4o for AlpacaEval and
   Evol-Instruct, **GPT-4o-mini** for UltraFeedback. "GPT-4o win rate" is wrong.
3. DistiLLM-1's five seeds are **eval-time decoding seeds, not training seeds.**

Also: **do not claim position bias.** DistiLLM-2 already controls it — "we
average the results by switching the order of the compared responses."

## The anchor, verified

Table 2 confirmed byte-identical across v1 and v2. Qwen2-7B-Inst to Qwen2-1.5B,
AVG: **GKD 56.14, DistiLLM 56.35, DistiLLM-2 58.69.** Zero variance reporting
anywhere in 19 pages including appendices. Evaluation is stochastic on both
sides — student sampling at temperature 0.8 / top-p 0.95, judge at temperature
0.7 — from a single unseeded run.

## The real asset: the self-refutation, at zero compute

**DistiLLM-1, from the same author group, published the refutation in 2024.**

It reports mean and standard deviation over five decoding seeds for **GPT-4
Eval**, not merely ROUGE-L. All 156 judge-eval standard deviations extracted
from its Tables 11-13:

> **min 0.02, median 0.46, mean 0.55, max 1.83 — and 75.6% exceed the 0.21-point
> gap DistiLLM-2 uses to order GKD above DistiLLM.**

And `0.46` is a **lower bound**: it varies only the decoding seed, holding
training seed, judge model, judge sampling, and temperature fixed.

The same group measured a 0.46 median judge standard deviation in 2024 and
shipped a 0.21-point ordering in 2025. **No generic judge-reliability paper can
tell that story, and it costs zero GPU-hours to establish.**

## Why the proposed framing fails, and what the pivot is

Three of the four proposed axes are occupied:

- **CyclicJudge** (`2603.01865`) already gives "a variance decomposition that
  partitions benchmark score variance into scenario, generation, judge, and
  residual components" — three of the four axes, jointly.
- **The Coin Flip Judge?** (`2606.13685`) measures run-to-run reliability,
  temperature, prompt sensitivity, position bias and sampling seed: pairwise
  preferences flip **13.6%** on average, and 11 repeated trials are needed to
  recover a 50-trial reference verdict.
- **Reliability without Validity** (`2606.19544`) covers the judge-model axis —
  21 judges, ~541k judgments, rankings shifting up to 14 positions.

And the *genre* was run two months ago at LLM scale: `2605.20798` audits 20
transformer modifications under iso-compute with a multi-seed noise floor and
Bonferroni correction, finding **only 2 of 20 survive at 1.2B**. That is this
paper's exact shape, for architecture rather than distillation.

**Required pivot, per the gate:**

1. **Lead with training seed**, not judge noise. It is the only factor that
   changes the *artifact being judged* rather than the measurement of it, and
   **zero arXiv papers in cs.CL match "training seed" AND "judge" jointly.**
2. **Hook on the self-refutation** above — unique, free, and unavailable to any
   generic judge-reliability paper.
3. **Report rank inversions, not fractions.** "N% of deltas fall below the
   floor" is weak; "these K published orderings invert under resampling" is a
   result.
4. **Scope honestly:** one teacher-student pair, ~4 methods, 5 training seeds
   is ~20 runs on 4x A100. State the training-seed variance component; do not
   promise a full factorial.

## Genuine opening: KD-specific reproducibility is empty

- **No reproducibility study, controlled comparison, or variance audit of
  LLM-era KD methods exists.**
- **EasyOPD (`2607.11012`) is a library only** — modular config, runnable YAMLs,
  three on-policy settings, and **no audit, no seeds, no variance, no
  published-versus-reproduced comparison.** It does not scoop this, but it
  competes for the infrastructure framing.
- Four surveys (`2402.13116`, `2407.01885`, `2503.12067` TMLR 102pp,
  `2504.14772`): **none runs its own controlled comparison with variance, none
  measures comparability.**

## Feasibility — the weak point

**No official student checkpoints are released** for DistiLLM-2. Only
third-party reproductions exist, and they target the DistiLLM-1 GPT-2/Llama-2
Dolly setup, not the Qwen2 setup. Every training-seed cell therefore requires
full retraining at 2-4x A100 per run.

The uncomfortable tension: **the one novel axis is the expensive one; the three
cheap axes are already published.** This is what forces the staged plan below.

## Table-integrity observation — HANDLE WITH CARE, DO NOT PUBLISH FIRST

In Table 2, GKD is **57.74** on UltraFeedback in *both* the Qwen2 and Mistral
groups, and DistiLLM is **58.18** in both. Two exact two-decimal coincidences
across different teacher-student pairs. AVG cells are internally consistent with
those values, so it is not an averaging typo. Separately, DistiLLM's Qwen2 AVG
recomputes to `56.36` against a printed `56.35`.

**This is weak evidence on its own.** Win rates are bounded and clustered, and
across many compared cells the chance of some coincidence is not small. It is
suspicious, not conclusive.

**Required handling before this appears anywhere:** independently recompute from
the PDF; check whether v1 and camera-ready differ; and **contact the authors
privately first.** An integrity allegation against a named ICML Oral is a
serious act with real consequences for real people, and it must not be a
headline or a rhetorical device. The defensible framing throughout is
**literature-wide, with DistiLLM as a case study** — never an attack on one
group.

## Staged plan

**Stage 1 — zero compute, days.** Extract DistiLLM-1's 156 published standard
deviations, characterize the distribution, overlay published deltas from the KD
literature, and report which orderings are unresolvable at the field's own
measured noise. Fully verifiable by anyone from published PDFs. This alone is a
short or workshop paper, and it carries no reproduction risk.

**Stage 2 — conditional on Stage 1, ~20 runs on 4x A100.** Add the training-seed
component for one teacher-student pair and four methods. Report rank inversions.

**Stage 3 — only if 1 and 2 land.** The reproduction-and-variance protocol as a
shipped artifact, competing with EasyOPD on the axis it lacks.

## Open items

1. **OpenReview reviews unverified** — forum `rc65N9xIrY` sits behind a
   Cloudflare challenge and ICML does not publicly release reviews. Whether
   reviewers raised the missing error bars is unknown and material.
2. The "nobody is doing this" finding (gate 6 of the numbered gate sequence
   kept in the svib repo wiki) rests on arXiv API queries only;
   the WebSearch budget was exhausted. Weaker than the other negatives.
3. Verify the table-integrity observation independently before any use.

## Related

[[LLM-KD-Direction-Gates]] — the survey this came from, and the
generating-process lesson that produced it.
[[KD-Noise-Floor-Stage1]] — what Stage 1 of the staged plan above became
(broadened to two literatures, then suspended).

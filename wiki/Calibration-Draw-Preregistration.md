# Calibration-Draw Noise Floor in LLM Compression — Pre-Registration

Status: **GATE FAILED 2026-07-26. SCOOPED. Do not run.** The experiment was
published in ACL 2024 and independently replicated twice. Recorded in full
because the failure was mine and the mode is instructive.

## Gate result — and my error

**Williams & Aletras, *On the Impact of Calibration Data in Post-training
Quantization and Pruning* ([`2311.09755`](https://arxiv.org/abs/2311.09755), ACL 2024) ran exactly this experiment**,
down to the token count. Verbatim:

> *"To examine the variability introduced by random sampling, we repeat the
> sampling process to create ten non-overlapping calibration sets for each
> source dataset. This provides a total of 50 distinct calibration sets."*

128 examples x 2,048 tokens = 262,144 tokens per set — the identical figure this
protocol proposed. Five sources x ten draws, sampled without replacement.
Methods: [GPTQ](https://arxiv.org/abs/2210.17323), [SpQR](https://arxiv.org/abs/2306.03078), [SparseGPT](https://arxiv.org/abs/2301.00774), [Wanda](https://arxiv.org/abs/2306.11695). Models: LLaMA/Vicuna 7-33B, OPT 6.7-30B.

**My error was severe and self-inflicted.** This protocol cited Williams &
Aletras *as prior work that varies calibration source or configuration* — a
design factor — and built its entire wedge on the claim that "nobody varies
draws from the same source." That is exactly what they varied. I asserted the
distinction without reading the paper, and the wedge was false from the first
draft.

**Independently replicated twice.** Ji et al., *Beware of Calibration Data for
Pruning Large Language Models* ([`2410.17711`](https://arxiv.org/abs/2410.17711), ICLR 2025, Soochow/Huawei):
*"all our experiments repeat the calibration data sampling 20 times with
different random seeds."* And Williams, Chrysostomou & Aletras ([`2410.17170`](https://arxiv.org/abs/2410.17170),
NAACL 2025) repeat the protocol on Gemma 2B, Phi-2, Mistral 7B, Llama 3.1 8B.

## The measured answer, so nobody re-derives it

Standard deviation across ten draws at fixed source and size, mean zero-shot
accuracy:

| Method | std (pp) | best-worst range |
|---|---:|---:|
| [SpQR](https://arxiv.org/abs/2306.03078) | 0.1-0.2 | 0.6-1.0% |
| [GPTQ](https://arxiv.org/abs/2210.17323) | 0.2-0.4 | 0.9-1.6% |
| [Wanda](https://arxiv.org/abs/2306.11695) | 0.1-0.4 | 0.6-2.9% |
| [SparseGPT](https://arxiv.org/abs/2301.00774) | 0.1-0.6 | 2.4-4.8% |

WikiText PPL std: SpQR `0.00-0.02`, SparseGPT `0.04-0.31`, Wanda `0.01-0.41`.
Single-task extremes are far larger than the aggregate: LLaMA-7B + SparseGPT
across ten C4 draws moves **RTE 52.7 to 61.7 (9.0 pp)** and **[BoolQ](https://arxiv.org/abs/1905.10044) 66.4 to
73.0 (6.6 pp)**.

Ji et al. add the dose-response: draw-noise shrinks as calibration size grows,
and sensitivity rises sharply with compression — under 0.1% at low sparsity,
0.5% at 50%, **2.3% at 60%**.

## What genuinely remains, and it is thin

1. **[AWQ](https://arxiv.org/abs/2306.00978) is uncovered by construction.** Zero AWQ mentions in Williams &
   Aletras, and `mit-han-lab/llm-awq` hardcodes `dataset.shuffle(seed=42)` with
   no seed flag — so **every published AWQ model used the same 512 pileval
   samples.** Unstudied because it is un-varyable without patching the library.
2. **Aggressive bit-widths.** Ji et al.'s amplification curve is established for
   sparsity, not for 2-3 bit, NVFP4 or MXFP4.
3. **Generative and reasoning metrics.** All three papers use multiple-choice
   and perplexity. Draw variance on [GSM8K](https://arxiv.org/abs/2110.14168), [MMLU-CoT](https://arxiv.org/abs/2009.03300), or long-form generation is
   unmeasured — and matters more now than it did in 2023.
4. **Hessian concentration with effective sample size for correlated tokens** —
   genuinely open. The strongest existing theory ([`2508.04853`](https://arxiv.org/abs/2508.04853)) bounds OPTQ
   error *conditionally on the realized* X and never bounds its fluctuation
   across draws. Note [GPTQ](https://arxiv.org/abs/2210.17323)'s mandatory damping (`percdamp=0.01`) is indirect
   evidence the empirical Hessian is not tightly concentrated at this size.
5. **The calibration monoculture.** `llm-compressor`'s canonical example is
   `load_dataset(ID, split="train_sft[:512]")` followed by `.shuffle(seed=42)`
   — the slice precedes the shuffle, so set membership is the deterministic
   first 512 rows and the shuffle only permutes order. Combined with AWQ's
   hardcoded seed, **two of the most widely deployed quantization paths
   calibrate on a single fixed set.** Whether that is deliberate (reproducibility)
   or incidental, nobody has documented it or measured its cost.

Items 1-3 are cell-filling on someone else's framing with effect sizes
predictable in advance from Ji et al. Item 5 is a software-archaeology
observation, verifiable by reading source, and possibly worth a short note
independent of any paper. Item 4 is real theory and out of scope for this group
without a collaborator.

## Second gate: confirms the kill, adds three more errors of mine

**Williams & Aletras is larger than reported above:** *"We vary **only** the
calibration data... This provides a total of **1,800 compressed models**...
**19,800 model evaluations**."* They also pre-empt the intended secondary
finding — *"a seemingly robust perplexity of 12.72±0.18. In contrast, the same
models achieve 66.7%±4.7 on [BoolQ](https://arxiv.org/abs/1905.10044), with accuracy ranging from 57.0% to 71.6%"* —
and the reproducibility recommendation to release calibration data.

**The appendix risk fired exactly as predicted, in two of the papers I named.**

- **[SparseGPT](https://arxiv.org/abs/2301.00774)**, verbatim: *"We repeat a standard 50% pruning run 5 times with
  different random seeds for data sampling and get **13.52 ± 0.075**...
  SparseGPT is quite robust to the precise calibration data being used."*
- **[Wanda](https://arxiv.org/abs/2306.11695)** App. D.2, Table 18: mean±std over 5 seeds, 2 methods x 4 models x 3
  sparsities.
- [OBC](https://arxiv.org/abs/2208.11580) and *Is C4 Optimal* likewise. [AQLM](https://arxiv.org/abs/2401.06118) reports SD `0.127` at 128 sequences
  falling to `0.005` at 4096.

**My premise was also factually wrong about the setup.** [QuIP#](https://arxiv.org/abs/2402.04396) and [QTIP](https://arxiv.org/abs/2406.11235) do *not*
use 128 C4 sequences — *"Hessian matrices H were generated with 6144
sequences"* of RedPajama. [AWQ](https://arxiv.org/abs/2306.00978) uses Pile-val at sequence length 512 and claims 16
sequences suffice. **"Everyone uses ~128 random C4" is wrong for three of the
five methods I named**, and AQLM's numbers predict draw noise is negligible at
the sample counts modern methods actually use. The expected result was null.

**The older-vocabulary trap fired again, and I had been warned about it.**
Bouthillier et al. (MLSys 2021, [`2103.03098`](https://arxiv.org/abs/2103.03098)) own the statistic: **P(A > B)**,
*"the probability of measuring a better performance for A than B across
fluctuations,"* with threshold `gamma = 0.75`. My "inversion probability" is
`1 - P(A > B)`, reinvented.

**The primary endpoint is computable from a 2023 appendix with arithmetic.**
From Wanda Table 18, SparseGPT versus Wanda: **four of twelve published cells
are coin flips** — LLaMA-7B 50% and LLaMA-13B 4:8 and LLaMA-2-13B 2:4 all at
`P(inversion) = 50%`, LLaMA-7B 4:8 at 40%. No GPUs required.

## What transfers to the surviving study

Two items are worth carrying into [[KD-Noise-Floor-Stage1]] as a **second case
study, zero compute, published numbers only**:

1. **[QTIP](https://arxiv.org/abs/2406.11235)'s NeurIPS checklist, verbatim:** *"Answer: No. Justification: **It is
   standard practice in LLM quantization papers to not report error bars on
   metrics.**"* A second literature stating the norm openly, in a mandated
   disclosure form. That is cross-domain evidence for the KD paper's thesis —
   the field treats empirical margins as self-evident — and it is quotable
   rather than inferred.
2. **The four coin-flip cells above**, computed from published mean±std with
   arithmetic, using Bouthillier's `P(A > B)` and citing it as theirs.

Clean negatives worth recording: **[GPTQ](https://arxiv.org/abs/2210.17323)** has zero occurrences of "seed" and no
`±` anywhere; **[AWQ](https://arxiv.org/abs/2306.00978)**'s robustness experiment swaps two different-domain Pile
subsets at one run per cell and is always worded as insensitivity to
*distribution*, with the first author stating on GitHub *"we have not
extensively ablated the use of calibration sets"*; **[OmniQuant](https://arxiv.org/abs/2308.13137)**'s Table A10 is
labelled "Varience" but is between-source variance at `n=1` per source, unable
to separate source effect from draw noise; **[SmoothQuant](https://arxiv.org/abs/2211.10438)** calibrates *"once
with 512 random sentences"*; **[QuaRot](https://arxiv.org/abs/2404.00456)** has no seeds and no `±`.

## Process finding — twelve gates, and two self-inflicted

This is the twelfth consecutive candidate killed at the gate. **Two of the
twelve — this one and the temperature confound — died to a paper I had already
cited in my own protocol and mischaracterized.** That is not a search-coverage
problem. It is a reading problem: I summarized prior work from its framing
rather than from its method section, and in both cases the method section
contained the experiment I was proposing.

**Rule going forward: before any protocol claims a gap, the two or three nearest
prior works must be read in their method sections, not their abstracts, and the
specific sentence establishing the gap must be quoted.** A gap asserted without
a quote is not a gap.

---

## Original protocol, retained for the record. Superseded.

Six open items blocked locking, including the prior-art gate. Written
2026-07-26 following [[Compression-Audit-Direction]].

## The question

> Post-training compression methods are calibrated on a small sample — typically
> **128 random C4 sequences**. Almost no paper reports *which* 128, and none
> resamples. **Do published method rankings survive redrawing that sample?**

## The wedge — state this precisely or a reviewer will conflate it

Prior work varies the calibration **source or configuration**, which is a
**design factor**:

- Williams & Aletras (ACL 2024, [`2311.09755`](https://arxiv.org/abs/2311.09755)) — first extensive calibration-data
  study; *"substantial variations in downstream task performance."*
- *Is C4 Dataset Optimal for Pruning?* ([`2410.07461`](https://arxiv.org/abs/2410.07461)) — no; arithmetic data does
  as well or better.
- *Rethinking Layer Redundancy* (ACL ARR 2026) — *"the calibration
  configuration plays a substantially larger role than the choice of search
  algorithm."*

**Nobody varies draws from the same source**, which is a **noise source**. The
standard defence is that PTQ is deterministic given a calibration set — true,
and irrelevant, because the set is itself a random draw that is never reported
and never resampled.

This distinction must appear in the abstract. It is the entire contribution.

## Design

### Manipulated variable

`K = 10` independent calibration draws, each 128 sequences x 2048 tokens, from
the **same** source (C4 `en`, declared snapshot), under declared seeds
`{1001..1010}`. **The exact document IDs of all 1,280 sequences per draw are
released** — the point of the paper is that nobody does this.

### Methods

Chosen to be one-shot, inference-only, calibration-dependent, and actually
cited.

| Method | Type | Calibration role |
|---|---|---|
| [GPTQ](https://arxiv.org/abs/2210.17323) | quantization | Hessian estimation |
| [AWQ](https://arxiv.org/abs/2306.00978) | quantization | activation-scale search |
| [SparseGPT](https://arxiv.org/abs/2301.00774) | pruning | Hessian estimation |
| [Wanda](https://arxiv.org/abs/2306.11695) | pruning | activation-norm statistics |
| A VQ-class method ([QTIP](https://arxiv.org/abs/2406.11235) or [QuIP#](https://arxiv.org/abs/2402.04396)) | quantization | Hessian + codebook |
| **RTN** | quantization | **none — calibration-free control** |

**RTN is load-bearing.** It uses no calibration, so its across-draw variance
must be **zero**. Any nonzero spread on the RTN arm reveals a second noise
source (kernel nondeterminism, eval harness) and invalidates the measurement
until explained. It is the pipeline's null.

### Second null: same-draw replication

Each method is run **twice on the identical calibration draw**. Any nonzero
difference is implementation nondeterminism, not calibration noise, and is
subtracted before any claim. Pre-declared: if same-draw spread exceeds 25% of
across-draw spread, the study reports that instead and stops.

### Fixed factors

- **Models.** Stage A: one family (Llama-3.1-8B). Stage B: replicate on a second
  (Qwen2.5-7B) only if Stage A resolves.
- **Compression levels.** 4-bit, 3-bit, 2-bit for quantization; 50% unstructured
  and 2:4 semi-structured for pruning. H3 predicts the effect grows as
  compression becomes more aggressive.
- **Evaluation.** [WikiText-2](https://arxiv.org/abs/1609.07843) perplexity (the field's currency) plus the standard
  lm-eval zero-shot six ([ARC-easy](https://arxiv.org/abs/1803.05457), ARC-challenge, [HellaSwag](https://arxiv.org/abs/1905.07830), [PIQA](https://arxiv.org/abs/1911.11641), [WinoGrande](https://arxiv.org/abs/1907.10641),
  [BoolQ](https://arxiv.org/abs/1905.10044)). All are likelihood-scored and deterministic given the model, so **the
  calibration draw is the only randomness source in the pipeline.** That is what
  makes the measurement clean.

## Endpoints

**Primary.** For each ordered pair of methods `(m, n)` within a
`benchmark x compression-level` cell, the **inversion probability**

`P_inv(m, n) = Pr[ S(m, k) < S(n, k') ]` over independent draws `k, k'`,

estimated from the `K x K` cross-draw comparison matrix. An ordering claimed in
the literature is **unresolved** if `P_inv >= 0.10`.

**Secondary.** The spread ratio `R = sigma_between / sigma_within`, where
`sigma_within` is the pooled across-draw standard deviation per method and
`sigma_between` is the standard deviation of per-method draw-means.

**Pre-registered hypotheses.**

- **H1 (magnitude).** The median within-method across-draw standard deviation
  **exceeds the median published inter-method delta** in the harvested set
  (below). This is the claim that the field's currency is smaller than its
  unreported noise.
- **H2 (decisive).** For at least **25%** of published method pairs recoverable
  from the harvest, `P_inv >= 0.10`.
- **H3 (dose-response).** `R` decreases monotonically as bit-width falls from 4
  to 3 to 2 — rankings become less resolvable as compression gets more
  aggressive.

**Statistics.** Cross-draw comparison matrices for `P_inv`; bootstrap over draws
for `R` with 10,000 resamples, seed `20260726`. Effects reported in raw units
**and** in units of within-method across-draw standard deviation.

## Published-delta harvest — rules fixed before extraction

1. **Paper list declared before any extraction**: all papers in a declared venue
   and date range whose headline table compares two or more of the six methods
   above at a matched compression level.
2. **Table-selection rule declared**: headline/main-results table only, never an
   appendix chosen after seeing results.
3. **Report the full denominator**, including published deltas that comfortably
   clear the noise floor. Selective reporting here would be the same failure the
   paper diagnoses.
4. Where a paper reports its own variance, use that in preference to ours.
5. Record, as a descriptive quantity, **how many harvested papers report any
   variance at all** and **how many state which calibration sequences were
   used**.

## Validity gates before any result is accepted

- **Anchor reproduction:** each method reproduces its own published number for
  at least one declared configuration within a pre-declared tolerance, before
  any new claim. Harness validated against published values first.
- RTN across-draw spread is exactly zero.
- Same-draw replication spread below the 25% threshold above.
- One manifest recording checkpoint hashes, calibration document IDs, lm-eval
  version, kernel and library versions, and GPU model; the analyzer refuses
  mixed-manifest artifacts.

## Decision gate

**Full paper** if H2 holds — named published orderings invert under
resampling of a set the field draws at random and never reports.

**Reduced scope** if H1 holds but H2 fails: the noise is real and comparable to
published deltas, but no specific ordering inverts. That is a
reporting-standards contribution, weaker, and to be judged against the
prior-art gate at that time rather than assumed publishable.

**Stop, and report the negative** if across-draw spread is negligible. **This
outcome is genuinely plausible and must be stated as such in advance:** 128 x
2048 tokens is a substantial sample, and Hessian and activation statistics may
well be stable across draws. A clean negative — *"PTQ is robust to calibration
resampling, and here is the measurement"* — defends the field, closes the
question, and is worth a short paper. The study is designed so that this is a
result rather than a failure.

## What may not be claimed

- **Not** that calibration data matters — Williams & Aletras established that.
  Our claim is about draws within a fixed source.
- **Not** that perplexity is a poor proxy — [LLM-KICK](https://arxiv.org/abs/2310.01382), *Accuracy is Not All You
  Need*, *[The Benchmark Illusion](https://arxiv.org/abs/2606.17609)* and *Silent Failures* own that.
- **Not** that compression methods are useless. The finding, if it lands,
  concerns the resolvability of *rankings*, not the value of compression.
- **Not** an accusation of poor practice by any author. Reporting 128 random
  sequences without their identities is the field's universal convention, and
  the subject is the convention.

## Known hazards

1. **The prediction may be wrong**, per the stop condition above.
2. **Kernel nondeterminism** could masquerade as calibration noise — hence the
   same-draw null.
3. **Method implementations differ across repositories.** Use each method's
   official implementation and record the commit; do not reimplement.
4. **`K = 10` gives coarse `P_inv` resolution** (increments of 0.01 on the
   `K x K` matrix). Adequate for a 0.10 threshold; do not report finer.
5. **Anchor reproduction may fail** for some method, as it did in *Revisiting
   RaBitQ and TurboQuant*. If so, that is reported as a finding and the method
   is excluded from the ranking analysis, not silently dropped.

## Open items blocking lock

1. **Prior-art gate**, searched by task and older vocabulary rather than method
   name: whether anyone resamples calibration draws from a fixed source; the
   coreset and data-selection literature, where this may exist under different
   words; and forward citations of Williams & Aletras and *Is C4 Optimal*.
2. Confirm each method's official implementation, commit, and calibration
   interface.
3. Declare the harvest paper list and venue/date range.
4. Pre-declare the anchor-reproduction tolerance per method.
5. Pre-declare the practically meaningful `R`, so a statistically resolved but
   negligible effect is not reported as success.
6. Confirm VQ-class method availability and whether its calibration interface is
   comparable; drop it rather than force a mismatched comparison.

## Compute

Stage A: `6 methods x 10 draws x 5 compression levels x 2 (replication) = 600`
compression runs at 7-8B, each roughly 20-40 minutes including evaluation, so
**roughly 300-400 GPU-hours — about two days on eight GPUs.** Stage B doubles
it. Inference-only throughout: no training, no released-checkpoint dependency.

## Related

[[Compression-Audit-Direction]] — the survey and the convergence argument.
[[KD-Noise-Floor-Stage1]] — the companion study in a second literature.

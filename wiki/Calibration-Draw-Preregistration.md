# Random Calibration Samples in LLM Compression — Pre-Registered Plan

**Calibration data** is a small sample used to set up a model-compression
method after training. A **noise floor** is the amount a score changes only
because that random sample changes.

Status: **GATE FAILED 2026-07-26. ALREADY PUBLISHED. Do not run.** ACL 2024
published this experiment, and two other papers repeated it independently. This
page keeps the complete plan because the mistake was mine and teaches an
important lesson.

## Result of the earlier-work check, including my mistake

**Williams & Aletras, *On the Impact of Calibration Data in Post-training
Quantization and Pruning* ([`2311.09755`](https://arxiv.org/abs/2311.09755), ACL 2024), ran exactly this
experiment.** Even the token count matched. They wrote:

> *"To examine the variability introduced by random sampling, we repeat the
> sampling process to create ten non-overlapping calibration sets for each
> source dataset. This provides a total of 50 distinct calibration sets."*

Each set had 128 examples with 2,048 tokens each:
`128 x 2,048 = 262,144` tokens. That is the exact size proposed below. The paper
used five data sources and ten non-overlapping samples from each source. It
tested [GPTQ](https://arxiv.org/abs/2210.17323), [SpQR](https://arxiv.org/abs/2306.03078), [SparseGPT](https://arxiv.org/abs/2301.00774), and [Wanda](https://arxiv.org/abs/2306.11695) on LLaMA/Vicuna models from
7B to 33B parameters and OPT models from 6.7B to 30B.

**My mistake was serious and avoidable.** The old plan cited Williams & Aletras
as work that changes the calibration source or setup—a planned design choice.
Then it based the whole exact unanswered question on the claim that "nobody
redraws samples from the same source." But that paper did exactly that. I made
the distinction without reading the paper. The claimed gap was false from the
first draft.

**Two papers repeated it independently.** Ji et al., *Beware of Calibration
Data for Pruning Large Language Models* ([`2410.17711`](https://arxiv.org/abs/2410.17711), ICLR 2025,
Soochow/Huawei), says that every experiment repeats calibration sampling 20
times with different random seeds. Williams, Chrysostomou & Aletras
([`2410.17170`](https://arxiv.org/abs/2410.17170), NAACL 2025) repeat the setup with Gemma 2B, Phi-2, Mistral 7B,
and Llama 3.1 8B.

## The published answer

The table shows the standard deviation across ten samples when the source and
sample size stay fixed. Scores are mean zero-shot accuracy. `pp` means
percentage points.

| Method | standard deviation (pp) | best-to-worst range |
|---|---:|---:|
| [SpQR](https://arxiv.org/abs/2306.03078) | 0.1–0.2 | 0.6–1.0% |
| [GPTQ](https://arxiv.org/abs/2210.17323) | 0.2–0.4 | 0.9–1.6% |
| [Wanda](https://arxiv.org/abs/2306.11695) | 0.1–0.4 | 0.6–2.9% |
| [SparseGPT](https://arxiv.org/abs/2301.00774) | 0.1–0.6 | 2.4–4.8% |

For WikiText perplexity (PPL), the standard deviations are SpQR `0.00-0.02`,
SparseGPT `0.04-0.31`, and Wanda `0.01-0.41`. Individual tasks can move much
more than the average. For LLaMA-7B with SparseGPT across ten C4 samples, **RTE
moves from 52.7 to 61.7, a 9.0-point range**, and [BoolQ](https://arxiv.org/abs/1905.10044) moves from 66.4 to
73.0, a **6.6-point range**.

Ji et al. also show how the effect changes with sample size and compression
strength. Random-sample noise shrinks as the calibration sample grows. It rises
quickly as compression becomes harsher: below 0.1% at low sparsity, 0.5% at 50%
sparsity, and **2.3% at 60%**.

## What is still unanswered, but only narrowly

1. **[AWQ](https://arxiv.org/abs/2306.00978) was not covered by this design.** Williams & Aletras never mention
   AWQ. The `mit-han-lab/llm-awq` code fixes `dataset.shuffle(seed=42)` and has
   no seed option. Therefore, **every published AWQ model used the same 512
   pileval samples.** Researchers did not vary AWQ's calibration sample because
   doing so requires changing the library.
2. **Very low bit-widths.** Ji et al. establish that stronger pruning makes
   sample noise larger. They do not test 2–3 bit weights, NVFP4, or MXFP4.
3. **Generation and reasoning scores.** All three papers use multiple-choice
   accuracy and perplexity. They do not measure sample-to-sample changes on
   [GSM8K](https://arxiv.org/abs/2110.14168), [MMLU-CoT](https://arxiv.org/abs/2009.03300), or long-form generation. These tasks matter more now
   than they did in 2023.
4. **Theory for how a Hessian settles as samples grow, including the effective
   sample size of correlated tokens.** A Hessian is a matrix that describes
   curvature and helps some compression methods estimate which changes are
   safe. This question is truly open. The strongest theory found
   ([`2508.04853`](https://arxiv.org/abs/2508.04853)) bounds OPTQ error only after the sampled matrix `X` is fixed;
   it never bounds how that result changes across samples. [GPTQ](https://arxiv.org/abs/2210.17323)'s required
   damping (`percdamp=0.01`) is indirect evidence that the measured Hessian is
   not tightly stable at this sample size.
5. **Most users reuse the same calibration sample.** The standard
   `llm-compressor` example calls `load_dataset(ID, split="train_sft[:512]")`
   and then `.shuffle(seed=42)`. The slice happens before the shuffle, so the
   selected set is always the first 512 rows. Shuffling only changes their
   order. Together with AWQ's fixed seed, **two widely used quantization tools
   calibrate on one fixed set.** Nobody has recorded whether this was a
   deliberate choice for repeatability or an accident, or measured its cost.

Items 1–3 mostly fill extra cells in someone else's study, and Ji et al. make
their likely effect sizes predictable. Item 5 is a source-code-history finding
that can be checked by reading code and may support a short note without a full
paper. Item 4 is real theory, but this group would need a theory collaborator.

## A second check confirmed the decision and found more errors

Williams & Aletras ran a larger study than summarized above. They changed
**only** calibration data and produced **1,800 compressed models** and **19,800
model evaluations**. They also already made the planned secondary point. A
perplexity of `12.72±0.18` looked stable while the same models' [BoolQ](https://arxiv.org/abs/1905.10044) accuracy
was `66.7%±4.7`, ranging from 57.0% to 71.6%. They also recommended releasing
the calibration data so others can reproduce results.

The exact appendix problem that the plan warned about had already happened in
two papers it named:

- **[SparseGPT](https://arxiv.org/abs/2301.00774)** reports five normal 50%-pruning runs with different random
  data seeds: **`13.52 ± 0.075`**. It concludes that SparseGPT is robust to the
  exact calibration sample.
- **[Wanda](https://arxiv.org/abs/2306.11695)** Appendix D.2, Table 18 reports mean ± standard deviation across 5
  seeds for 2 methods, 4 models, and 3 sparsity levels.
- [OBC](https://arxiv.org/abs/2208.11580) and *Is C4 Optimal* also test this. [AQLM](https://arxiv.org/abs/2401.06118) reports a standard
  deviation of `0.127` at 128 sequences, falling to `0.005` at 4,096.

The old plan also described the common setup incorrectly. [QuIP#](https://arxiv.org/abs/2402.04396) and [QTIP](https://arxiv.org/abs/2406.11235) do
**not** use 128 C4 sequences. They build Hessian matrices from 6,144 RedPajama
sequences. [AWQ](https://arxiv.org/abs/2306.00978) uses Pile-val with sequence length 512 and says 16 sequences
are enough. **The statement "everyone uses about 128 random C4 sequences" was
wrong for three of the five named methods.** AQLM's numbers also predict that
random-sample noise is tiny at the large sample counts used by modern methods.
The likely result was no meaningful effect.

The draft also invented an older statistic under a new name. Bouthillier et al.
(MLSys 2021, [`2103.03098`](https://arxiv.org/abs/2103.03098)) define **`P(A > B)`**, the chance that method A
measures better than method B across random changes, using threshold
`gamma = 0.75`. The draft's "inversion probability" was just
`1 - P(A > B)`.

The proposed main result can be calculated from a 2023 appendix. From Wanda
Table 18, four of twelve published SparseGPT-versus-Wanda cells are effectively
coin flips. LLaMA-7B 50%, LLaMA-13B 4:8, and LLaMA-2-13B 2:4 each have
`P(inversion) = 50%`; LLaMA-7B 4:8 has 40%. No GPUs are needed.

## Evidence that can move to the surviving KD study

Two parts can become a **second case study with no new compute, using only
published numbers** in [[KD-Noise-Floor-Stage1]]:

1. [QTIP](https://arxiv.org/abs/2406.11235)'s NeurIPS checklist says: *"It is standard practice in LLM
   quantization papers to not report error bars on metrics."* This is a second
   research area openly stating the same reporting norm in a required form. It
   supports the KD paper's claim that the field treats small measured score
   differences as obvious. It is a direct statement, not something inferred
   from silence.
2. Use the four coin-flip cells above, calculated from published means and
   standard deviations. Use Bouthillier's `P(A > B)` and clearly credit it.

Other clear negative results worth saving:

- [GPTQ](https://arxiv.org/abs/2210.17323) contains zero uses of "seed" and no `±` symbol.
- [AWQ](https://arxiv.org/abs/2306.00978)'s robustness test swaps two Pile subsets from different subject areas,
  with one run per cell. It always describes robustness to the **data
  distribution**, not to random samples. The first author says on GitHub that
  they have not extensively tested calibration sets.
- [OmniQuant](https://arxiv.org/abs/2308.13137) Table A10 is labeled "Varience," but it reports variation
  between data sources with `n=1` per source. It cannot separate a source effect
  from random-sample noise.
- [SmoothQuant](https://arxiv.org/abs/2211.10438) calibrates once with 512 random sentences.
- [QuaRot](https://arxiv.org/abs/2404.00456) gives no seeds and no `±` values.

## Process lesson: twelve failed ideas, including two avoidable failures

This was the twelfth idea in a row to fail at the earlier-work check. **Two of
the twelve—this one and the temperature issue—failed because of papers already
cited in their own plans and described incorrectly.** Search coverage was not
the problem. Reading was. I summarized how the papers described their topic
instead of reading the methods. In both cases, the method section already
contained the proposed experiment.

**Standing rule: before any plan claims an open gap, read the methods of the
two or three nearest papers and quote the exact sentence that proves the gap.
Without that quote, the gap is not established.**

---

## Original plan, kept only as a record — superseded

Written 2026-07-26 after [[Compression-Audit-Direction]]. Six items, including
the earlier-work check, still blocked the plan from being locked.

## Original question

> Post-training compression methods use a small calibration sample, often
> **128 random C4 sequences**. Almost no paper says *which* 128 it used, and
> none draws another sample. **Would published method rankings remain the same
> if the calibration sample changed?**

## Original exact unanswered question — this claim was false

The plan said that earlier work changed the calibration **source or setup**, a
planned design choice:

- Williams & Aletras (ACL 2024, [`2311.09755`](https://arxiv.org/abs/2311.09755)): first broad calibration-data
  study; "substantial variations in downstream task performance."
- *Is C4 Dataset Optimal for Pruning?* ([`2410.07461`](https://arxiv.org/abs/2410.07461)): no; arithmetic data
  works as well or better.
- *Rethinking Layer Redundancy* (ACL ARR 2026): "the calibration configuration
  plays a substantially larger role than the choice of search algorithm."

It then incorrectly said that **nobody changed random samples within one
source**, which would be a noise source. PTQ being deterministic after choosing
a sample did not answer this question because the sample itself was random,
unreported, and supposedly never repeated. The plan said this difference must
appear in the abstract because it was the entire new contribution.

## Original design

### What would change

Use `K = 10` independent calibration samples. Each contains 128 sequences of
2,048 tokens from the **same declared snapshot of C4 `en`**, with seeds
`{1001..1010}`. Release the exact document IDs for all 1,280 sampled sequences
across the ten samples. The old text said "all 1,280 sequences per draw," but
its own design gives 128 sequences per draw and 1,280 in total. The point was
that other papers did not release these IDs.

### Methods

The methods had to be one-shot, use inference only, depend on calibration data,
and have real citations.

| Method | Type | How it uses calibration data |
|---|---|---|
| [GPTQ](https://arxiv.org/abs/2210.17323) | quantization | estimates a Hessian |
| [AWQ](https://arxiv.org/abs/2306.00978) | quantization | searches activation scales |
| [SparseGPT](https://arxiv.org/abs/2301.00774) | pruning | estimates a Hessian |
| [Wanda](https://arxiv.org/abs/2306.11695) | pruning | measures activation norms |
| A vector-quantization method: [QTIP](https://arxiv.org/abs/2406.11235) or [QuIP#](https://arxiv.org/abs/2402.04396) | quantization | Hessian plus codebook |
| **RTN** | quantization | **none; calibration-free control** |

**RTN was essential.** It uses no calibration data, so its score must be
exactly the same across samples. Any spread would reveal another source of
randomness, such as non-deterministic GPU kernels or the evaluation program.
Until explained, that would make the measurement invalid. RTN was the pipeline's
null control.

### Second control: repeat the same sample

Run every method **twice with the exact same calibration sample**. Any
difference would come from the implementation, not the sample. That difference
must be removed before making a claim. The rule fixed in advance was: if the
same-sample spread exceeds 25% of the across-sample spread, report the
implementation randomness instead and stop.

### Things held fixed

- **Models:** Stage A uses one family, Llama-3.1-8B. Only if Stage A gives a
  clear answer does Stage B repeat on Qwen2.5-7B.
- **Compression:** 4-bit, 3-bit, and 2-bit quantization; 50% unstructured and
  2:4 semi-structured pruning. H3 predicts more random-sample noise when
  compression is harsher.
- **Evaluation:** [WikiText-2](https://arxiv.org/abs/1609.07843) perplexity, plus six standard zero-shot tasks
  from lm-eval: [ARC-easy](https://arxiv.org/abs/1803.05457), ARC-challenge, [HellaSwag](https://arxiv.org/abs/1905.07830), [PIQA](https://arxiv.org/abs/1911.11641), [WinoGrande](https://arxiv.org/abs/1907.10641),
  and [BoolQ](https://arxiv.org/abs/1905.10044). These use deterministic likelihood scores once a model is
  fixed. The calibration sample would therefore be the pipeline's only source
  of randomness.

## Original measurements

For every ordered method pair `(m, n)` in one
`benchmark x compression-level` cell, the main measurement was called
**inversion probability**:

`P_inv(m, n) = Pr[ S(m, k) < S(n, k') ]`

where `k, k'` are independent calibration samples. It would be estimated
from all `K x K` cross-sample comparisons. A published order would be called
**unresolved** if `P_inv >= 0.10`. This statistic was later found to be
`1 - P(A > B)` from Bouthillier et al., not a new measure.

The second measurement was `R = sigma_between / sigma_within`.
`sigma_within` was the pooled standard deviation across calibration samples for
each method. `sigma_between` was the standard deviation across each method's
average score.

### Original hypotheses

- **H1, size:** the median standard deviation within a method across samples is
  **larger than the median published difference between methods** in the paper
  set below. This would mean that typical published gains are smaller than
  unreported sample noise.
- **H2, decisive:** at least **25%** of recoverable published method pairs have
  `P_inv >= 0.10`.
- **H3, dose response:** `R` falls steadily when bit-width falls from 4 to 3 to
  2. Harsher compression makes rankings harder to separate.

The plan would use cross-sample comparison matrices for `P_inv` and bootstrap
over samples for `R` with 10,000 resamples and seed `20260726`. It would report
effects in raw units and as multiples of the within-method across-sample
standard deviation.

## Rules for collecting published differences

1. List papers before extracting any numbers: all papers in a fixed venue and
   date range whose main table compares at least two of the six named methods
   at the same compression level.
2. Use only the main-results table, never an appendix chosen after seeing the
   results.
3. Report every comparison, including differences far above the noise floor.
   Choosing only weak results would repeat the problem being criticized.
4. If a paper reports its own variance, use that instead of this study's.
5. Also count how many collected papers report any variance and how many name
   the exact calibration sequences.

## Checks required before accepting a result

- **Reproduce an anchor:** before making any new claim, each method must match
  one published score within a tolerance chosen in advance.
- RTN must have exactly zero spread across calibration samples.
- Repeats using the same sample must stay below the 25% limit.
- One manifest must list checkpoint hashes, calibration document IDs, lm-eval
  version, kernel and library versions, and GPU model. The analysis program
  must reject files from different manifests.

## Original decision rules

- **Full paper:** H2 passes. Name published rankings that reverse when a random,
  unreported calibration sample is redrawn.
- **Smaller paper:** H1 passes but H2 fails. Noise is real and similar to
  published differences, but no named ranking reverses. This would be a weaker
  reporting-standards paper that must face another earlier-work check.
- **Stop and report the negative:** sample-to-sample spread is tiny. This was
  always a realistic outcome: 128 sequences of 2,048 tokens are a large sample,
  and Hessian and activation measurements may be stable. A clean result saying
  PTQ is robust to new calibration samples could defend current practice and
  close the question in a short paper.

## Claims the old study was not allowed to make

- Not that calibration data matters; Williams & Aletras already showed that.
- Not that perplexity is a bad replacement for task accuracy; [LLM-KICK](https://arxiv.org/abs/2310.01382),
  *Accuracy is Not All You Need*, *[The Benchmark Illusion](https://arxiv.org/abs/2606.17609)*, and *Silent
  Failures* already cover that.
- Not that compression methods are useless. The question concerned the order
  between methods, not whether compression has value.
- Not that any author behaved badly. Using unidentified random calibration
  sequences was the normal field practice, so the study would examine the norm.

## Known risks in the old plan

1. The main prediction could be wrong, as the stop rule admitted.
2. GPU-kernel randomness could look like calibration-sample noise; this is why
   the same-sample control was required.
3. Official method implementations differ. Use each official repository and
   record its commit; do not rewrite a shared version.
4. With `K = 10`, `P_inv` changes only in steps of 0.01 in the `K x K` matrix.
   That is enough for a 0.10 threshold, but not for finer claims.
5. Some methods might fail to reproduce their own published anchor, as happened
   in *Revisiting RaBitQ and TurboQuant*. Report that as a result and remove the
   method from ranking analysis. Do not silently drop it.

## Items that had to be settled before running

1. Search older terms, data-selection and representative-subset research, and
   citations to Williams & Aletras and *Is C4 Optimal* for anyone redrawing
   calibration samples within one source. This gate later killed the plan.
2. Confirm every official implementation, commit, and calibration interface.
3. Fix the paper list and venue/date range.
4. Choose each method's allowed anchor-reproduction error.
5. Choose a practically important `R` in advance, so a tiny but statistically
   clear effect cannot count as success.
6. Confirm that a vector-quantization method is available and has a comparable
   calibration interface. Drop it if the comparison does not fit.

## Old compute estimate

Stage A required
`6 methods x 10 draws x 5 compression levels x 2 (replication) = 600`
compression
runs on a 7–8B model. Each run was expected to take 20–40 minutes including
evaluation: **about 300–400 GPU-hours, or two days on eight GPUs.** Stage B
would double the total. All work was inference only, with no training and no
dependence on released compressed checkpoints.

## Related

[[Compression-Audit-Direction]] — the survey and the proposed combined study.
[[KD-Noise-Floor-Stage1]] — the planned second case study in another research
area.

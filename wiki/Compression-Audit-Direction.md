# White-Box Compression — Survey, and the Convergence with the KD Audit

Status: **Surveyed 2026-07-26; the headline candidate is DEAD on both legs.**
*Updated 2026-08-02 for the general research wiki.* The survey text below is
preserved as record; the recommendation it reaches was falsified within days.

> **Outcome, read this first (2026-08-02). Do not run the study this page
> recommends.** Both halves of the "convergence" failed, for different reasons:
>
> 1. **The calibration-draw noise floor (the lead candidate below) — GATE
>    FAILED, SCOOPED.** See [[Calibration-Draw-Preregistration]]. Williams &
>    Aletras, *On the Impact of Calibration Data in Post-training Quantization
>    and Pruning* ([`2311.09755`](https://arxiv.org/abs/2311.09755), **ACL 2024**) ran exactly this experiment down
>    to the token count — ten non-overlapping draws per source, 1,800 compressed
>    models, 19,800 evaluations — and it has been independently replicated twice
>    ([`2410.17711`](https://arxiv.org/abs/2410.17711) ICLR 2025 with 20 seeds; [`2410.17170`](https://arxiv.org/abs/2410.17170) NAACL 2025). The wedge
>    asserted below ("prior work varies calibration *source or configuration*;
>    nobody varies *draws from the same source*") is **factually false** and was
>    asserted without reading the paper it cites. The measured answer, the thin
>    residue that genuinely remains ([AWQ](https://arxiv.org/abs/2306.00978)'s hardcoded seed, aggressive
>    bit-widths, generative/reasoning metrics, Hessian concentration theory, the
>    calibration monoculture), and the full post-mortem are on that page.
> 2. **The KD half — SUSPENDED**, not scooped. See [[KD-Noise-Floor-Stage1]].
>    It cleared every prior-art gate and then failed on audience and incentives:
>    the field already knows the norm, has measured it, published against it,
>    and continued, so the barrier is not knowledge. Plus a reviewer
>    conflict-of-interest problem inherent to auditing N papers.
>
> **The two general rules this cost bought.** (a) *Never state a wedge of the
> form "nobody varies X" from a paper's framing — open the method section and
> read what they varied.* (b) *Before drafting, answer: who changes their
> behaviour if this is true, and what stops them today?* If the answer is "the
> field already knows and the incentives are unchanged," the work is not worth
> doing however cleanly it gates. Both are now standing rules on [[Home]].
>
> **What on this page is still live:** the OpenReview access finding, the
> handling rules for numerical irregularities, the compression-landscape survey,
> and supporting questions Q3–Q5 (which were never gated — gate them before use).

## OpenReview chase — resolved, and the answer is mundane

**ICML 2025 does publish reviews for accepted papers.** From the official
reviewer instructions: *"Reviews and the discussion between reviewers and
authors for all accepted papers will be made public on OpenReview after the
reviewing period."*

The blocker is a **site-wide Cloudflare browser challenge** on `openreview.net`
that rejects every non-browser client regardless of IP. Ten programmatic routes
failed (API v1/v2, proxies, r.jina.ai, four CORS relays). Four Wayback snapshots
exist but are identical 48,905-byte shells — replies load client-side from the
blocked API and were never archived. Thirty-six HuggingFace OpenReview mirrors
have **zero ICML main-conference rows for any year**.

**Action: open the forum in a normal browser.** It is a bot challenge, not an
access restriction. Stop trying programmatically.

### Substitute evidence obtained from the camera-ready

Pulled PMLR v267 (`ko25a.pdf`) plus both arXiv versions:

- **Zero variance reporting in the camera-ready.** Word-boundary counts: `seed`
  0, `seeds` 0, `std` 0, `error bar` 0, `standard deviation` 0, `+/-` 0,
  `confidence interval` 0, `p-value` 0, `statistical` 0. Identical in arXiv v1
  and v2.
- **Nothing changed during review.** The relevant table rows are byte-identical
  across arXiv v1 (10 Mar 2025, pre-decision), v2, and the camera-ready. No
  reviewer-driven correction, and variance was never added.
- The `0.21` gap is confirmed: GKD AVG `56.14` versus DistiLLM AVG `56.35`.

### Numerical irregularities — handling rules, not findings

Four exact cell duplications across different teacher-student groups, plus one
arithmetic inconsistency: DistiLLM/Qwen2 prints AVG `56.35`, but
`(66.30 + 44.61 + 58.18)/3 = 56.363`, and the minimum achievable under any
rounding is `56.358`, so `56.35` is unreachable. GKD's `56.14` is exact.

**These are not to be used.** Each is individually explicable by ordinary
table-preparation error, which is common and not misconduct. Accumulating them
turns a methodological argument into an accusation against identifiable people,
which is both wrong and — separately — a reputational hazard to whoever
publishes it.

**Rule: report them privately to the authors and otherwise drop them.** They do
not appear in Stage 1, they are not evidence for any claim, and the fact that
they accumulate in one paper is a reason for *more* caution, not less.

**Consequent reframing of the KD study.** Use DistiLLM-1's published variance as
**the field's own noise estimate** — a gift, not an indictment — and treat the
0.21 gap as one illustrative delta among many harvested. The more irregularities
cluster in one paper, the more a DistiLLM-centred framing reads as an attack
regardless of intent.

## Compression survey: "smarter than quantization" is the wrong question

Quantization has already won empirically — [LLM-KICK](https://arxiv.org/abs/2310.01382), [Decoding Compressed Trust](https://arxiv.org/abs/2403.15447)
(ICML 2024) and [UniComp](https://arxiv.org/abs/2602.09130) independently find it dominates pruning on the
accuracy/compression frontier. And the information-theoretic thread has closed
most remaining headroom **on the objective everyone optimizes**: [WaterSIC](https://arxiv.org/abs/2603.04956) is
within `0.255` bits of the IT limit uniformly over covariances, the universality
price is `<= 0.11` bit, and [GSQ](https://arxiv.org/abs/2604.18556) shows even *scalar* quantization recovers most
of the vector-quantization gap at 2-3 bits.

**Every method lane is industry-gated.** [GPTQ](https://arxiv.org/abs/2210.17323)/[SparseGPT](https://arxiv.org/abs/2301.00774)/GSQ = IST Austria.
[QuIP#](https://arxiv.org/abs/2402.04396)/QTIP/[YAQA](https://arxiv.org/abs/2505.22988) = Cornell. [AWQ](https://arxiv.org/abs/2306.00978)/[SmoothQuant](https://arxiv.org/abs/2211.10438) = MIT Han Lab + NVIDIA. [SpinQuant](https://arxiv.org/abs/2405.16406) =
Meta. [BitNet](https://arxiv.org/abs/2310.11453)/[VPTQ](https://arxiv.org/abs/2409.17066) = Microsoft. [GPTVQ](https://arxiv.org/abs/2402.15319) = Qualcomm. [TurboQuant](https://arxiv.org/abs/2504.19874) = Google. A credible
PTQ paper now needs custom CUDA kernels — a multi-person quarter. The IT thread
(Ordentlich/Polyanskiy, HUJI + MIT) is pure theory, which is why an outsider
cannot win there either.

> **Partial correction (2026-08-02).** The custom-kernel bar is **not** general
> to compression. [[Direction-Reevaluation-2026-08]] refutes it for the
> KV-cache lane specifically: [KVTuner](https://arxiv.org/abs/2502.04420) (ICML 2025), [EvolKV](https://arxiv.org/abs/2509.08315) (EMNLP) and [SCBench](https://arxiv.org/abs/2412.10319)
> (ICLR 2025) all published with **zero kernel work** — the actual bar there is
> serving-compatible granularity plus throughput on top of existing kernels.
> Treat "needs custom kernels" as a claim about weight-PTQ competition against
> the named industrial groups, not as a property of compression research.

**"Smarter" saliency is mostly repackaged.** Magnitude → [Wanda](https://arxiv.org/abs/2306.11695) → GPTQ/SparseGPT
(Hessian/OBS) → AWQ → Fisher-based (YAQA, [GFWSVD](https://arxiv.org/abs/2505.17974)) are one second-order idea with
better curvature estimators. Circuit-aware compression is thin **for a reason**:
the interpretability literature runs the arrow backwards, using pruning to
*discover* circuits. And "compress what the model doesn't use" is ill-posed —
usage is input-distribution-dependent, so it collapses into calibration-set
choice.

Which is exactly where the opening is.

## The candidate: the calibration-draw noise floor *(GATE FAILED — do not run)*

> **Scooped, confirmed 2026-07-26.** Everything in this section is retained as
> record of the error. The wedge stated in the next-but-one paragraph is the
> false claim; the correct facts are in [[Calibration-Draw-Preregistration]].
> Three further factual errors were found in the same gate: [SparseGPT](https://arxiv.org/abs/2301.00774) and [Wanda](https://arxiv.org/abs/2306.11695)
> both already report mean ± std over 5 seeds in their appendices; [QuIP#](https://arxiv.org/abs/2402.04396)/QTIP do
> **not** use 128 C4 sequences (6,144 RedPajama sequences) and [AWQ](https://arxiv.org/abs/2306.00978) uses 16
> Pile-val sequences at length 512, so "everyone uses ~128 random C4" is wrong
> for three of the five methods named below; and [AQLM](https://arxiv.org/abs/2401.06118)'s published SD (`0.127` at
> 128 sequences → `0.005` at 4,096) predicts the effect is negligible at the
> sample counts modern methods actually use. **The expected result was null and
> that was knowable from the papers being cited.**

**Nobody treats the calibration set as a random seed.** PTQ papers use "128
random C4 sequences" and virtually none report *which* 128, let alone resample.
The standard defence is that PTQ is deterministic given a calibration set — but
the set is itself a random draw. That is a legitimate, universally present, and
universally unsampled noise source.

**The wedge, stated precisely.** Prior work varies calibration **source or
configuration** — Williams & Aletras (ACL 2024, "substantial variations in
downstream task performance"), *Is C4 Optimal for Pruning?*, *Rethinking Layer
Redundancy* ("the calibration configuration plays a substantially larger role
than the choice of search algorithm"). Those treat it as a **design factor**.
**Nobody varies draws from the same source — a noise source.** State that
distinction explicitly in the paper or a reviewer will conflate them.

**Design.** For each of {[GPTQ](https://arxiv.org/abs/2210.17323), AWQ, SparseGPT, Wanda, a QTIP-class VQ method,
one IT-optimal method}, draw `K = 10` independent 128x2048 calibration sets from
the same source, compress, evaluate. **Primary pre-registered quantity: the
ratio of between-method spread to within-method calibration-draw spread, per
benchmark x bit-width cell.** Prediction: in a majority of cells, published
orderings do not survive calibration resampling. The currency of this literature
is `0.05-0.2` PPL and `0.5-1.5` accuracy points.

**Feasibility is the decisive advantage.** ~75 compressed models, roughly **400
GPU-hours, about two days on eight GPUs** — all inference-only one-shot
compression, no retraining, no released-checkpoint dependency. Contrast the KD
Stage 2, which needs full retraining because DistiLLM-2 released no student
checkpoints.

**Published precedent that this genre lands here.** *Revisiting RaBitQ and
TurboQuant: A Symmetric Comparison* ([`2604.19528`](https://arxiv.org/abs/2604.19528), Apr 2026) overturns a Google
result — [TurboQuant](https://arxiv.org/abs/2504.19874) worse than [RaBitQ](https://arxiv.org/abs/2405.12497) in most settings, and *"several reported
runtime and recall results in the TurboQuant paper could not be reproduced from
the released implementation under the stated configuration."*

## Supporting questions, in priority order

**Q3 (intellectually strongest).** Does the IT-optimal proxy buy end-task
capability? Measure rank correlation between layerwise proxy loss and a
downstream capability battery across methods. **[WaterSIC](https://arxiv.org/abs/2603.04956) concedes the limitation
itself** — it optimizes "Euclidean post-matmul loss, not PPL/KL targets," with
no seeds, on **Llama-3.2-1B**, evaluated by **single-run WikiText-2
perplexity**. A near-optimality theorem validated that way is a visible
evidentiary gap. This is the honest answer to "is there something smarter than
quantization": not a better compressor, but evidence the objective is loosely
coupled to what matters.

**Q4 (turns critique into an instrument).** Compression x probability
calibration. ECE/Brier/selective-prediction AUC versus bit-rate, against Q1's
noise floor. Prediction: calibration degrades earlier and more
method-discriminatively than accuracy — i.e. it is a **higher-SNR discriminator
between compression methods**. Area is thin: essentially only [`2606.24970`](https://arxiv.org/abs/2606.24970).

**Q5.** Multilingual is the least-crowded capability cell (~35 hits versus ~210
for reasoning) with a documented calibration-*language* confound ([`2408.14398`](https://arxiv.org/abs/2408.14398)).

**Avoid.** Recognition-versus-production (*The Benchmark Illusion*,
[`2606.17609`](https://arxiv.org/abs/2606.17609)) is one month old and its authors will extend it. KV-cache
compression is the most crowded area in the field (556 hits, new papers weekly)
and already has its own critique layer.

> **Superseded 2026-08-02 — the KV-cache "avoid" was a crowd-count verdict, and
> crowd counts predicted nothing.** [[Direction-Reevaluation-2026-08]] rates
> KV-cache **★★★★ when narrowed to safety-aware allocation**: six targeted
> searches found no safety-objective allocator at all, [KVFundaBench](https://arxiv.org/abs/2502.01941) v2 dropped
> safety from its abstract, and [`2510.00231`](https://arxiv.org/abs/2510.00231) documents instructions "completely
> ignored" under compression. The crowded part is the perplexity-objective part.
> Note the direction of the correction — **a direction is filtered out only when
> the remaining opportunity is gone, never because it is hot** — and note that
> Q4 below (compression × probability calibration) survives for the same reason:
> the objective everyone optimizes is loosely coupled to what matters. See
> [[Method-Opportunities]] Cluster 2.

## The convergence

The compression Q1 and the KD Stage 1 are **the same study**: take a randomness
source the literature leaves unreported, measure the noise floor, and show which
published orderings do not survive it. Run together they become a general
methodological contribution with two case studies, sharing one instrument.

**Recommended ordering: compression first.** It needs no retraining, has no
released-checkpoint dependency, does not centre on one paper or one group, and
has a published precedent. The KD analysis then becomes the second case study,
carrying the free self-refutation hook without carrying the reputational risk of
being the headline.

> **Both legs dead — see the banner at the top of this page.** The compression
> leg was scooped ([[Calibration-Draw-Preregistration]]); the KD leg was
> suspended on incentives ([[KD-Noise-Floor-Stage1]]). The observation that the
> two are *the same study* was correct and is the one thing worth carrying
> forward: a noise-floor instrument built once transfers across literatures.
> But it also concentrated the risk — a single false premise about "unreported
> randomness" took out both case studies at once. **When two candidates share
> an instrument, they usually share a failure mode; gate the shared premise
> first, not each case study separately.**

## Related

[[Calibration-Draw-Preregistration]] — the gate that killed the lead candidate,
with the measured answer and the thin residue that remains.
[[KD-Noise-Floor-Stage1]] — the companion analysis; measurements verified,
study suspended on audience/incentive grounds.
[[KD-Evidence-Audit-Gate]] — its gate, and the handling rules for numerical
irregularities.
[[Direction-Reevaluation-2026-08]] — current direction ratings; corrects the
KV-cache and custom-kernel verdicts on this page.
[[Calibration-Opportunity-Survey]] — the *other* "calibration" direction, also
closed; the two are frequently confused.

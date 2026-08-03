# White-Box Model Compression — Survey and Connection to the KD Audit

Status: **Surveyed 2026-07-26; both parts of the main idea are DEAD.**
*Updated 2026-08-02 for the general research wiki.* This page keeps the survey
as a research record, but later checks disproved its recommendation within
days.

**White-box compression** makes a model smaller or cheaper while using its
internal weights or activations. **Post-training quantization (PTQ)** stores
weights with fewer bits after training. **Pruning** removes weights. **Knowledge
distillation (KD)** trains a smaller model to copy a larger one.

> **Final outcome, read first (2026-08-02). Do not run the study recommended
> below.** The two parts failed for different reasons.
>
> 1. **Random calibration-sample noise, the main idea below: ALREADY PUBLISHED;
>    GATE FAILED.** See [[Calibration-Draw-Preregistration]]. Williams &
>    Aletras, *On the Impact of Calibration Data in Post-training Quantization
>    and Pruning* ([`2311.09755`](https://arxiv.org/abs/2311.09755), **ACL 2024**) ran the same experiment with
>    the same token count: ten non-overlapping samples from each source, 1,800
>    compressed models, and 19,800 evaluations. Two papers independently
>    repeated it: [`2410.17711`](https://arxiv.org/abs/2410.17711) at ICLR 2025 with 20 seeds, and [`2410.17170`](https://arxiv.org/abs/2410.17170)
>    at NAACL 2025. The claim below—"earlier work changes the source or setup,
>    but nobody redraws samples from the same source"—is **factually false**.
>    It was written without reading the methods of the paper it cited. The
>    linked page gives the measured result, full failure analysis, and the few
>    pieces still open: [AWQ](https://arxiv.org/abs/2306.00978)'s fixed seed, very low bit-widths,
>    generation/reasoning tests, theory for Hessian concentration, and the
>    field's use of the same few calibration sets.
> 2. **The KD part: SUSPENDED, not already published.** See
>    [[KD-Noise-Floor-Stage1]]. It passed every earlier-work check but failed a
>    more basic test of audience and incentives. Researchers already know the
>    convention, have measured it, have published criticism of it, and have not
>    changed it. Also, auditing N papers creates a reviewer conflict-of-interest
>    problem because likely reviewers may have written those papers.
>
> Two rules came from these failures. First, never claim "nobody changes X"
> from an abstract or framing; read the methods and verify what was changed.
> Second, before writing a plan, ask who would act differently if the result is
> true and what currently stops them. If the answer is "people already know,
> but incentives favor no change," a clean paper will not solve the problem.
> Both are now standing rules on [[Home]].
>
> Parts that remain useful: the OpenReview access finding, rules for handling
> odd table values, the map of compression research, and questions Q3–Q5. Q3–Q5
> have not passed earlier-work checks and must be checked before use.

## OpenReview access: the simple answer

**ICML 2025 does publish reviews for accepted papers.** Its official reviewer
instructions say that reviews and discussions for all accepted papers become
public on OpenReview after review ends.

The problem was a **site-wide Cloudflare browser challenge** on
`openreview.net`. It blocked every automated client, no matter which IP address
was used. Ten programmatic routes failed: API v1, API v2, proxies, r.jina.ai,
and four CORS relays. The Wayback Machine has four snapshots, but each is the
same 48,905-byte page shell. Replies load later in the browser from the blocked
API, so the archive never captured them. Thirty-six Hugging Face OpenReview
mirrors contain **zero ICML main-conference rows for every year**.

**Use a normal web browser to open the forum.** This is an anti-bot challenge,
not a rule hiding the reviews. Stop trying automated routes.

### Evidence recovered from the final paper

PMLR volume 267 (`ko25a.pdf`) and both arXiv versions show:

- **The final paper reports no variation.** Exact word counts are `seed` 0,
  `seeds` 0, `std` 0, `error bar` 0, `standard deviation` 0, `+/-` 0,
  `confidence interval` 0, `p-value` 0, `statistical` 0. arXiv v1 and v2 are
  the same on these counts.
- **Review did not change the relevant evidence.** The table rows are
  byte-for-byte the same in arXiv v1 (10 Mar 2025, before the decision), v2,
  and the final paper. Review added no variance report or correction.
- The `0.21` difference is real: GKD AVG is `56.14`, and DistiLLM AVG is
  `56.35`.

### Odd numbers in the table: rules for handling them

The table contains four exact repeated cells across different teacher-student
groups and one arithmetic mismatch. DistiLLM/Qwen2 prints AVG `56.35`, but
`(66.30 + 44.61 + 58.18)/3 = 56.363`. Even the lowest value possible after
rounding is `56.358`, so `56.35` cannot result from those three printed values.
GKD's `56.14` average is exact.

**Do not use these oddities as evidence.** Any one could come from an ordinary
table-preparation error, which is common and is not misconduct. Listing several
together would turn a methods argument into an accusation against named people.
That would be wrong and could also harm the person making the claim.

**Tell the authors privately, and otherwise leave these values out.** They do
not belong in Stage 1 and do not support any public claim. Several oddities in
one paper are a reason for *more* care, not less.

The KD study must also avoid centering one paper. Use DistiLLM-1's published
variation as **the field's own estimate of its noise**, not as an attack. Treat
the 0.21 difference as one example among many published differences.

## Compression research: asking for something "smarter" is the wrong start

Quantization already wins on measured accuracy for a given compression level.
[LLM-KICK](https://arxiv.org/abs/2310.01382), [Decoding Compressed Trust](https://arxiv.org/abs/2403.15447) (ICML 2024), and [UniComp](https://arxiv.org/abs/2602.09130) independently find
that it beats pruning on this tradeoff. Information-theory work has also used
most of the remaining room **for the exact goal that the field optimizes**:
[WaterSIC](https://arxiv.org/abs/2603.04956) stays within `0.255` bits of the information-theoretic limit for all
covariance patterns, the cost of using one general method is `<= 0.11` bit, and
[GSQ](https://arxiv.org/abs/2604.18556) shows that even *scalar* quantization recovers most of the remaining gap
with vector quantization at 2–3 bits.

Each main method family is led by large, specialized teams:

- [GPTQ](https://arxiv.org/abs/2210.17323), [SparseGPT](https://arxiv.org/abs/2301.00774), and GSQ: IST Austria.
- [QuIP#](https://arxiv.org/abs/2402.04396), QTIP, and [YAQA](https://arxiv.org/abs/2505.22988): Cornell.
- [AWQ](https://arxiv.org/abs/2306.00978) and [SmoothQuant](https://arxiv.org/abs/2211.10438): MIT Han Lab and NVIDIA.
- [SpinQuant](https://arxiv.org/abs/2405.16406): Meta.
- [BitNet](https://arxiv.org/abs/2310.11453) and [VPTQ](https://arxiv.org/abs/2409.17066): Microsoft.
- [GPTVQ](https://arxiv.org/abs/2402.15319): Qualcomm.
- [TurboQuant](https://arxiv.org/abs/2504.19874): Google.

A competitive weight-PTQ paper now needs custom CUDA kernels, roughly a full
quarter of work for several people. The information-theory line led by
Ordentlich/Polyanskiy at HUJI and MIT requires pure theory, so it is also hard
for an outsider to enter.

> **Correction added 2026-08-02.** Custom kernels are **not required for every
> compression topic**. [[Direction-Reevaluation-2026-08]] disproves that claim
> for KV-cache compression. [KVTuner](https://arxiv.org/abs/2502.04420) (ICML 2025), [EvolKV](https://arxiv.org/abs/2509.08315) (EMNLP), and
> [SCBench](https://arxiv.org/abs/2412.10319) (ICLR 2025) all published with **no kernel work**. In that area,
> the real requirement is a level of control that serving systems can use, plus
> throughput measurements on existing kernels. The custom-kernel statement
> applies only to competing in weight PTQ against the named industry groups.

Many supposedly "smarter" importance scores are the same second-order idea in
new forms: magnitude → [Wanda](https://arxiv.org/abs/2306.11695) → GPTQ/SparseGPT using Hessian/OBS information →
AWQ → Fisher-based methods such as YAQA and [GFWSVD](https://arxiv.org/abs/2505.17974). They mostly improve how
curvature is estimated. Circuit-aware compression is thin for a reason:
interpretability research usually runs the idea backwards and uses pruning to
*find* circuits. "Compress what the model does not use" is not fully defined,
because use changes with the input data. The question becomes one of which
calibration examples are chosen.

That observation originally led to the failed idea below.

## Historical main idea: noise from the random calibration sample — DO NOT RUN

> **Confirmed already published on 2026-07-26.** This section stays only to
> record the mistake. Its central claim is false; use
> [[Calibration-Draw-Preregistration]] for the correct facts. The same check
> found three more errors. [SparseGPT](https://arxiv.org/abs/2301.00774) and [Wanda](https://arxiv.org/abs/2306.11695) already report mean ± standard
> deviation over 5 seeds in their appendices. [QuIP#](https://arxiv.org/abs/2402.04396) and QTIP use 6,144
> RedPajama sequences, **not** 128 C4 sequences. [AWQ](https://arxiv.org/abs/2306.00978) uses 16 Pile-val
> sequences of length 512. Thus, "everyone uses about 128 random C4 sequences"
> was wrong for three of the five named methods. [AQLM](https://arxiv.org/abs/2401.06118)'s published standard
> deviation drops from `0.127` at 128 sequences to `0.005` at 4,096. That
> evidence predicted that modern methods' large sample counts would make the
> effect tiny. The likely result was null, and the cited papers already made
> that knowable.

The old claim said that nobody treated the calibration set as a random seed.
It said PTQ papers used "128 random C4 sequences," usually did not list those
sequences, and never drew a new sample. Although PTQ is deterministic once the
sample is fixed, the sample itself is random. The draft called this a universal
but unmeasured source of noise.

The exact unanswered question was supposed to be: earlier work changes the
calibration **data source or setup**, treating it as a design choice. Examples
were Williams & Aletras (ACL 2024, "substantial variations in downstream task
performance"), *Is C4 Optimal for Pruning?*, and *Rethinking Layer Redundancy*
("the calibration configuration plays a substantially larger role than the
choice of search algorithm"). The draft incorrectly claimed that **nobody
redrew samples from the same source** to measure randomness. Williams &
Aletras had already done exactly that.

The old design was to take [GPTQ](https://arxiv.org/abs/2210.17323), AWQ, SparseGPT, Wanda, a QTIP-style vector
quantization method, and one information-theory-optimal method. For each, draw
`K = 10` independent calibration samples with 128 sequences of 2,048 tokens
from the same source, compress the model, and evaluate it. The **main planned
measurement** was the ratio of score spread between methods to score spread
between random calibration draws of one method, for every benchmark and
bit-width. The prediction said that random redraws would reverse the published
method order in most cells. Typical published differences were only `0.05-0.2`
perplexity and `0.5-1.5` accuracy points.

The main practical advantage appeared to be cost: about 75 compressed models,
about **400 GPU-hours, or two days on eight GPUs**. All work used one-shot
inference-only compression. It needed no retraining and no released student
checkpoints. KD Stage 2 was much harder because DistiLLM-2 released no student
checkpoints and required full retraining.

The style of paper did have a recent publication example. *Revisiting RaBitQ
and TurboQuant: A Symmetric Comparison* ([`2604.19528`](https://arxiv.org/abs/2604.19528), Apr 2026) overturned a
Google result: [TurboQuant](https://arxiv.org/abs/2504.19874) was worse than [RaBitQ](https://arxiv.org/abs/2405.12497) in most settings, and several
reported speed and recall results could not be reproduced from the released
code using the stated settings.

## Other questions, ordered by priority

These questions have **not** passed earlier-work checks.

### Q3 — strongest scientific question

Does an information-theory-optimal proxy loss actually preserve task ability?
Across methods, measure rank correlation between per-layer proxy loss and a set
of downstream capability tests. **[WaterSIC](https://arxiv.org/abs/2603.04956) states this limit itself:** it
optimizes "Euclidean post-matmul loss, not PPL/KL targets." It reports no seeds,
uses **Llama-3.2-1B**, and evaluates with **one WikiText-2 perplexity run**. A
near-optimality theorem checked in this narrow way leaves an obvious evidence
gap. The better question is not "can we invent a smarter compressor?" but "is
the optimized score weakly connected to what users care about?"

### Q4 — turn the criticism into a measurement tool

Study compression against probability calibration. Measure ECE, Brier score,
and selective-prediction AUC at each bit-rate, compared with Q1's noise floor.
Prediction: calibration gets worse before accuracy does and separates methods
more clearly. In other words, it gives a **higher signal-to-noise ratio (SNR)**
for comparing compression methods. This area appears thin; the only close work
found was [`2606.24970`](https://arxiv.org/abs/2606.24970).

### Q5 — multilingual behavior

Multilingual capability is the least crowded cell found, with about 35 results
versus about 210 for reasoning. It also has a documented hidden factor: the
language of the calibration data changes the result ([`2408.14398`](https://arxiv.org/abs/2408.14398)).

### Areas the original survey said to avoid

Recognition versus production was covered one month earlier by *The Benchmark
Illusion* ([`2606.17609`](https://arxiv.org/abs/2606.17609)), whose authors were likely to extend it. The survey also called KV-cache
compression the busiest area, with 556 search results and new papers every
week, and said it already had a criticism literature.

> **The KV-cache warning was superseded on 2026-08-02.** Counting papers did not
> predict whether a focused question remained open.
> [[Direction-Reevaluation-2026-08]] rates safety-aware KV-cache allocation
> **★★★★**. Six focused searches found no allocator that optimizes a safety
> goal. [KVFundaBench](https://arxiv.org/abs/2502.01941) version 2 removed safety from its abstract, while
> [`2510.00231`](https://arxiv.org/abs/2510.00231) reports that instructions can be "completely ignored" after compression.
> Only the perplexity-focused part is crowded. The lesson is that a direction
> should be rejected only when its remaining focused opportunity is gone, not
> because many people work nearby. Q4 survives for the same reason: the common
> optimization goal may not match what matters. See [[Method-Opportunities]],
> Cluster 2.

## Historical link between the compression and KD studies

The old compression Q1 and KD Stage 1 had the same structure: take a source of
randomness that papers do not report, measure its noise floor, and find which
published method rankings do not survive. Together, they were supposed to form
one methods paper with two case studies and one shared measurement tool.

The recommended order was compression first. It needed no retraining or
released checkpoints, did not focus on one group, and had a published example.
KD would then be the second case study, keeping its useful self-contradiction
without making one paper or group the main target.

> **Both parts are now dead; see the top of this page.** Compression was already
> published ([[Calibration-Draw-Preregistration]]). KD was suspended because
> the audience already knew and incentives did not support change
> ([[KD-Noise-Floor-Stage1]]). The one useful idea is that one noise-floor tool
> can transfer across research areas. But sharing a tool also made both ideas
> depend on the same claim. One false assumption about "unreported randomness"
> damaged both. **When two ideas use the same tool, check their shared failure
> point first rather than checking each case separately.**

## Related

[[Calibration-Draw-Preregistration]] — the gate that ended the main idea, the
measured answer, and the few open pieces.
[[KD-Noise-Floor-Stage1]] — verified KD measurements; study suspended because
of audience and incentives.
[[KD-Evidence-Audit-Gate]] — the KD gate and rules for handling odd numbers.
[[Unified-Direction-Ranking-2026-08]] — current direction decisions.
[[Direction-Reevaluation-2026-08]] — historical re-evaluation that first
corrected the KV-cache and custom-kernel claims here; its ranking was later
superseded.
[[Calibration-Opportunity-Survey]] — the other calibration idea, also closed
and often confused with this one.

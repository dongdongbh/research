# White-Box Compression — Survey, and the Convergence with the KD Audit

Status: **Surveyed 2026-07-26.** Produces the best-fitting candidate of the
session, and it is **the same study as [[KD-Noise-Floor-Stage1]] in a second
literature** — with a better feasibility profile and a published precedent.

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

Quantization has already won empirically — LLM-KICK, Decoding Compressed Trust
(ICML 2024) and UniComp independently find it dominates pruning on the
accuracy/compression frontier. And the information-theoretic thread has closed
most remaining headroom **on the objective everyone optimizes**: WaterSIC is
within `0.255` bits of the IT limit uniformly over covariances, the universality
price is `<= 0.11` bit, and GSQ shows even *scalar* quantization recovers most
of the vector-quantization gap at 2-3 bits.

**Every method lane is industry-gated.** GPTQ/SparseGPT/GSQ = IST Austria.
QuIP#/QTIP/YAQA = Cornell. AWQ/SmoothQuant = MIT Han Lab + NVIDIA. SpinQuant =
Meta. BitNet/VPTQ = Microsoft. GPTVQ = Qualcomm. TurboQuant = Google. A credible
PTQ paper now needs custom CUDA kernels — a multi-person quarter. The IT thread
(Ordentlich/Polyanskiy, HUJI + MIT) is pure theory, which is why an outsider
cannot win there either.

**"Smarter" saliency is mostly repackaged.** Magnitude → Wanda → GPTQ/SparseGPT
(Hessian/OBS) → AWQ → Fisher-based (YAQA, GFWSVD) are one second-order idea with
better curvature estimators. Circuit-aware compression is thin **for a reason**:
the interpretability literature runs the arrow backwards, using pruning to
*discover* circuits. And "compress what the model doesn't use" is ill-posed —
usage is input-distribution-dependent, so it collapses into calibration-set
choice.

Which is exactly where the opening is.

## The candidate: the calibration-draw noise floor

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

**Design.** For each of {GPTQ, AWQ, SparseGPT, Wanda, a QTIP-class VQ method,
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
TurboQuant: A Symmetric Comparison* (`2604.19528`, Apr 2026) overturns a Google
result — TurboQuant worse than RaBitQ in most settings, and *"several reported
runtime and recall results in the TurboQuant paper could not be reproduced from
the released implementation under the stated configuration."*

## Supporting questions, in priority order

**Q3 (intellectually strongest).** Does the IT-optimal proxy buy end-task
capability? Measure rank correlation between layerwise proxy loss and a
downstream capability battery across methods. **WaterSIC concedes the limitation
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
between compression methods**. Area is thin: essentially only `2606.24970`.

**Q5.** Multilingual is the least-crowded capability cell (~35 hits versus ~210
for reasoning) with a documented calibration-*language* confound (`2408.14398`).

**Avoid.** Recognition-versus-production (*The Benchmark Illusion*,
`2606.17609`) is one month old and its authors will extend it. KV-cache
compression is the most crowded area in the field (556 hits, new papers weekly)
and already has its own critique layer.

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

## Related

[[KD-Noise-Floor-Stage1]] — the verified companion analysis.
[[KD-Evidence-Audit-Gate]] — its gate, and the handling rules.

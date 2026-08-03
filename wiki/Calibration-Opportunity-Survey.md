# Calibration for Discriminative Matching — Opportunity Survey

Status: **Survey complete 2026-07-25; direction subsequently gated and NOT
viable as framed.** *Updated 2026-08-02 for the general research wiki.* Two
independent searches converged on the same gaps, but the gate run against the
conformal probe pre-registration (svib repo wiki) falsified one leg of the
original recommendation. See "Prior-art correction" below before using this
page.

> **Outcome, read this first (2026-08-02).** Two separate directions carry the
> word "calibration" and **both were closed**. Do not restart either from this
> page without re-reading its gate.
>
> 1. **This page's direction — proper-scoring calibration of the image-text
>    matching decision.** [[Calibration-Prior-Art-Gate]] found both gaps
>    literally unclaimed **but the direction not viable as framed**: temperature
>    scaling is monotone, and AURC / risk-coverage curves depend only on the
>    *ranking* of confidences, so the only fix a calibration paper would offer
>    leaves both metric families **bit-identical**. Decision delta exactly zero
>    against a bar of diagnosis → mechanism → fix → changed decision. The gate
>    also found three papers that have since taken pieces of Gap A
>    ([`2607.03143`](https://arxiv.org/abs/2607.03143), three weeks old at gate time, publishes the exact protocol
>    at ~400-way retrieval). Only the three escape routes listed there —
>    threshold transfer, CLIP-as-scorer downstream, cross-model fusion — carry a
>    changed decision.
> 2. **The separately-named "calibration-draw" direction** (treating the
>    *calibration set* as a random seed in LLM compression, a different topic
>    that lives in [[Compression-Audit-Direction]]) was **GATE-FAILED — SCOOPED,
>    do not run**: see [[Calibration-Draw-Preregistration]]. It was published in
>    ACL 2024 and independently replicated twice.
>
> What stays useful here regardless: the saturation verdicts table, the
> refuted K-mismatch hypothesis (so it is not retried), the terminology
> disambiguation, and the rank-invariance trap.

"Calibration" here means proper-scoring calibration (ECE, Brier, reliability
diagrams), not conformal coverage.

## Prior-art correction (2026-07-25, same day)

The mandatory gate on the conformal probe located work this survey missed:

- **Leveraging Data to Say No: Memory Augmented Plug-and-Play Selective
  Prediction** (ICLR 2026, OpenReview [`wWxdT6LB2D`](https://openreview.net/forum?id=wWxdT6LB2D)) reports AURC and
  risk-coverage on [SugarCrepe](https://arxiv.org/abs/2306.14610), [Winoground](https://arxiv.org/abs/2204.03162), [What'sUp](https://arxiv.org/abs/2310.19785), [VL-Checklist](https://arxiv.org/abs/2207.00221), and [Foil](https://arxiv.org/abs/1705.01359).
- **Look Again Before You Abstain** ([arXiv `2606.16667`](https://arxiv.org/abs/2606.16667), v4) further crowds VLM
  conformal abstention.

**Consequence for this page.** The "consequence" leg of the merged-paper
proposal below — risk-coverage evaluation on compositional matching — is
**claimed, not open**. Gap A and Gap B are unaffected by these two papers on
their face, because selective-prediction papers report AURC rather than ECE or
Brier, and neither does a source-matched fine-tuning comparison. **But that is
an inference, not a verification.** Before any run, both papers must be read
directly to confirm they contain no reliability-diagram or proper-scoring
analysis and no source-matched calibration comparison. Treat Gap A as
*plausibly* open until then.

Process note: this survey's zero-hit arXiv phrase queries were too narrow, the
same failure mode recorded in [[Next-Direction-Literature-Survey]]. Exact-phrase
searching does not recover work that uses different vocabulary
("selective prediction" plus benchmark names). Future gates must search by
benchmark name and by task, not only by method term.

## Verdicts

| Area | Verdict | Detail |
|---|---|---|
| Calibration of CLIP zero-shot **classification** | **SATURATED** | LeVine et al. [2303.12748](https://arxiv.org/abs/2303.12748); Tu et al. NeurIPS 2023 ([2402.07410](https://arxiv.org/abs/2402.07410)) and ICML 2024 ([2402.07417](https://arxiv.org/abs/2402.07417)). Consensus: mildly overconfident, single-digit ECE, and almost entirely fixed by one global temperature |
| Prompt-tuning calibration | **OVERSATURATED** | [C-TPT](https://arxiv.org/abs/2403.14119) (ICLR'24) → [DAC](https://arxiv.org/abs/2402.04655) (ICML'24) → [DOR](https://arxiv.org/abs/2410.02681) (ICML'25) → [O-TPT](https://arxiv.org/abs/2503.12096) (CVPR'25) → [D-TPT](https://arxiv.org/abs/2510.09473) → [TCPT](https://arxiv.org/abs/2602.19024) (CVPR'26). Four generations in two years on the same 11 datasets. Do not enter |
| Calibration under distribution shift | **SATURATED for classification** | [CaRot](https://arxiv.org/abs/2311.01723) (NeurIPS'24) gives a theoretical bound; [Murugesan et al.](https://arxiv.org/abs/2407.13588) (ECCV'24) show adapters/TTA break it. All classification |
| Calibrated ranking (IR/ads) | **SATURATED** | KDD'22 → CIKM'23 → KDD'23 → SIGIR'25 → MLPlatt 2026. Mature industrial subfield; never ported to cross-modal |
| **Calibration of the image-text MATCHING decision** | **OPEN** | See below |
| **Hard-negative fine-tuning → miscalibration** | **OPEN** | See below |

## The two open gaps

### Gap A: nobody has measured whether matching confidence is calibrated

Searched across seven query formulations in one agent and independently in a
second. **No paper computes ECE, Brier, NLL, or reliability diagrams for the
binary image-text matching decision.** Nothing on [SugarCrepe](https://arxiv.org/abs/2306.14610), [ARO](https://arxiv.org/abs/2210.01936), [VALSE](https://arxiv.org/abs/2112.07566), or
[Winoground](https://arxiv.org/abs/2204.03162) asks whether the model's confidence in picking the right caption is
calibrated.

Stated precisely: for `p = sigma(logit_scale * (s_pos - s_neg))`, nobody has
reported "when CLIP says 0.9 on SugarCrepe, it is right X% of the time."

Closest existing work is a **side table**: Oh et al., Geodesic Multi-Modal
Mixup (NeurIPS 2023, [2203.03897](https://arxiv.org/abs/2203.03897)) reports Flickr30k retrieval ECE — zero-shot
`1.90/1.88`, naive fine-tune `2.26/2.00`. Three numbers in a mixup paper, no
definition of what confidence means for retrieval, no analysis.

Everything else that looks adjacent is something else: the probabilistic-
embedding line ([PCME](https://arxiv.org/abs/2101.05068), [PCME++](https://arxiv.org/abs/2305.18171), [ProbVLM](https://arxiv.org/abs/2307.00398), [GroVE](https://arxiv.org/abs/2505.05163), Post-hoc Probabilistic VLMs
ICLR'26) plots uncertainty-bin versus Recall@1, a monotonicity diagnostic with
no proper scoring rule; the hubness/score-normalization line ([QB-Norm](https://arxiv.org/abs/2112.12777) CVPR'22,
[DBNorm](https://arxiv.org/abs/2310.11612), Sinkhorn, Test-Time Distribution Normalization NeurIPS'23) fixes
ranking and reports R@K, never ECE.

### Gap B: hard-negative fine-tuning versus its own source checkpoint

Source-matched calibration comparison is **standard practice** in the VLM
literature — Fine-Tuning is Fine, if Calibrated (NeurIPS'24, [2409.16223](https://arxiv.org/abs/2409.16223)),
CAC ([2501.19060](https://arxiv.org/abs/2501.19060)), [DAC](https://arxiv.org/abs/2402.04655), [DOR](https://arxiv.org/abs/2410.02681), [CaRot](https://arxiv.org/abs/2311.01723) all use the zero-shot checkpoint as
reference. **Every one of them studies base/new-class classification with
prompt tuning or adapters.** None studies contrastive hard-negative
fine-tuning, and none evaluates on matching or retrieval.

No paper states: *contrastive hard-negative fine-tuning systematically inflates
confidence, measured as ECE against the source checkpoint.* The second agent
called this "the most under-claimed link in the whole survey."

The prior that it is true is strong and already published as accuracy-only
findings: [SugarCrepe](https://arxiv.org/abs/2306.14610) (NeurIPS'23 D&B) showed NegCLIP exploits hard-negative
artifacts; [The Hard Positive Truth](https://arxiv.org/abs/2409.17958) (ECCV'24) measured up to `38.7%` drops for
hard-negative-finetuned CLIP under hard positives; Oh et al. already show plain
fine-tuning worsens retrieval ECE. [Murugesan et al.](https://arxiv.org/abs/2407.13588)'s diagnosis — adaptation
degrades calibration via **increased logit range** — transfers to this setting
almost mechanically and nobody has run it.

**The instrument for this already existed.** The SVIB project's Stage E built
five source-matched checkpoint pairs (hard-negative-finetuned model versus its
own pretraining source) with hashes, provenance and reproduced published
endpoints, plus per-item records. Anyone testing Gap B needs exactly that
construction — a *source-matched* pair, not a method comparison — and it is the
expensive part. Full detail: **svib repo wiki**, Stage-E pages.

## My K-mismatch hypothesis was wrong, and is recorded here so it is not retried

Proposed hypothesis: the learned `logit_scale` is optimized for in-batch
contrastive classification at batch size N, then applied to K-way decisions at
test time (K=2 SCPP++, K=4 [Winoground](https://arxiv.org/abs/2204.03162), K=5000 COCO retrieval), creating
uncorrected K-dependent miscalibration.

Refuted on three counts:

1. **Factually wrong mechanism.** CLIP's `logit_scale` is **clamped at 100**
   (tau = 0.01) and essentially every released checkpoint sits *at the clamp*.
   It is a training-stability artifact, not an optimum for batch size N.
2. **The theorem is published.** "softmax is not enough (for sharp size
   generalisation)" (Veličković et al., **ICML 2025**, [2410.01104](https://arxiv.org/abs/2410.01104)) proves
   softmax provably disperses as item count grows at test time and proposes
   adaptive inference temperature. K-dependent dispersion is a corollary.
3. **"Nobody corrects it" is false.** [Inverted softmax](https://arxiv.org/abs/1702.03859), [DSL](https://arxiv.org/abs/2109.04290), [QB-Norm](https://arxiv.org/abs/2112.12777) (CVPR'22),
   [DBNorm](https://arxiv.org/abs/2310.11612), Sinkhorn normalization, and Test-Time Distribution Normalization
   (NeurIPS'23) all renormalize over the actual candidate set. The recsys logQ
   correction line has addressed the train/test normalizer mismatch since 2019.
4. **Contradicting evidence exists.** LeVine et al. and Tu et al. both find a
   single temperature transfers across K in `[10, 1000]` and across label sets
   for classification.

The salvageable version is sharper than the original: the temperature every
downstream decision inherits is **a saturated stability constraint carrying no
information about any downstream decision problem** — neither the pretraining
batch size nor the deployment candidate count. Use as a secondary axis with the
prior art cited up front, never as the headline.

## Hard constraint on any paper here

Both agents independently reported that **measurement-only calibration papers
are dead at main tracks**. The accepted template is uniformly
**diagnosis → mechanism → cheap fix → a changed decision**. Descriptive ECE
studies now go to D&B tracks or the ICML Calibration workshop.

**A trap to avoid:** for a 2AFC decision, `p = sigma(tau * (s_pos - s_neg))` is
monotone in tau, so **temperature scaling cannot change which caption wins**.
Accuracy is invariant. No claim that recalibration changes a published
*accuracy* conclusion is available here, and asserting one would be wrong.

What recalibration *does* change: abstention and threshold decisions,
risk-coverage curves, and cross-model comparison at matched confidence. The
defensible changed-conclusion claim is therefore about **reliability rankings
differing from accuracy rankings** — e.g. two models with equal accuracy being
right 95% versus 72% of the time when they report 90% confidence.

## Why this should merge with the conformal probe

**Outcome update 2026-07-25.** The locked conformal probe is now complete and
its distinctive dispersion signal fails its main hypothesis on every model
tested (detail: svib repo wiki, Conformal-Probe-Results). Therefore the
proper-scoring calibration direction must **not** inherit a method contribution
from equivalence dispersion. It may reuse the verified score artifacts and
abstention baselines, but it needs an independent prior-art gate and a
standalone diagnosis/mechanism/fix case. *(That independent gate was then run —
[[Calibration-Prior-Art-Gate]] — and the standalone fix case failed on
rank-invariance. The merged-paper plan below is therefore historical.)*

The two directions share every asset and answer complementary halves of one
question. Calibration asks *is the confidence number correct*; selective
prediction asks *when should the system decline*. Neither has been done for
image-text matching.

Merged, the paper has the shape main tracks accept — with the **consequence**
leg demoted to supporting evidence, since the prior-art correction above shows
compositional risk-coverage is already published:

- **Diagnosis** — matching confidence is uncalibrated, apparently never measured (Gap A). *Verify against the two ICLR/arXiv papers first.*
- **Mechanism** — hard-negative fine-tuning inflates margins faster than accuracy, so descendants are more overconfident than their sources; test via logit-range diagnosis on the five Stage E pairs (Gap B). **This is now the distinctive leg.**
- **Consequence** — reliability rankings differ from accuracy rankings. Cite the existing risk-coverage work rather than claiming the protocol.
- **Fix** — source-matched recalibration restores usable reliability behaviour.

It would also supply the mechanism behind a result the SVIB project measured
independently: a hard-negative-finetuned checkpoint contracting sharply and
falling **below chance** on a swap-attribute split is exactly the signature of
a confidently-wrong, margin-inflated model under surface-form shift. (Numbers:
svib repo wiki, Stage-E pages.)

## Terminology disambiguation required in any writeup

"Calibration" is badly overloaded here and reviewers will pattern-match. One
early paragraph must distinguish proper-scoring calibration from: [CalibCLIP](https://arxiv.org/abs/2510.05586)
(ACM MM'25, *attention* calibration), Contrast-Aware Calibration
(classification), hubness score normalization ([QB-Norm](https://arxiv.org/abs/2112.12777) line), calibrated
recommendations (Steck RecSys'18, genre distribution), conformal coverage, and
the uncertainty-versus-recall curves of the [PCME](https://arxiv.org/abs/2101.05068) line.

## Other risks

- **[Minderer et al.](https://arxiv.org/abs/2106.07998), NeurIPS 2021** ("Revisiting the Calibration of Modern
  Neural Networks") is the standard counterattack on any "modern models are
  miscalibrated" framing. Address it directly.
- [SigLIP](https://arxiv.org/abs/2303.15343)'s learnable bias `b` is structurally a candidate-count prior
  (introduced for batch pos:neg imbalance, never revisited at inference). Cheap
  unclaimed sub-gap, but verify the claim that SigLIP resists temperature
  scaling — the only source found for it is not a real venue.

## Related

[[Calibration-Prior-Art-Gate]] — the gate that closed this direction as framed,
and the three escape routes that carry a changed decision.
[[Calibration-Draw-Preregistration]] — the *other* calibration direction
(calibration-set draw noise floor), gate-failed and scooped.
[[Compression-Audit-Direction]] — where that draw direction was proposed.
[[Next-Direction-Literature-Survey]] — the survey that selected this family.
Conformal-Probe-Preregistration and Conformal-Probe-Results (svib repo wiki) —
the probe this was meant to merge into, and its negative outcome.
Stage-E pages (svib repo wiki) — the five source-matched pairs and per-item
records.

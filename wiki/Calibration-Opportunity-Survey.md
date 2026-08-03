# Confidence Calibration for Image-Text Matching — Opportunity Survey

Status: **Survey completed 2026-07-25; a later check found that the direction
does NOT work as proposed.** *Updated 2026-08-02 for the general research
wiki.* Two independent searches found the same possible gaps. A later gate for
the conformal-probe plan in the svib repo disproved one part of the original
recommendation. Read the correction below before using any idea from this page.

Here, **calibration** means checking whether confidence numbers match real
success rates. If a model says "90% confident" on many examples, it should be
correct about 90% of the time. Measurements such as expected calibration error
(ECE), Brier score, negative log-likelihood (NLL), and reliability diagrams
test this. This page does **not** use "calibration" to mean conformal coverage.

> **Final outcome, read first (2026-08-02).** Two different ideas used the word
> "calibration," and **both are closed**. Do not restart either idea from this
> page without reading its gate.
>
> 1. **This page's idea: calibrate the image-text matching decision with proper
>    scoring rules.** [[Calibration-Prior-Art-Gate]] found that neither exact
>    gap had been claimed, but the plan still failed. Temperature scaling is
>    monotone, meaning it cannot change score order. AURC and risk-coverage
>    curves use only that order. Therefore, the proposed fix leaves both
>    accuracy and risk-coverage results **bit-identical**. It changes exactly
>    zero decisions, while a publishable paper needs a full story: find the
>    problem, explain the cause, fix it, and show a changed decision. That gate
>    also found three papers that now cover parts of Gap A. One of them,
>    [`2607.03143`](https://arxiv.org/abs/2607.03143), was three weeks old when checked and uses the exact
>    calibration protocol for about 400-way retrieval. Only three remaining
>    paths can change decisions: transfer a threshold between datasets, use
>    CLIP as a scorer inside another system, or combine scores across models.
> 2. **The separate calibration-draw idea:** treat the random *calibration data
>    sample* as a source of randomness in LLM compression. That idea lives in
>    [[Compression-Audit-Direction]]. It **FAILED ITS GATE AND WAS ALREADY
>    PUBLISHED; do not run it.** See [[Calibration-Draw-Preregistration]]. ACL
>    2024 published the experiment, and two later studies independently
>    repeated it.
>
> The parts of this page that remain useful are the table showing which areas
> are already heavily studied, the disproved K-mismatch idea, the definitions
> separating different meanings of "calibration," and the warning that
> rank-based measurements do not change under monotone scaling.

## Correction from the earlier-work check (2026-07-25, same day)

The required check for the conformal probe found work that this survey missed:

- **Leveraging Data to Say No: Memory Augmented Plug-and-Play Selective
  Prediction** (ICLR 2026, OpenReview [`wWxdT6LB2D`](https://openreview.net/forum?id=wWxdT6LB2D)) reports AURC and
  risk-coverage results on [SugarCrepe](https://arxiv.org/abs/2306.14610), [Winoground](https://arxiv.org/abs/2204.03162), [What'sUp](https://arxiv.org/abs/2310.19785),
  [VL-Checklist](https://arxiv.org/abs/2207.00221), and [Foil](https://arxiv.org/abs/1705.01359).
- **Look Again Before You Abstain** ([arXiv `2606.16667`](https://arxiv.org/abs/2606.16667), version 4) makes the
  VLM conformal-abstention area even more crowded.

This means the "consequence" part of the combined-paper plan below—measuring
risk and coverage on compositional matching—is **already claimed, not open**.
Gap A and Gap B still appeared open from these papers' reported results because
selective-prediction papers report AURC, not ECE or Brier, and neither paper
reported a source-matched fine-tuning comparison. **That was only an inference,
not a direct check.** Before any run, both papers had to be read in full to
confirm that they included no reliability diagrams, proper-scoring analysis,
or source-matched calibration comparison. Gap A was only *probably* open until
then. [[Calibration-Prior-Art-Gate]] later performed the check.

The search process also failed. Exact arXiv phrase searches returned zero
results because the papers used different words: "selective prediction" plus
benchmark names. [[Next-Direction-Literature-Survey]] records the same problem.
Future checks must search by task and benchmark, not only by a preferred method
term.

## Which areas are already crowded?

| Area | Result | What the search found |
|---|---|---|
| Calibration of CLIP zero-shot **classification** | **ALREADY STUDIED HEAVILY** | LeVine et al. [2303.12748](https://arxiv.org/abs/2303.12748); Tu et al. NeurIPS 2023 ([2402.07410](https://arxiv.org/abs/2402.07410)) and ICML 2024 ([2402.07417](https://arxiv.org/abs/2402.07417)). Shared conclusion: the models are only mildly overconfident, ECE is in the single digits, and one global temperature fixes almost all of it |
| Prompt-tuning calibration | **EXTREMELY CROWDED** | [C-TPT](https://arxiv.org/abs/2403.14119) (ICLR'24) → [DAC](https://arxiv.org/abs/2402.04655) (ICML'24) → [DOR](https://arxiv.org/abs/2410.02681) (ICML'25) → [O-TPT](https://arxiv.org/abs/2503.12096) (CVPR'25) → [D-TPT](https://arxiv.org/abs/2510.09473) → [TCPT](https://arxiv.org/abs/2602.19024) (CVPR'26). Four generations in two years on the same 11 datasets. Do not enter |
| Calibration after the data distribution changes | **ALREADY STUDIED HEAVILY FOR CLASSIFICATION** | [CaRot](https://arxiv.org/abs/2311.01723) (NeurIPS'24) gives a theory bound; [Murugesan et al.](https://arxiv.org/abs/2407.13588) (ECCV'24) show adapters and test-time adaptation (TTA) break it. All are classification studies |
| Calibrated ranking for information retrieval and ads | **ALREADY STUDIED HEAVILY** | KDD'22 → CIKM'23 → KDD'23 → SIGIR'25 → MLPlatt 2026. This is a mature industry area, but it had not been moved to cross-modal matching |
| **Calibration of the image-text MATCHING decision** | **APPEARED OPEN** | See Gap A and the later correction |
| **Hard-negative fine-tuning causing bad calibration** | **APPEARED OPEN** | See Gap B and the later correction |

## The two gaps that originally appeared open

### Gap A: does matching confidence mean what it says?

One search tried seven query forms, and a second search worked independently.
At that time, **neither found a paper computing ECE, Brier, NLL, or reliability
diagrams for a binary image-text matching decision.** No found study on
[SugarCrepe](https://arxiv.org/abs/2306.14610), [ARO](https://arxiv.org/abs/2210.01936), [VALSE](https://arxiv.org/abs/2112.07566), or [Winoground](https://arxiv.org/abs/2204.03162) asked whether the confidence
in the chosen caption was accurate.

For the probability
`p = sigma(logit_scale * (s_pos - s_neg))`, the missing statement was:
"When CLIP reports 0.9 confidence on SugarCrepe, it is correct X% of the time."

The closest paper was only a small side table. Oh et al., *Geodesic Multi-Modal
Mixup* (NeurIPS 2023, [2203.03897](https://arxiv.org/abs/2203.03897)), reports Flickr30k retrieval ECE values:
zero-shot `1.90/1.88`, and simple fine-tuning `2.26/2.00`. It gives three
numbers in a mixup paper, does not define confidence for retrieval, and does
not analyze the result.

Other nearby work measures a different thing:

- Probabilistic embedding papers—[PCME](https://arxiv.org/abs/2101.05068), [PCME++](https://arxiv.org/abs/2305.18171), [ProbVLM](https://arxiv.org/abs/2307.00398), [GroVE](https://arxiv.org/abs/2505.05163), and
  *Post-hoc Probabilistic VLMs* (ICLR'26)—plot uncertainty groups against
  Recall@1. That only checks whether more uncertainty tends to mean worse
  recall. It does not use a proper scoring rule.
- Score-normalization and hubness work—[QB-Norm](https://arxiv.org/abs/2112.12777) (CVPR'22), [DBNorm](https://arxiv.org/abs/2310.11612),
  Sinkhorn, and *Test-Time Distribution Normalization* (NeurIPS'23)—improves
  rank order and reports Recall@K. It does not report ECE.

The later [[Calibration-Prior-Art-Gate]] found that parts of this gap had been
filled and that the small remaining piece was not strong enough as framed.

### Gap B: compare hard-negative fine-tuning with the exact source checkpoint

A **hard negative** is a wrong example designed to look very similar to the
right one. A **source-matched comparison** compares a fine-tuned model with the
exact checkpoint it started from, rather than with a different model.

Source-matched calibration comparisons are normal in VLM research:
*Fine-Tuning is Fine, if Calibrated* (NeurIPS'24, [2409.16223](https://arxiv.org/abs/2409.16223)), CAC
([2501.19060](https://arxiv.org/abs/2501.19060)), [DAC](https://arxiv.org/abs/2402.04655), [DOR](https://arxiv.org/abs/2410.02681), and [CaRot](https://arxiv.org/abs/2311.01723) all use the zero-shot checkpoint
as the starting reference. **All study classification on base and new classes
with prompt tuning or adapters.** None studies contrastive hard-negative
fine-tuning, matching, or retrieval.

No found paper claimed that contrastive hard-negative fine-tuning raises
confidence too much, as measured by ECE against its own source checkpoint. The
second search called this "the most under-claimed link in the whole survey."

There was good reason to expect the effect:

- [SugarCrepe](https://arxiv.org/abs/2306.14610) (NeurIPS'23 D&B) shows that NegCLIP uses artifacts in its hard
  negatives.
- [The Hard Positive Truth](https://arxiv.org/abs/2409.17958) (ECCV'24) measures drops as large as `38.7%` for
  CLIP models fine-tuned with hard negatives when tested with hard positives.
- Oh et al. show that ordinary fine-tuning makes retrieval ECE worse.
- [Murugesan et al.](https://arxiv.org/abs/2407.13588) explain that adaptation hurts calibration by **increasing
  the logit range**. That explanation should transfer directly, but nobody had
  tested it in this setting.

The SVIB Stage E work had already built the needed measurement setup: five
fine-tuned checkpoints, each paired with its exact pretraining source, with
hashes, origin records, reproduced published results, and one record per test
item. This expensive source-matched construction is exactly what Gap B needs.
Full details are in the svib repo wiki's Stage-E pages.

## The K-mismatch idea was wrong

This wrong idea is kept so nobody tries it again. It said that `logit_scale`
was learned for an in-batch classification problem with N choices, then reused
for K choices at test time—K=2 for SCPP++, K=4 for [Winoground](https://arxiv.org/abs/2204.03162), and K=5000 for
COCO retrieval—causing calibration to change with K.

Four facts refuted it:

1. **The proposed mechanism was factually wrong.** CLIP clamps `logit_scale` at
   **100** (`tau = 0.01`), and almost every released checkpoint reaches that
   limit. It is a training-stability limit, not the best value for batch size N.
2. **The theorem already exists.** *softmax is not enough (for sharp size
   generalisation)* (Veličković et al., **ICML 2025**, [2410.01104](https://arxiv.org/abs/2410.01104)) proves that
   softmax spreads probability too thinly as the number of test items grows. It
   proposes changing temperature at inference. K-dependent spreading follows
   directly.
3. **Existing methods already correct candidate-set size.** [Inverted softmax](https://arxiv.org/abs/1702.03859),
   [DSL](https://arxiv.org/abs/2109.04290), [QB-Norm](https://arxiv.org/abs/2112.12777) (CVPR'22), [DBNorm](https://arxiv.org/abs/2310.11612), Sinkhorn normalization, and
   *Test-Time Distribution Normalization* (NeurIPS'23) all normalize over the
   actual choices at test time. Recommender systems have used the logQ
   correction for the training/test normalizer mismatch since 2019.
4. **Published evidence points the other way.** LeVine et al. and Tu et al.
   both find that one temperature transfers across K in `[10, 1000]` and across
   label sets for classification.

One narrower statement remained useful: the temperature used by every later
decision is **a training-stability limit that contains no information about the
later task**. It describes neither the pretraining batch size nor the number of
deployment choices. This can be a secondary measurement only, with the earlier
work stated clearly. It cannot be the headline.

## What a paper in this area would have needed

Both searches found that a paper that only measures calibration is unlikely to
reach a main conference track. The expected form is:
**problem → cause → cheap fix → changed decision**. Pure ECE measurement now
fits a D&B track or the ICML Calibration workshop better.

For a two-choice decision,
`p = sigma(tau * (s_pos - s_neg))` changes monotonically with `tau`. Temperature
scaling therefore **cannot change which caption wins**, so it cannot change
accuracy.

The old survey then incorrectly said that recalibration changes risk-coverage
curves. [[Calibration-Prior-Art-Gate]] corrected this: AURC and risk-coverage
also use only score order, so monotone scaling leaves them bit-identical.
Recalibration can still change **fixed confidence thresholds**, thresholds moved
between datasets, and combined decisions across models. The old example of a
useful reliability difference remains descriptive: two equally accurate models
might be correct 95% versus 72% of the time when each reports 90% confidence.

## Historical plan to merge this with the conformal probe — superseded

The locked conformal probe finished on 2026-07-25. Its special dispersion
(score-spread) signal failed the main hypothesis for every tested model; see
Conformal-Probe-Results in the svib repo wiki. Therefore, this direction could
not borrow a method contribution from equivalence dispersion (score spread
within equivalent items). It could
reuse verified score files and abstention baselines, but needed its own
earlier-work check and a complete problem/cause/fix story.
[[Calibration-Prior-Art-Gate]] then ran that check and found that the fix could
not change rank-based decisions. The merged plan below is kept only as history.

The two ideas shared all data and answered two sides of one question.
Calibration asks whether a confidence number is accurate. Selective prediction
asks when the system should refuse to answer. At the time, neither seemed to
have been studied for image-text matching.

The planned paper structure was:

- **Problem:** matching confidence is wrong and had apparently never been
  measured (Gap A). First verify this against the two ICLR/arXiv papers.
- **Cause:** hard-negative fine-tuning grows score margins faster than it grows
  accuracy. Fine-tuned models are therefore more overconfident than their
  source models. Test this by measuring logit ranges on the five Stage E pairs
  (Gap B). **This became the one distinctive part.**
- **Consequence:** reliability rankings differ from accuracy rankings. Cite the
  existing risk-coverage work rather than claiming that protocol as new.
- **Fix:** source-matched recalibration restores useful confidence behavior.

The proposed cause also matched an independent SVIB result. One hard-negative-
fine-tuned checkpoint became much less stable and fell **below chance** on an
attribute-swap split. That is what a confidently wrong, large-margin model
looks like when wording changes. Exact numbers are in the svib repo wiki's
Stage-E pages.

## Define "calibration" clearly in any future report

The word has many meanings. An early paragraph must separate proper-scoring
probability calibration from [CalibCLIP](https://arxiv.org/abs/2510.05586) (ACM MM'25, which calibrates
*attention*), Contrast-Aware Calibration for classification, score
normalization for hubness such as [QB-Norm](https://arxiv.org/abs/2112.12777), calibrated recommendation lists
(Steck, RecSys'18, about genre mix), conformal coverage, and the uncertainty-
versus-recall plots used by [PCME](https://arxiv.org/abs/2101.05068)-family models.

## Other risks

- **[Minderer et al.](https://arxiv.org/abs/2106.07998), NeurIPS 2021**, *Revisiting the Calibration of Modern
  Neural Networks*, is the standard reply to any broad claim that modern models
  are miscalibrated. Address it directly.
- [SigLIP](https://arxiv.org/abs/2303.15343)'s learned bias `b` acts like a prior for the number of candidates.
  It was added for the positive-to-negative imbalance in a batch and not
  reconsidered at inference. This is a cheap, unclaimed small gap. But first
  verify the claim that SigLIP resists temperature scaling; the only source
  found for it was not a real publication venue.

## Related

[[Calibration-Prior-Art-Gate]] — the check that closed this idea and lists the
three paths that can change decisions.
[[Calibration-Draw-Preregistration]] — the other calibration idea, about random
calibration samples; already published and gate-failed.
[[Compression-Audit-Direction]] — where that second idea began.
[[Next-Direction-Literature-Survey]] — the survey that selected this family.
Conformal-Probe-Preregistration and Conformal-Probe-Results (svib repo wiki) —
the probe this was supposed to join and its negative result.
Stage-E pages (svib repo wiki) — the five source-matched model pairs and
per-example records.

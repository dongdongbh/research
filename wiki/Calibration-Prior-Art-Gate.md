# Calibration Idea — Check Whether Recent Work Already Did It

*Updated 2026-08-02 for the general research wiki.*

Status: **Gate run 2026-07-25, after Conformal-Probe-Results (svib repo wiki)
closed the dispersion (score-spread) direction.** A gate is a check that must
pass before we
trust an idea or spend resources on it. This gate carries out decision #4 from
that results page. It checks the proper-scoring calibration idea on its own.

Here, **proper-scoring calibration** asks whether a model's confidence number
matches how often the model is correct. For example, a model that says "90%"
many times should be right about 90% of those times.

This check used the SVIB project's compositional image-text matching task:
CLIP-family models scored SugarCrepe, NegCLIP, VALSE, and Winoground examples.
The parts that transfer to other projects are the finding about earlier work
and the proof below that recalibration keeps score rankings unchanged. The SVIB
probe results remain in the svib repo.

**Verdict: nobody has made either exact claim before, but the idea does not work
in its current form. The proposed "fix" cannot change the reported decisions.**

## The fact that closes this direction

Recalibrating a model after training cannot change any decision that this study
planned to report.

- Temperature scaling changes how confident a model is in a smooth, one-way
  order. It is **monotone**, meaning that it keeps the scores in the same order.
  For a choice between two captions, it therefore cannot change which caption
  wins. Accuracy stays exactly the same. This was already recorded in
  [[Calibration-Opportunity-Survey]].
- **AURC (area under the risk-coverage curve) and risk-coverage curves use only
  the order of the confidence scores.** Any monotone recalibration keeps that
  order, so the selective-prediction results also stay **bit-identical**.

The nearby research uses these two groups of measurements. Neither group can
change under the only fix that a calibration paper would offer. A paper that
only measures calibration and applies temperature scaling would therefore
change exactly zero decisions. Earlier surveys found that publication normally
requires a complete story: find the problem, explain why it happens, fix it,
and show that the fix changes a decision.

## Required direct checks — both passed

These checks used the full papers, not search-result snippets.

| Paper | ECE | Brier | NLL | Reliability diagram | Source-matched calibration |
|---|---:|---:|---:|---:|---|
| Leveraging Data to Say No (ICLR 2026, [arXiv `2601.22570`](https://arxiv.org/abs/2601.22570)) | 0 | 0 | 0 | 0 | No |
| Look Again Before You Abstain / BCEA ([arXiv `2606.16667`](https://arxiv.org/abs/2606.16667) v4) | 0 | 0 | 0 | 0 | No |

ECE, Brier score, and NLL are different ways to measure whether confidence
numbers match real outcomes. A reliability diagram shows the same match in a
plot.

PaPSP uses the word "calibration" in a **geometric** sense, not a probability
sense. It studies how distances to examples with the same meaning change with
their location in the embedding space. It handles this with contrastive score
normalization. Table B.4 compares PaPSP-FT with unfine-tuned MA-PaPSP. That is a
comparison between methods, not a comparison between a fine-tuned model and the
exact source model it came from. BCEA uses forms of "calibrat*" about 40 times,
but every use refers to split-conformal methods. Neither paper studies either
gap below.

The OpenReview ID saved earlier ([`wWxdT6LB2D`](https://openreview.net/forum?id=wWxdT6LB2D)) is different from the
publicly indexed ID ([`2OcklgJiU0`](https://openreview.net/forum?id=2OcklgJiU0)). The arXiv camera-ready version
confirms that both IDs point to the same work.

## What remains open

### Gap A: confidence calibration for image-text matching

Status: **STILL OPEN, but much narrower.** Many full-text searches found zero
matches: SugarCrepe + ECE, SugarCrepe + reliability diagram, NegCLIP + ECE,
NegCLIP + overconfident, VALSE + ECE, and Winoground + reliability diagram.
However, three papers now cover important pieces:

- **Text as Partial Constraint** ([arXiv `2607.03143`](https://arxiv.org/abs/2607.03143), 3 Jul 2026 — *three weeks
  old*) publishes the exact protocol. It applies softmax to CLIP similarities
  over a standard candidate set. It then reports ECE and NLL for whether the
  top choice is correct, plus Brier scores and risk-coverage curves. It studies
  about 400-way retrieval on COCO and Flickr30K, not a two-caption pair, and it
  is a method paper. **Check it again before submitting anything, because it
  may expand to compositional benchmarks.**
- **DeCC** (EMNLP 2024, [`2407.07840`](https://arxiv.org/abs/2407.07840)) reports Brier scores on Winoground for
  generative vision-language models (VLMs).
- **PaPSP** already reports AURC on exactly our image-text-matching benchmark
  set.

The unanswered piece is now only this exact setting: a binary decision,
compositional hard negatives, and a contrastive scorer. A skeptical reviewer
can reasonably call that a change of dataset and model type on Text as Partial
Constraint's protocol.

### Gap B: compare hard-negative fine-tuning with its own source model

Status: **STILL OPEN and stronger, but the general story is already occupied.**
Nobody has measured this exact effect. Four full-text searches found nothing,
and direct checks of [FSC-CLIP](https://arxiv.org/abs/2410.05210), [CLIC](https://arxiv.org/abs/2505.24424), and
[TripletCLIP](https://arxiv.org/abs/2411.02545) also found nothing. FSC-CLIP's phrase "Selective Calibrated
Regularization" is a wording trap: it means focal loss plus label smoothing.
It reports `ECE` = 0 and `Brier` = 0.

But the broader story — "fine-tuning CLIP makes calibration worse than in its
zero-shot source model, and here is a fix" — is already the subject of ICML
2024 Open-Vocabulary Calibration ([`2402.04655`](https://arxiv.org/abs/2402.04655)) and NeurIPS 2024 DOR
([`2410.02681`](https://arxiv.org/abs/2410.02681)). Only the task and the cause would be new, not the type of
observation.

## Places where recalibration could change a decision

These options are ranked from strongest to weakest. All three avoid the problem
that monotone scaling keeps rankings unchanged.

1. **Move a threshold between datasets.** PaPSP and BCEA both need a labeled
   calibration set from the same data distribution. A threshold chosen on
   dataset A and used on dataset B can change decisions. Monotone rescaling is
   no longer useless because the fixed threshold lands at a different place in
   the new distribution. Possible claim: *fine-tuned checkpoints break
   threshold transfer that their source checkpoints support.*
2. **Use CLIP as a scorer inside another system.** NegCLIP-family models are
   used to filter data, rerank captions, and provide reward scores. If they are
   too confident, a filter with a fixed confidence cutoff accepts more bad
   image-text pairs even when top-1 accuracy improved. This turns a calibration
   number into a dataset-quality number. It also connects to Stage E: a model
   that confidently rejects valid rewrites is a filter that regularly throws
   away good data. Full details are in the svib repo wiki's Stage-E pages.
3. **Combine or route across models.** Combining a contrastive scorer with a
   large vision-language model (LVLM) requires probabilities that mean the same
   thing across models. Recalibration can change a fused decision because
   ranking inside one model does not protect rankings across several models.

## Limits of this gate

- OpenAlex does not contain every paper's full text, and arXiv `all:` does not
  search full text. Zero results are strong evidence, but not proof that
  nothing exists.
- OpenReview was blocked by Cloudflare, so 2026 papers still under review are a
  blind spot.
- [`2607.03143`](https://arxiv.org/abs/2607.03143) was only three weeks old. Check it again before any submission.

## Related

[[Calibration-Opportunity-Survey]] — the idea checked here.
[[Home]] — the standing rule that requires checking earlier work before any run
(rule 1); first recorded as Stage-E-Prior-Art-Audit in the svib repo wiki.
Conformal-Probe-Results (svib repo wiki) — the negative result that started this
gate.

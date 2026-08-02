# Calibration Hypothesis — Prior-Art Gate

*Updated 2026-08-02 for the general research wiki.*

Status: **Gate run 2026-07-25, after Conformal-Probe-Results (svib repo wiki)
closed the dispersion direction.** Executes decision #4 of that page: evaluate
the proper-scoring calibration hypothesis on its own merits with its own gate.

Substrate note: the gate was run against the SVIB project's compositional
image-text matching setting (CLIP-family scorers on SugarCrepe/NegCLIP/VALSE/
Winoground). The transferable content is the prior-art verdict and the
rank-invariance argument below; the project-side probe results live in the
svib repo.

**Verdict: both gaps survive as literally unclaimed, but the direction is not
viable as framed, because the "fix" step is structurally unavailable.**

## The finding that decides it

Post-hoc recalibration cannot change any decision this direction would report.

- Temperature scaling is **monotone**, so on a 2-way caption decision it cannot
  change which caption wins. Accuracy is invariant. (Recorded previously in
  [[Calibration-Opportunity-Survey]].)
- **AURC and risk-coverage curves depend only on the *ranking* of confidences.**
  Any monotone recalibration therefore leaves the selective-prediction
  evaluation **bit-identical** as well.

Both metric families used by the adjacent literature are invariant to the only
fix a calibration paper would offer. A measurement-plus-temperature-scaling
paper here has a decision delta of exactly zero, against a publication bar that
both earlier surveys recorded as requiring
diagnosis → mechanism → fix → changed decision.

## Mandatory direct checks — both cleared

Verified by full-text extraction, not search snippets.

| Paper | ECE | Brier | NLL | Reliability diagram | Source-matched calibration |
|---|---:|---:|---:|---:|---|
| Leveraging Data to Say No (ICLR 2026, arXiv `2601.22570`) | 0 | 0 | 0 | 0 | No |
| Look Again Before You Abstain / BCEA (arXiv `2606.16667` v4) | 0 | 0 | 0 | 0 | No |

PaPSP's "calibration" is **geometric, not probabilistic** — distances to
semantically equivalent examples varying by embedding location, addressed with
contrastive score normalization. Its Table B.4 compares PaPSP-FT against
unfine-tuned MA-PaPSP, which is a method comparison, not a fine-tuned-versus-
its-own-source calibration comparison. BCEA's ~40 uses of "calibrat*" are all
split-conformal. Neither touches either gap.

Note: the OpenReview ID recorded earlier (`wWxdT6LB2D`) differs from the
publicly indexed one (`2OcklgJiU0`); the arXiv camera-ready confirms it is the
same work.

## Gap verdicts

**Gap A (proper-scoring calibration of the matching decision): STILL OPEN,
badly narrowed.** Zero hits across many full-text queries — SugarCrepe + ECE,
SugarCrepe + reliability diagram, NegCLIP + ECE, NegCLIP + overconfident,
VALSE + ECE, Winoground + reliability diagram. But three papers now own pieces:

- **Text as Partial Constraint** (arXiv `2607.03143`, 3 Jul 2026 — *three weeks
  old*) publishes the exact protocol: softmax CLIP similarities over a
  standardized candidate set, then ECE/NLL on top-1 correctness, Brier, and
  risk-coverage curves. It is ~400-way retrieval on COCO/Flickr30K, not the
  2-way caption pair, and it is a method paper. **Re-check before any
  submission in case it expands to compositional benchmarks.**
- **DeCC** (EMNLP 2024, `2407.07840`) reports Brier on Winoground, for
  generative VLMs.
- **PaPSP** owns AURC on exactly our ITM benchmark set.

What remains is "binary decision + compositional hard negatives + contrastive
scorer," which a hostile reviewer reads as a substrate swap on TPC's protocol.

**Gap B (source-matched calibration of hard-negative fine-tuning): STILL OPEN,
stronger, narrative shape occupied.** Nobody has measured it — confirmed by
four zero-hit full-text queries plus direct inspection of FSC-CLIP, CLIC, and
TripletCLIP. FSC-CLIP's "Selective Calibrated Regularization" is a vocabulary
trap: focal loss plus label smoothing, `ECE` = 0, `Brier` = 0.

But "fine-tuning CLIP miscalibrates it relative to its zero-shot source, here
is a fix" is precisely ICML 2024 Open-Vocabulary Calibration (`2402.04655`) and
NeurIPS 2024 DOR (`2410.02681`). Novelty is confined to substrate and
mechanism, not observation type.

## Where a changed decision would actually exist

Ranked by strength. All three escape the rank-invariance problem.

1. **Threshold transfer.** PaPSP and BCEA both require a labeled in-domain
   calibration set. A threshold calibrated on dataset A and deployed on B
   *does* change decisions, and monotone rescaling is no longer a no-op because
   the threshold moves across the distribution. Claim shape: *fine-tuned
   checkpoints break threshold transfer that their source checkpoints support.*
2. **CLIP-as-scorer downstream.** NegCLIP-family models are used as data
   filters, caption rerankers, and reward signals. If systematically
   overconfident, a fixed-confidence filter admits measurably more bad pairs
   *even where top-1 accuracy improved*. This converts a calibration number
   into a corpus-quality number, and it connects to the project's own Stage E
   result — a model that confidently rejects valid rewrites is a filter that
   systematically discards good data. (Full detail: svib repo wiki, Stage-E
   pages.)
3. **Cross-model fusion/routing.** Combining a contrastive scorer with an LVLM
   needs probabilities comparable across models; recalibration is not
   rank-preserving across the fused decision.

## Gate caveats

- OpenAlex full-text coverage is incomplete and arXiv `all:` does not index
  full text, so zero-counts are strong but not absolute.
- OpenReview was unreachable (Cloudflare), so 2026 submissions under review are
  a blind spot.
- `2607.03143` is three weeks old and must be re-checked before submission.

## Related

[[Calibration-Opportunity-Survey]] — the hypothesis being gated.
[[Home]] — the standing rule requiring a prior-art gate before any run (rule 1);
originally recorded as Stage-E-Prior-Art-Audit (svib repo wiki).
Conformal-Probe-Results (svib repo wiki) — the negative result that triggered
this gate.

# Pre-registration: Is One-Step Generative Diversity Collapse Intrinsic, or an Averaging Artifact?

Status: **DRAFT v1, 2026-08-03 — for professor sign-off.** Locks after the
week-1 substrate gate (§8). Target venue: **CVPR 2027** (deadline ~Nov 13,
2026 per our deadlines table — confirm exact date at lock).

Paper type: **diagnostic arbitration with a named method-unlock** (standing
rules 5–6). Sibling pages: [[Prereg-RoboJudge-Audit]] ·
[[Prereg-Crop-Consistency-Distillation]] · [[Prereg-Epistemic-Contextualization]].

---

## 1. The problem, in plain language

One-step ("1-NFE") image generators — MeanFlow, Shortcut models, and now
Drifting — produce a full image in a single network call instead of dozens,
making generation ~50× cheaper. Their headline quality (FID) now rivals
multi-step diffusion. But several groups report they are **less diverse**:
they cover fewer of the modes the data contains.

Two explanations are on the table and they demand different fixes:
- **Intrinsic:** any single network call must commit at once; diversity loss
  is the price of 1-NFE, and no objective change removes it.
- **Averaging artifact:** the dominant 1-NFE training objectives are
  MSE-on-average-velocity; MSE makes the model learn a *frequency-weighted
  mean over sub-modes* ("averaging distortion"), so the collapse is specific
  to that objective family and a non-averaging objective escapes it.

Nobody has run the experiment that separates them.

## 2. Current research state

- **SubFlow** (arXiv 2604.12273, Apr 2026) named the mechanism — "when
  trained with MSE objectives, class-conditional flows learn a
  frequency-weighted mean over intra-class sub-modes" — and fixed it with
  sub-mode conditioning. **But it only tested the averaging family**
  (MeanFlow, Shortcut). Its code is announced, not released.
- **Drifting** (He group, arXiv 2602.04770) reaches 1-NFE **without an
  averaging objective** (it "evolves the pushforward distribution during
  training"). It is the natural falsifier — and it has never been measured
  for diversity. Decisive substrate fact (verified Aug 2026): the released
  `inference.py` in `lambertae/drifting` (485★) **already computes FID, IS,
  precision, and recall — the paper reports only FID/IS. Recall is
  computable today and unreported.**
- Matched averaging-family substrate: `kvfrans/shortcut-models` (762★)
  releases ImageNet-256 weights with **native 1/4/128-step sampling from the
  same checkpoint**; MeanFlow/iMF/pMF weights on HF.
- The crowded periphery is orthogonal: distillation-side diversity fixes
  (1.x-Distill, Diversity-Preserved DMD, Data-Forcing, Don't-Settle-at-the-
  Mode) all live *inside* the averaging/distillation family. The
  **cross-family arbitration is unrun** (lane sweep, Aug 2026).

## 3. Our method and novelty

**The within-model NFE ladder.** A naive cross-family recall comparison is
meaningless (Drifting FID 1.53 vs Shortcut 1-step FID 10.6 — quality
confound). Instead, each family is its own control:

- For families with a native NFE ladder (Shortcut 1/4/128 from one
  checkpoint; MeanFlow-family variants), the primary endpoint is the
  **slope of recall vs NFE at matched precision**.
- Drifting (natively 1-NFE) is scored against a precision-matched multi-step
  reference; its recall *deficit* is compared to the averaging family's
  1-vs-128-step deficit.
- A **matched-budget from-scratch pair** (averaging vs drifting objective,
  B/4 scale, ImageNet-64/CIFAR) controls capacity, data and compute — the
  released checkpoints differ in all three.

Predictions separate cleanly: if collapse is **intrinsic to 1-NFE**, recall
falls toward 1-NFE in *every* family including Drifting; if it is
**averaging-specific**, the drop is family-specific and Drifting retains
recall at 1-NFE. Neither outcome is a null.

Novelty (verified): first cross-family 1-NFE diversity measurement; first
diversity numbers for Drifting at all; first test of SubFlow's mechanism
outside the family it was derived on.

## 4. Pre-registered design

**Substrates:** released checkpoints — Drifting latent/pixel B & L
(`Goodeat/drifting`), Shortcut DiT-B/XL, MeanFlow/iMF/pMF (HF weights) — plus
a many-step reference (standard flow/diffusion at matched arch) for the
precision-matched recall ceiling; plus the from-scratch B/4 pair.

**Metrics:** precision/recall (Kynkäänniemi k-NN, k pre-fixed), coverage &
density, per-class recall on ImageNet-256 (class-conditional sub-mode
coverage is where averaging distortion bites), and a memorization check
(nearest-train-neighbor distance) so "diverse" is not "copying." All at
matched precision via guidance/temperature sweeps, matched sample count
(50k), fixed seeds.

**Hypotheses (directional, locked):**
- **H1:** recall decreases monotonically as NFE→1 within the averaging
  family (confirms SubFlow's premise on its own turf).
- **H2 (decisive):** Drifting's 1-NFE recall deficit vs its precision-matched
  multi-step reference is **less than half** the averaging family's
  1-vs-128 deficit (predict TRUE = averaging-specific).
- **H3:** the from-scratch matched pair reproduces the H2 ordering (predict
  TRUE; if FALSE, checkpoint confounds drove H2 — the honest downgrade).
- **H4:** SubFlow-style sub-mode conditioning (reimplemented if their code
  stays unreleased — scoped to the *published* recipe only) narrows the
  averaging family's deficit but does not change Drifting's (predict TRUE).

**Decision rules:** bootstrap CIs over sample splits; Holm across H1–H4;
precision-matching tolerance pre-fixed (±0.02). H4 is dropped without
penalty if reimplementation cannot reproduce SubFlow's published headline
within tolerance (substrate-liveness rule — we do not publish nulls on our
own reimplementation; H4 is confirmatory-only).

**What we will NOT claim:** anything about text-to-image or video; anything
about distillation-based one-step methods (different mechanism family);
that any specific released model is "bad" — the object is the objective
class, not the checkpoint.

## 5. Expected outcomes

- **Averaging-specific (H2 TRUE):** headline — "1-NFE does not force
  diversity collapse; the objective does." Validates the non-averaging
  route; method-unlock = objective-level guidance for one-step training,
  and SubFlow's fix is scoped to where it is needed.
- **Intrinsic (H2 FALSE):** headline — "one call, fewer modes: the
  diversity cost of 1-NFE is objective-independent." Method-unlock = an
  NFE-aware diversity budget (how many steps buy how much recall) and the
  recommendation that coverage-critical applications not default to 1-NFE.
- Artifact either way: the cross-family diversity harness (matched-precision
  P/R/coverage/memorization protocol) — the measuring stick this lane lacks.

## 6. Resources and timeline

**Cost:** 150–350 GPU-h. Inference sweeps on OrangeGrid A100; the
from-scratch pair (~200 GPU-h) on Anvil A100 or Delta H200. **JAX-on-H200
must be verified before relying on Delta** (recorded hardware lesson).
Timeline to CVPR (~14 wks): Wk1 gate → Wk2–4 checkpoint sweeps → Wk5–8
from-scratch pair → Wk9–10 H4 → Wk11–13 analysis/writing.

## 7. Risks and scoop watch

- SubFlow releasing code + cross-family results is the main scoop path —
  watch `2604.12273` versions; 6-week re-gate clock.
- The He group could self-measure Drifting's recall (it is their
  `inference.py`) — mitigated by speed: the first sweep is days of work.
- Precision-matching may be impossible for some checkpoint pairs → report
  the reachable precision range honestly; the from-scratch pair is the
  clean fallback.

## 8. Week-1 gate (locks the prereg)

1. Run Drifting + Shortcut recall once end-to-end (the "recall is
   computable today" claim, verified in our hands).
2. JAX stack check on H200 and A100.
3. Confirmatory literature pass, most recent 8 weeks explicit.
4. Confirm CVPR 2027 exact deadline. Professor sign-off → LOCKED + git hash.

## Related

[[Unified-Direction-Ranking-2026-08]] · the removed SigLIP-2 ladder draft (git history) ·
[[GPU-Resources-Across-Clusters]]

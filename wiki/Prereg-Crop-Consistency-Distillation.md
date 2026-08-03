# Pre-registration: Crop-Consistency Distillation — Region Information at Single-Pass Cost

Status: **LOCK HOLD, 2026-08-02.** The confirmatory literature pass found
**[arXiv 2604.11496](https://arxiv.org/abs/2604.11496)** (Apr 2026, 3.5/4-axis match): its TF_Local publishes
the headline insight (fine-grained alignment over FROZEN patch/token
features → large compositional gains, retrieval preserved; [SugarCrepe](https://arxiv.org/abs/2306.14610)
73.0→86.3) and its §3 diagnostic is our crop teacher. Surviving delta:
**distillation into ROI-pooled patch features that preserves ~1.1×
DUAL-ENCODER inference with cacheable embeddings** — TF_Local is a per-pair
cross-encoder. Also: **[FineCLIP](https://openreview.net/forum?id=nExI4FuKWD) (NeurIPS 2024)** shows region
self-distillation helps compositionality when the backbone trains → the A4
conclusion must be scoped to "[CLIPSelf](https://arxiv.org/abs/2310.01403)'s released checkpoint, frozen ITM."
**Re-gate verdict (2026-08-02): SURVIVES-NARROWED, level 2/5, ★★.** The
exact cell (crop/cross-encoder teacher → ROI-pooled frozen-backbone student,
compositional benchmarks, efficiency frontier) is empty, but every
ingredient is owned: DCLIP ([2505.21549](https://arxiv.org/abs/2505.21549)) publishes the efficiency thesis
verbatim in the retrieval-only cell ("without requiring region processing
at inference"); [CPRD](https://arxiv.org/abs/2407.07479) (CVPR 2024) owns cross→bi VLM distillation; CLIPSelf/
[DeCLIP](https://arxiv.org/abs/2505.04410) own the crop-teacher→ROI mechanism; [CLIC](https://arxiv.org/abs/2505.24424) owns retrieval
preservation. Reviewer shape: "DCLIP + [SugarCrepe](https://arxiv.org/abs/2306.14610)." TF_Local code and
BiSCoR-Ctrl are UNRELEASED (repo is a "Coming soon" stub, 0 citations) →
the upper anchor must be re-implemented. 2604.11496 never discusses cost/
cacheability — the whitespace is real but reads as an obvious follow-up,
and concurrent NeurIPS'26/ICLR'27 submissions in this cell are invisible
by construction. ★★½ only if: ≥10× cost advantage at ≥70% gain retention,
paired CIs on ≥2 benchmarks, end-to-end latency (not FLOPs). ★½ if the
TF_Local re-implementation fails to reproduce.
**Decision at sign-off: proceed-reframed (★★ shape) vs bench the method**
(A4/A5 results then fold into the SVIB write-up narrative). Prior status: DRAFT v1 for
professor sign-off; week-1 decisive checks complete (§8). Target venue:
**ICLR 2027** (abstracts Sep 18) — deadline unchanged, reframe cost is
wording + one added baseline, not new experiments for the abstract.

Paper type: **METHOD** (owner definition: a new approach — here, a training
procedure that breaks the assumption that patch tokens already carry region
information). Gate record: [[Method-Gates-2026-08]]. Companion diagnostic:
[[Prereg-RoboJudge-Audit]].

---

## 1. The problem, in plain language

Contrastive VLMs are compositionally brittle. Our SVIB post-mortem measured
exactly where the fix lives and what it costs: re-encoding image **regions at
full resolution** (a 20-crop grid + one self-attention layer) buys the gains
(+2.66 [SugarCrepe++](https://arxiv.org/abs/2406.11171) on corrected CLIP) but at **~8× inference cost**; reusing
the ViT's own ROI-pooled patch tokens is nearly free (1.06×) but **loses 1.32
points** (paired 95% CI [−2.51, −0.12]). The region information is simply not
in the patch tokens — and putting it there at training time, so inference
needs one pass, is the method.

## 2. Current research state (gated 2026-08-03)

- **[CLIPSelf](https://arxiv.org/abs/2310.01403) (ICLR 2024)** owns the ancestor mechanism — aligning dense-map
  region representations with the image-level embedding of the corresponding
  crop — but for **open-vocabulary dense prediction only**, with full ViT
  fine-tuning and **no compositional/ITM evaluation**; none of its 100
  citing papers applies it there. Repo public (207★, frozen Feb 2024, **no
  license** — evaluate the checkpoint, reimplement the mechanism).
- **DeGLA ([2504.16801](https://arxiv.org/abs/2504.16801))** improves compositionality at ~1× via a *different*
  mechanism (global EMA-teacher preservation + LLM-generated hard negatives;
  +3.5% avg [VALSE](https://arxiv.org/abs/2112.07566)/SugarCrepe/[ARO](https://arxiv.org/abs/2210.01936)) → **mandatory baseline**, not prior art on
  our mechanism.
- **The aggregation line** (LABCLIP, [DCSM ICCV 2025](https://arxiv.org/abs/2503.08723), ["Similarity Is Not
  Logic"](https://arxiv.org/abs/2607.23052) ICML 2026) argues binding failure is *execution not representation*.
  Our −1.32 evidence concerns region information for grid+attention gains,
  not only binding — but reviewers will conflate them, so the aggregation
  fix is a pre-registered **arm and kill criterion**, not a citation.
- **Wave-2 gate addendum (2026-08-02, [[Method-Gates-Wave-2-2026-08]]):**
  "LABCLIP" identified as **[arXiv 2502.03566](https://arxiv.org/abs/2502.03566) (ICLR 2026)** — a D×D
  text-side matrix on *frozen* encoders (~590K params, shuffled-negative
  training, ARO+[SugarCrepe](https://arxiv.org/abs/2306.14610)). It and DCSM are the published **concrete
  instantiations of A5** (use them, not a home-built aggregation fix) and
  both become named baseline rows. The gate's neighborhood read supports
  §2's positioning: every 1× competitor (DCSM, LABCLIP, [ABE-CLIP](https://arxiv.org/abs/2512.17178), TF-Local)
  reads out existing frozen features; none injects a stronger teacher
  signal into the patch path.
- [SILC](https://arxiv.org/abs/2310.13355) / [SigLIP-2](https://arxiv.org/abs/2502.14786) self-distillation is pretraining-time local-to-global on
  the training pipeline's own crops — different direction, setting, and
  objective from post-hoc crop→patch distillation on a frozen model.

## 3. The method

**Teacher (frozen, training-time only):** the full-resolution crop pathway —
frozen backbone applied to the 20-view grid, plus our *trained
grid+self-attention head* — i.e., a structured multi-region teacher, not
single-crop embeddings (this is the delta from [CLIPSelf](https://arxiv.org/abs/2310.01403)'s teacher).
**Student:** ROI-pooled patch tokens of the same frozen backbone, passed
through a **light adapter** (primary: adapter after the frozen ViT;
secondary: LoRA on the last N blocks).
**Losses:** per-region feature matching (cosine) + a relational term
matching the teacher head's inter-region attention pattern (transfers the
dense-routing structure our interventions showed is load-bearing).
**Data:** images only (COCO/VG; our cached crops) — the distillation is
self-supervised; no captions, no annotations.
**At inference:** one backbone pass + adapter ≈ **1.1×** cost; evaluation
under our corrected, validation-locked ITM protocol.

## 4. Pre-registered design

**Arms** (CLIP ViT-B/32 primary; SigLIP2 B/16 secondary):
- A0 raw frozen backbone (corrected baselines).
- A1 patch-ROI, no distillation (lower anchor, −1.32).
- A2 grid+self-attention @8× (upper anchor, +2.66).
- A3 **ours**: distilled adapter @~1.1×.
- A4 [CLIPSelf](https://arxiv.org/abs/2310.01403) released checkpoint, ROI-pooled, same protocol (**week-1
  decisive check**).
- A5 aggregation-fix over raw patch tokens @1× (kill-arm).
- A6 DeGLA (published numbers; checkpoint if released).
- Retrieval-preservation eval (COCO both directions) for A3.

**Hypotheses (directional, locked):**
- **H1:** A3 recovers ≥⅔ of the (A2−A1) gap on [SugarCrepe++](https://arxiv.org/abs/2406.11171) under
  validation-locked selection (predict TRUE).
- **H2:** A3 − A0 ≥ +1.0 SCPP++ with paired CI excluding 0 (predict TRUE).
- **H3:** A4 does NOT close the gap (predict TRUE — CLIPSelf's objective is
  dense-prediction alignment, not ITM).
- **H4:** A5 alone does NOT close the gap (predict TRUE; FALSE = the
  representation was sufficient and the method is unnecessary → §kill).
- **H5:** A3 preserves COCO retrieval within 0.5 R@1 both directions.
- **H6:** A3 inference overhead ≤1.2× measured (same L40S protocol as SVIB).

**Decision rules:** 3 seeds; paired bootstrap CIs (existing machinery);
validation-locked α/config selection only; Holm across H1–H6.
**Hyperparameter honesty:** an attempt budget of **6 training configs
total** (loss weights × adapter size), fixed now; no post-hoc mining; all
attempts reported.

**Kill criteria:** (i) week-1: A4 already closes ≥⅔ of the gap → the paper
collapses into a calibration of CLIPSelf (drop to bench; salvage = transfer
study). (ii) A5 closes ≥⅔ of the gap → the fix is execution, not
representation; fold our evidence into the readout-ladder paper instead.
(iii) A3 fails to beat A1 beyond seed noise within the attempt budget →
publish inside the honest-negative section of the SVIB write-up, not alone.

**What we will NOT claim:** SOTA compositionality versus hard-negative
fine-tuned models (different regime — we claim the frozen-backbone,
annotation-free-at-inference lane); anything about generative/decoder VLMs;
backbones beyond the two tested.

## 5. Expected outcomes

- **Central (H1–H4 as predicted):** compositional gains at single-pass cost
  — a deployable method converting SVIB's honest negative into its fix, with
  the relational-distillation term as the mechanism novelty.
- **A5-wins branch:** the field learns the representation was sufficient
  after all — merged into the readout-arbitration line, still publishable.
- Artifacts regardless: adapter weights, distillation recipe, and the
  harness — on top of the already-released SVIB evaluation stack.

## 6. Resources and timeline

**Cost:** 200–400 GPU-h (adapter training is small; teacher features are
already cached for CLIP). OrangeGrid A100 primary; Anvil for the SigLIP2
arm. Wk1: A4 + A5 decisive checks, lock. Wk2–3: A3 training sweep (attempt
budget). Wk4: secondary backbone + retrieval. Wk5: seeds/CIs. Wk6:
analysis. Wk7: write-up. Abstract needs A3-vs-A1 on the primary backbone.

## 7. Risks and scoop watch

- Aggregation-line groups moving training-side is the live scoop path —
  watch LABCLIP/DCSM/SNL citations; standard 6-week re-gate; 48h re-gate on
  any "crop distillation compositional" hit.
- [CLIPSelf](https://arxiv.org/abs/2310.01403) has no license → its checkpoint is evaluated, never forked; our
  implementation is from the paper.
- Teacher ceiling: A2 is only +2.66 — effect sizes are small, hence the
  paired-CI machinery and the ≥⅔-gap framing rather than absolute SOTA.

## 8. Lock checklist

1. Professor sign-off on §4 (esp. attempt budget and kill criteria).
2. ~~Week-1 A4/A5 results in hand.~~ **DONE 2026-08-02, both as predicted,
   neither kill criterion fires** (cropdistill repo, results/a4 + results/a5,
   manifested + same-day wiki records):
   - **A4/H3 TRUE, kill (i) cleared decisively:** [CLIPSelf](https://arxiv.org/abs/2310.01403) ROI-pooled lands
     8–10 pts BELOW the base patch arm on every seed (locked A4−A1 = −12.8,
     CI [−14.1, −11.5]); its global endpoint collapses to 42.7 vs 68.6 —
     the dense-prediction objective destroys compositional ITM rather than
     injecting region information.
   - **A5/H4 TRUE, kill (ii) cleared:** LABCLIP closure vs the (A2−A1) gap
     is negative on all three seeds (−0.97/−0.32/−0.18) while the same
     matrices replicate the paper's +5.2 [SugarCrepe](https://arxiv.org/abs/2306.14610) gain (reimplementation
     validated; null localized to the strict SCPP++ protocol). α=1 arm
     reproduction check: 5.1e-7.
3. **PIN BEFORE LOCK (from the A4/A5 runs):**
   (a) A5 closure-anchor choice — A0- vs A1-anchored ratio (both computed;
   pin one). (b) A4 conclusion worded as "A4 ≪ A1", not a closure ratio —
   [EVA02](https://arxiv.org/abs/2303.11331)'s native crop-vs-patch gap is small/noisy (one seed exactly 0).
   (c) A5 leakage deviation from the published recipe recorded: 19,006
   Karpathy-train rows excluded.
4. Confirmatory literature pass, most recent 8 weeks explicit (incl.
   OpenReview — tooling installed 2026-08-02).
5. → LOCKED + git hash; deviations logged below this line.

## Related

[[Method-Gates-2026-08]] · [[Unified-Direction-Ranking-2026-08]] ·
[[Prereg-RoboJudge-Audit]] · [[Status-And-Survivors]]

# Pre-registration: Is One-Step Generative Diversity Collapse Intrinsic, or an Averaging Artifact?

Status: **LOCKED 2026-08-08 (professor sign-off; recommended defaults
adopted).** Locked amendments from the week-1/week-2 blocks below:
H1 is restated to matched-precision-only comparisons (the step-count
trend is guidance-confounded, so raw step slopes carry no evidential
weight); the model roster drops the unreleased MeanFlow flagship and
ROMS-IMLE, adds iMF, pMF, IMM and AFM; H2's within-checkpoint control
is IMM's 1/2/4/8-step sampling; **H3 is primarily IMM's single-knob
averaging-vs-distributional contrast (M=1 vs M=4), with the
from-scratch matched pair as the fallback if the knob proves
confounded**; the JAX measurement path is canonical (byte-identical
FID references across families); the caching/loader speedups may be
used only after the bit-for-bit regression check passes; the CVPR
deadline is treated as Nov 2026 ± 2 weeks with a September re-check.
Lock hash: `ad85987d4b5e13c2e59c2afd4aa557fc5178338a`.

**Deviation 1 (2026-08-09, coordinator-approved; OWNER RATIFICATION
PENDING):** the lock's "bit-for-bit" speedup regression requirement is
unattainable on this hardware for ANY run — unmodified week-1 code
drifts by the same ~1e-4 in FID across repeats and machines (measured
three-repeat noise floor: 1e-4–4e-4 FID, ~1e-3 recall — two orders of
magnitude below the ~0.1-scale effects H1/H2 test). Replacement, with
the check strengthened where hardware permits: exact numerical
differences reported against the noise floor, PLUS two truly exact
tests noise cannot reach (cached reference features equal by
np.array_equal; labels-only loader's label stream equal element-wise).
Speedups passed and are cleared (6.18× faster; the sped-up run is
CLOSER to week-1 than the un-sped-up rerun — the signature of noise,
not bias). Also recorded at this gate: the guidance explanation for the
128-step anomaly is CONFIRMED by direct measurement (guided FID 19.57
vs pass rule ≤22); high-step numbers un-quarantined. Correction appended
2026-08-09, superseded same day by isolation experiment X1: the drift
first blamed on GPU co-tenancy is actually a BIMODAL XLA-autotuning
flip — six measurements of one configuration fall into two tight
clusters (within-cluster spread ≤5.4e-5 FID; between-cluster 7.3e-3),
and a run executed ALONE landed in the second cluster, so co-tenancy is
demonstrably not necessary. The noise floor a comparison must beat is
therefore the BETWEEN-cluster figure: FID 7.3e-3, precision 0.0047,
recall 0.0014 (ten times the within-cluster figure first reported). No
conclusion changes: the binding constraint everywhere remains the 0.020
reference-side resolution limit (14× the corrected recall floor), and
H1's shortfalls run 29–106× the corrected floor. The feature-saving
hook was exonerated by the same experiment and the held 50k rows
released. The exclusive-card rule stays as hygiene but is explicitly
NOT sufficient for reproducibility. Also recorded: the naive
with-replacement bootstrap on generated samples is structurally biased
for k-NN recall (measured −0.0895); intervals use reference-side and
paired-difference resampling instead, with the biased variant retained
only as a labeled diagnostic and the finite-generated-sample limitation
stated. Second amendment, same day: one of the two "noise-immune exact
tests" was itself overstated — the reference-feature computation is
subject to the same kernel nondeterminism, so cache-vs-recompute cannot
be bitwise (measured ≤0.0042, no larger than recompute-vs-recompute);
the honest statement is that the cache PINS the reference side, which
improves between-row comparability (shared-seed logic), and the
decision-relevant test is the cached-vs-fresh effect on
precision/recall (G0b, reported whatever it shows). Noise floors are
per-path: the Shortcut measurement path is unimodal (recall floor
0.0004, on which H1's evidence rests at 70–370×); the Drifting path is
bimodal (0.0014). Third amendment (2026-08-10): the week-1 claim
that a 10k recall is a "lower bound" on the 50k value was BACKWARDS —
recall FALLS as the generated sample count rises (k-nearest-neighbour
radii shrink as points densify; measured −0.04 to −0.10, and the
shrinkage is model-dependent, so no uniform correction exists). Rule
now in force and enforced in code: every recall figure carries its
sample count, and no two recalls are compared unless the counts match.
No measured conclusion is affected — all reported contrasts were
already same-count. Changes after this point are deviations and must be logged. Original
draft header follows.

Previous status: DRAFT v1, 2026-08-03. This plan locks
after the week-1 check in §8. Target venue: **CVPR 2027**. The deadline is
around Nov 13, 2026 in our deadlines table; confirm the exact date before lock.

**Week-1 check RESULT (2026-08-06; §8 steps 1–4 done, step 5 = owner):**

1. **The pipeline is ours.** We ran both families end-to-end on our own
   H100 (10,000 samples each, identical class labels and real reference).
   **Drifting's recall — never published anywhere — is 0.72 (latent L) /
   0.69 (pixel L) at 1 step.** Shortcut recall rises with step count
   (0.48 → 0.53 → 0.62 at 1/4/128), the direction H1 predicts. These rows
   are NOT precision-matched, so they say nothing about H2 yet.
2. **JAX works on our H100** (741 TFLOP/s, real GPU compute). Still open:
   A100 on OrangeGrid, H200 on Delta. **Correction to §6: Anvil's A100
   partition is not available to our account at all** — the from-scratch
   pair must use the H100 partition or Delta. Note both repos are JAX.
3. **Novelty holds** (8-week window searched; SubFlow still has no code;
   nobody has measured Drifting's diversity). One softening: ROMS-IMLE
   ([2607.19332](https://arxiv.org/abs/2607.19332)) publishes recall for a
   non-averaging one-step model — "nobody has measured this" must become
   "no one has measured it across objective families." It is also a
   candidate second non-averaging model for H2.
4. **The CVPR 2027 deadline is not yet published** (site 404s; CVPR 2027
   is Jun 20–24, Seattle). Treat as Nov 2026 ± 2 weeks; re-check in Sept.
5. ⚠ **Open anomaly before trusting high-step numbers:** our Shortcut
   128-step FID is 40.1 vs the repo's published 15.5 (reference-stats and
   CFG explanations ruled out; the released checkpoint may not be the one
   behind the README table). Recall comparisons are unaffected. Also: the
   §4 claim that MeanFlow/iMF/pMF weights are "on HF" did not survive a
   first search — verify before weeks 2–4 depend on it.

   Full record: `code/nfe1/runs/week1_check/20260806/`.

**Week-2 preparation RESULT (2026-08-08; lock-relevant — read before locking):**

1. **The 128-step FID anomaly is RESOLVED, and our number was right.**
   The repo's 15.5 is a GUIDED figure — the paper states CFG is used
   only at the smallest step size — while our 40.1 is unguided (DiT-B
   unguided baselines land exactly there; an independent paper reruns
   the same checkpoint guided and gets 15.0). High-step numbers are
   un-quarantined pending one 10-minute regression run.
   **Consequence for H1:** the week-1 recall rise with steps is
   guidance-confounded (guidance is baked into the low-step weights and
   absent at 128); H1 must be read at matched precision only, and its
   wording should say so at lock.
2. **Model roster corrections:** MeanFlow's flagship ImageNet weights
   were never released (§4 must drop that claim); **iMF and pMF weights
   DO exist** ([iMF](https://huggingface.co/Lyy0725/iMF),
   [pMF](https://huggingface.co/Lyy0725/pMF)); Drifting/iMF/pMF share a
   byte-identical FID reference file — the JAX path gives cross-family
   comparability for free. **ROMS-IMLE is out** (no code or weights,
   and its published recall of 0.50 is LOWER than its baselines — the
   week-1 note treating it as a helpful second non-averaging model was
   backwards). **Verified replacements:** [IMM](https://huggingface.co/lumaai/imm)
   (samples at 1/2/4/8 steps from ONE checkpoint — exactly the H2
   control; and its averaging-vs-distributional loss is a single knob,
   a cleaner H3 than training from scratch) and AFM (ByteDance, ships
   50k sample packs so recall costs nothing).
3. Reference-feature caching and the parallel loader are implemented
   and must reproduce week-1 metrics bit-for-bit before any sweep uses
   them (a 15-minute GPU check).

   Ten open lock questions: `code/nfe1/runs/week2_prep/20260808/`.

Paper type: **a study that decides between two explanations, with a named
method it could make possible** (standing rules 5–6). Related plans:
[[Prereg-RoboJudge-Audit]] · [[Prereg-Crop-Consistency-Distillation]] ·
[[Prereg-Epistemic-Contextualization]].

---

## 1. The problem

One-step, or "1-NFE," image generators create a full image with one network
call instead of dozens. Examples include [MeanFlow](https://arxiv.org/abs/2505.13447), [Shortcut models](https://arxiv.org/abs/2410.12557), and now
Drifting. They make generation about 50× cheaper. Their main quality score,
Fréchet Inception Distance (FID), is now close to that of multi-step diffusion
models. However, several groups report that these one-step models are **less
diverse**: they represent fewer of the different patterns in the data.

There are two possible explanations, and each needs a different fix:

- **Intrinsic to one step:** a single network call must make every choice at
  once. Losing diversity is the price of 1-NFE, and no change to the training
  goal can remove it.
- **Caused by averaging:** most 1-NFE methods train with mean squared error
  (MSE) on average velocity. MSE can make the model learn a
  *frequency-weighted average over sub-modes*, which is called averaging
  distortion. Under this explanation, the training goal causes the collapse,
  and a goal that does not average can avoid it.

Nobody has run the experiment that tells these explanations apart.

## 2. What recent work has shown

- **SubFlow** ([arXiv 2604.12273](https://arxiv.org/abs/2604.12273), Apr 2026) named the mechanism. It says
  that "when trained with MSE objectives, class-conditional flows learn a
  frequency-weighted mean over intra-class sub-modes." It uses sub-mode
  conditioning as a fix. **However, it tests only the averaging family:**
  MeanFlow and Shortcut. Its code is announced but not released.
- **Drifting** from the He group ([arXiv 2602.04770](https://arxiv.org/abs/2602.04770)) reaches 1-NFE
  **without an averaging objective**. It "evolves the pushforward distribution
  during training." This makes it the natural model that could disprove the
  averaging explanation, but nobody has measured its diversity. We verified an
  important practical fact in Aug 2026: the released `inference.py` in
  [`lambertae/drifting`](https://github.com/lambertae/drifting) (485★) **already calculates FID, Inception Score
  (IS), precision, and recall. The paper reports only FID and IS. We can
  calculate its unreported recall now.**
- For a matched model from the averaging family,
  [`kvfrans/shortcut-models`](https://github.com/kvfrans/shortcut-models) (762★) releases ImageNet-256 weights that can
  sample in **1, 4, or 128 steps from the same checkpoint**. MeanFlow, iMF, and
  pMF weights are also on Hugging Face (HF).
- Nearby work studies a different question. Diversity fixes for distillation,
  including [1.x-Distill](https://arxiv.org/abs/2604.04018), [Diversity-Preserved DMD](https://arxiv.org/abs/2602.03139),
  [Data-Forcing](https://arxiv.org/abs/2606.18478), and [Don't-Settle-at-the-Mode](https://arxiv.org/abs/2606.27371), stay inside the
  averaging/distillation family. The **comparison across objective families has
  not been run**, based on our Aug 2026 search.

## 3. What is new in our study

**Compare different step counts inside the same model family.** A simple
Drifting-versus-Shortcut recall comparison would be unfair because their
quality is different: Drifting FID is 1.53, while Shortcut 1-step FID is 10.6.
This difference would be a hidden factor. Instead, each family acts as its own
control:

- For a family that supports several numbers of function evaluations (NFE),
  such as Shortcut at 1/4/128 steps from one checkpoint and the MeanFlow-family
  variants, the main measurement is the **slope of recall versus NFE at matched
  precision**.
- Drifting is built for 1-NFE. Compare it with a multi-step reference matched
  for precision. Then compare its recall *shortfall* with the averaging
  family's shortfall from 128 steps to 1 step.
- Train a **matched-budget pair from scratch** at B/4 size on ImageNet-64/CIFAR:
  one uses an averaging goal and one uses the Drifting goal. This keeps model
  capacity, data, and compute the same. The released checkpoints differ in all
  three, so they cannot provide this control.

The predictions are clear. If collapse is **intrinsic to 1-NFE**, recall will
fall toward 1-NFE in every family, including Drifting. If collapse is
**specific to averaging**, only that family will show the drop, while Drifting
keeps recall at 1-NFE. Either answer is useful.

Checked novelty: this is the first cross-family measurement of 1-NFE diversity,
the first diversity result of any kind for Drifting, and the first test of
SubFlow's explanation outside the family on which it was built.

## 4. Exact experiment plan

**Models:** released Drifting latent/pixel B & L checkpoints from
[`Goodeat/drifting`](https://huggingface.co/Goodeat/drifting); Shortcut DiT-B/XL; MeanFlow/iMF/pMF HF weights; a
many-step standard flow or diffusion model with matched architecture as the
precision-matched recall ceiling; and the pair we train from scratch at B/4.

**Measurements:** precision/recall using [Kynkäänniemi k-NN](https://arxiv.org/abs/1904.06991), with k fixed
before the run; coverage and density; recall for each ImageNet-256 class,
because class-conditional coverage of sub-modes is where averaging distortion
should appear; and distance to the nearest training image, so a model does not
look "diverse" merely by copying. Match precision through guidance/temperature
sweeps. Use the same sample count, 50k, and fixed random seeds.

**Predictions, fixed before the run:**

- **H1:** recall decreases steadily as NFE→1 inside the averaging family. This
  confirms SubFlow's starting claim in the family it studied.
- **H2, the deciding test:** Drifting's 1-NFE recall shortfall from its
  precision-matched multi-step reference is **less than half** of the averaging
  family's shortfall from 128 steps to 1 step. We predict TRUE, which supports
  the averaging-specific explanation.
- **H3:** the matched pair trained from scratch has the same ordering as H2. We
  predict TRUE. If FALSE, differences between the released checkpoints caused
  H2, and we must honestly weaken the claim.
- **H4:** SubFlow-style sub-mode conditioning narrows the averaging family's
  shortfall but does not change Drifting's. We predict TRUE. If SubFlow's code
  remains unavailable, reimplement only its published recipe.

**Decision rules:** use bootstrap confidence intervals over sample splits and
Holm correction across H1–H4. Fix the precision-matching tolerance at ±0.02.
Drop H4 without penalty if our implementation cannot reproduce SubFlow's main
published result within tolerance. This is the rule for checking that a test
model is usable: we do not publish a negative result caused by our own failed
reimplementation. H4 is only a confirmation test.

**Claims we will not make:** claims about text-to-image or video; claims about
one-step distillation methods, which use a different mechanism; or claims that
any released model is "bad." We study the class of training goals, not one
checkpoint.

## 5. What each possible result means

- **Averaging-specific, H2 TRUE:** the main message is "1-NFE does not force
  diversity collapse; the objective does." This supports training methods that
  do not average. It also shows exactly where SubFlow's fix is needed. The
  method opened by the result is better guidance for choosing one-step training
  objectives.
- **Intrinsic, H2 FALSE:** the main message is "one call, fewer modes: the
  diversity cost of 1-NFE does not depend on the objective." The result opens a
  way to budget diversity by NFE: measure how many steps buy how much recall,
  and recommend that uses needing broad coverage should not default to 1-NFE.
- In either case, release the cross-family diversity test harness: matched
  precision, precision/recall, coverage, and memorization checks. This is the
  common measuring tool the area currently lacks.

## 6. Resources and schedule

**Cost:** 150–350 GPU-h. Run inference sweeps on OrangeGrid A100. Run the
from-scratch pair, about 200 GPU-h, on Anvil A100 or Delta H200. **We must check
that JAX works on H200 before depending on Delta**, based on an earlier hardware
lesson.

Schedule for about 14 weeks before CVPR: week 1 check → weeks 2–4 checkpoint
sweeps → weeks 5–8 from-scratch pair → weeks 9–10 H4 → weeks 11–13 analysis and
writing.

## 7. Risks and competing work to watch

- The main way this could be claimed first is for SubFlow to release code and
  cross-family results. Watch versions of [`2604.12273`](https://arxiv.org/abs/2604.12273) and search again every 6
  weeks.
- The He group could measure recall for its own Drifting model because the code
  is already in its `inference.py`. Move quickly: the first sweep takes days.
- Some checkpoint pairs may have no shared precision range. If so, report the
  reachable range honestly and rely on the matched from-scratch pair.

## 8. Week-1 check before locking the plan

1. Run Drifting and Shortcut recall once from beginning to end. This verifies
   ourselves that recall can be calculated now.
2. Check the JAX software on H200 and A100.
3. Repeat the literature search with the most recent 8 weeks named directly.
4. Confirm the exact CVPR 2027 deadline.
5. Get professor approval, then mark the page LOCKED and record the git hash.

## Related

[[Unified-Direction-Ranking-2026-08]] · the removed SigLIP-2 ladder draft (git history) ·
[[GPU-Resources-Across-Clusters]]

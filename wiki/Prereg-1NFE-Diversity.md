# Pre-registration: Is One-Step Generative Diversity Collapse Intrinsic, or an Averaging Artifact?

## Words used on this page

Read this list once. Every word below is used the same way everywhere on this
page.

- **1-NFE** — one network function evaluation. The model makes a whole image
  with one call to the network instead of dozens. NFE means "number of function
  evaluations," so 128-step means NFE = 128.
- **Checkpoint** — one saved copy of a trained model's weights.
- **Recall** — how much of the real data's variety the model covers. Higher is
  more diverse.
- **Precision** — how realistic the generated images look. We match precision
  between models so that a recall comparison is fair.
- **FID** (Fréchet Inception Distance) — the field's main image-quality score.
  Lower is better.
- **Guidance** (also written **cfg**) — a knob at sampling time that trades
  variety for image quality.
- **Shortfall** — how much recall a model loses when it drops from many steps
  to one step.
- **Cell** — one measurement condition in the sweep: one model, measured at one
  setting.
- **Arm** — one version of the experiment, run to answer one question.
- **Bootstrap** — a way to show how stable a result is. It re-samples the same
  data many times and re-computes the answer each time. Each re-computation is
  one **replicate**.
- **Confidence interval (CI)** — the range of values the data still allow. If
  the interval does not contain zero, chance alone is an unlikely explanation.
- **Holm correction** — a rule that guards against false positives when several
  hypotheses are tested at once.
- **Noise floor** — how much a number moves when nothing meaningful changes. A
  difference smaller than the noise floor means nothing.
- **Deviation** — any change made after the plan was locked. Every deviation is
  written down on this page.

## Sweep result (2026-08-10)

The full record is in nfe1
`runs/week2_check/20260809/RESULTS-week2-boundary.md`, plus 12 numbered notes.

**Where H1–H4 are defined:** in section 4 of this page, under
"Predictions, fixed before the run." Short version — **H1:** variety
(recall) falls as steps drop to one inside the averaging family.
**H2:** Drifting, trained without averaging, loses less than half of
what the averaging family loses. **H3:** a matched pair trained from
scratch shows the same ordering. **H4:** SubFlow's fix helps the
averaging family but not Drifting.

**H1 is confirmed in Shortcut and NOT confirmed in iMF.** In Shortcut every
matched contrast is positive: 6 of 6 for the B model and 7 of 7 for the XL
model, at +0.0917 at 50k samples. In iMF the same comparison gives +0.0004 on a
comparable grid. Both models train with an averaging objective. So the effect
depends on the individual model, and it is therefore not a property of the
averaging objective.

**H2 is true as an inequality, but the mechanism behind it is not supported.**

- The inequality: Drifting's 1-step recall EXCEEDS its matched 128-step
  reference by +0.2153 on the clean row of record. A twin row recovered after a
  crash agrees to within 0.0044, which validates feature salvage as a
  technique. The CI, recomputed on the primary row, is about [+0.205, +0.226].
- Why the mechanism fails: the comparison class is incoherent. Shortfalls
  within the averaging family span 0.0004–0.0969. And one averaging model and
  one non-averaging model are statistically indistinguishable at matched
  precision: +0.0053, CI [−0.0046, +0.0156]. That is an informative null,
  6.6× tighter than Drifting's advantage.

**Central finding: step-count shortfalls do not group by objective family. The
individual training recipe dominates.**

**What survives:** collapse is NOT intrinsic to one step. Drifting beats every
other 1-NFE model by +0.10–0.11 at matched precision. But the objective family
does not predict which models escape collapse.

**Two structural results:**

- Drifting's precision has a minimum at cfg 1.0, with a floor of 0.785. It
  shares no precision range with Shortcut-B at 1-NFE. That is the risk §7 named,
  now measured.
- Recall falls as the number of generated samples rises, by −0.04 to −0.10. The
  size of the fall depends on the model. The same-count rule is enforced.

**Cross-protocol comparability is established.** We reproduced the published
AFM, iMF, and Drifting numbers to within 0.03.

**Statistics used:** paired bootstrap, 2,000 replicates, Holm correction across
the hypotheses. Both adjusted p-values are below 0.001.

**Four errors that the worker found and corrected are logged in the ledger
amendments on this page.**

**Closing measurement, now COMPLETE.** The properly matched 50k 128-vs-1 pair
is **+0.1705, CI95 [+0.1609, +0.1802]** (paired bootstrap, 2,000 replicates).
The harness point estimate of +0.1699 agrees with it, within the documented
reimplementation difference. This SUPERSEDES the earlier +0.1365 interval. That
earlier interval belonged to a pair that sat 0.0006 outside the pre-registered
tolerance, so it is discarded, not averaged. H1 at 50k therefore reads: 128-vs-1
is +0.1705 [+0.1609, +0.1802], and 4-vs-1 is +0.0917. Both are resolved, and
they are ordered as predicted.

**Process note: two guards worked.** A stale cell reference nearly recomputed
the WRONG pair — the one outside tolerance — as if that were the correction.
The worker checked feature-file identity against the recorded rows before
running. Separately, the coordinator's garbled relay was refused by the sibling
worker.

**All GPU work is complete:** 39 cells. Both cards were handed to the
contextualization campaign under the drain rule.

**H3 (IMM M=1 vs M=4): the launch spec is delivered**, in nfe1
`runs/week2_check/20260809/H3-LAUNCH-SPEC.md`. It carries one design-critical
pre-declaration: the lock's "single knob" is CONFOUNDED. At a fixed batch size,
M also sets how many distinct time tuples each step uses, namely batch/M. So a
naive two-arm run would differ in the objective AND in time-sampling density at
the same time. The spec therefore prescribes:

- THREE arms, not two: M=4 at batch B; M=1 at batch B; and M=1 at batch B/4,
  which holds the tuple count equal.
- CIFAR scale, which sits inside the locked plan's reduced-scale clause.
- Fixed seeds and equal-tick stopping.
- Both guidance branches enumerated.
- CIFAR's own reference statistics and a re-derived resolution limit.
- An explicit "uninterpretable" outcome, declared in advance.
- A 20-tick pilot that MEASURES the cost before any allocation request, so no
  throughput is guessed.

Owner decision still needed: the compute window.

## Status: LOCKED 2026-08-08

The professor signed off and adopted the recommended defaults. Lock hash:
`ad85987d4b5e13c2e59c2afd4aa557fc5178338a`. These amendments came from the
week-1 and week-2 blocks below and are part of the lock.

- H1 is restated to matched-precision comparisons only. The step-count trend is
  confounded by guidance, so raw step slopes carry no evidential weight.
- The model roster drops the unreleased MeanFlow flagship and drops ROMS-IMLE.
  It adds iMF, pMF, IMM, and AFM.
- H2's within-checkpoint control is IMM's 1/2/4/8-step sampling.
- **H3 is primarily IMM's single-knob averaging-vs-distributional contrast, M=1
  vs M=4.** The matched pair trained from scratch is the fallback, used if that
  knob proves confounded.
- The JAX measurement path is canonical, because it gives byte-identical FID
  references across families.
- The caching and loader speedups may be used only after the bit-for-bit
  regression check passes.
- Treat the CVPR deadline as Nov 2026 ± 2 weeks, and re-check in September.

## Deviation 1 (2026-08-09, coordinator-approved; owner-ratified 2026-08-10)

**Who decided:** the coordinator approved this deviation. The owner ratified it
on 2026-08-10, together with all of its amendments below. Owner ratification,
verbatim: "1-NFE deviations ratified".

**What went wrong:** the lock demanded a "bit-for-bit" regression check on the
speedups. That is unattainable on this hardware for ANY run. Unmodified week-1
code drifts by the same ~1e-4 in FID across repeats and across machines. The
measured three-repeat noise floor is 1e-4–4e-4 FID and about 1e-3 recall. That
is two orders of magnitude below the ~0.1-scale effects that H1 and H2 test.

**What replaced it.** We report exact numerical differences against the noise
floor. We also added two tests that are truly exact, which noise cannot reach:
the cached reference features must be equal under `np.array_equal`, and the
labels-only loader's label stream must be equal element by element. So the
check is stronger wherever the hardware allows it.

**Result of the check:** the speedups passed and are cleared. They are 6.18×
faster. The sped-up run is CLOSER to week-1 than the un-sped-up rerun is, which
is the signature of noise rather than bias.

**Also recorded at this gate:** direct measurement CONFIRMED the guidance
explanation for the 128-step anomaly. Guided FID is 19.57, against a pass rule
of ≤22. The high-step numbers are un-quarantined.

### Correction (appended 2026-08-09, superseded the same day by isolation experiment X1)

**What we first believed:** GPU co-tenancy, meaning other jobs sharing the
card, caused the drift.

**What is actually true:** the drift is a bimodal XLA-autotuning flip. Six
measurements of one configuration fall into two tight clusters. Spread inside a
cluster is ≤5.4e-5 FID. The gap between clusters is 7.3e-3. A run executed
ALONE landed in the second cluster, so co-tenancy is demonstrably not needed to
produce the flip.

**What changes:** a comparison must now beat the between-cluster figure, not
the within-cluster one. The corrected noise floors are FID 7.3e-3, precision
0.0047, and recall 0.0014. That recall floor is ten times the within-cluster
figure we first reported.

**What does not change:** no conclusion moves. The binding constraint
everywhere is still the 0.020 reference-side resolution limit, which is 14×
the corrected recall floor. H1's shortfalls run 29–106× the corrected floor.

**Two more results from the same experiment:** the feature-saving hook was
exonerated, and the held 50k rows were released. The exclusive-card rule stays
in place as hygiene, but it is explicitly NOT sufficient for reproducibility.

**Also recorded:** the naive with-replacement bootstrap on generated samples is
structurally biased for k-NN recall. The measured bias is −0.0895. Intervals
now use reference-side resampling and paired-difference resampling instead. The
biased variant is kept only as a labeled diagnostic, and the
finite-generated-sample limitation is stated.

### Second amendment (same day)

**What went wrong:** one of the two "noise-immune exact tests" was itself
overstated. The reference-feature computation runs the same nondeterministic
kernels, so cache-versus-recompute cannot be bitwise equal. The measured
difference is ≤0.0042, which is no larger than recompute-versus-recompute.

**The honest statement:** the cache PINS the reference side. That improves
comparability between rows, by the same logic as sharing a seed. The
decision-relevant test is the effect of cached-versus-fresh features on
precision and recall. That test is G0b, and we report it whatever it shows.

**Noise floors are per measurement path.** The Shortcut path is unimodal, with
a recall floor of 0.0004; H1's evidence rests at 70–370× that floor. The
Drifting path is bimodal, with a recall floor of 0.0014.

### Third amendment (2026-08-10)

**What went wrong:** the week-1 claim that a 10k recall is a "lower bound" on
the 50k value was BACKWARDS. Recall FALLS as the number of generated samples
rises. The reason: k-nearest-neighbour radii shrink as the points get denser.
The measured fall is −0.04 to −0.10, and it depends on the model, so no single
correction can be applied to all of them.

**The rule now in force, and enforced in code:** every recall figure carries
its sample count. Never compare two recall figures unless their counts match.

**Effect on results:** none. Every contrast we reported was already
same-count.

Changes after this point are deviations and must be logged. The original draft
header follows.

## Previous status: DRAFT v1, 2026-08-03

This plan locks after the week-1 check in §8. Target venue: **CVPR 2027**. Our
deadlines table puts the deadline around Nov 13, 2026; confirm the exact date
before lock.

### Week-1 check result (2026-08-06; §8 steps 1–4 done, step 5 is the owner's)

1. **The pipeline is ours.** We ran both families end to end on our own H100,
   with 10,000 samples each and identical class labels and real reference.
   **Drifting's recall has never been published anywhere. It is 0.72 for
   latent L and 0.69 for pixel L at 1 step.** Shortcut recall rises with step
   count: 0.48 → 0.53 → 0.62 at 1, 4, and 128 steps. That is the direction H1
   predicts. These rows are NOT precision-matched, so they say nothing about H2
   yet.
2. **JAX works on our H100**, at 741 TFLOP/s of real GPU compute. Two questions
   stay open: the A100 on OrangeGrid, and the H200 on Delta. **Correction to
   §6: Anvil's A100 partition is not available to our account at all.** So the
   from-scratch pair must use the H100 partition or Delta. Note that both repos
   are JAX.
3. **Novelty holds.** We searched an 8-week window. SubFlow still has no code,
   and nobody has measured Drifting's diversity. One softening: ROMS-IMLE
   ([2607.19332](https://arxiv.org/abs/2607.19332)) publishes recall for a
   non-averaging one-step model. So "nobody has measured this" must become "no
   one has measured it across objective families." ROMS-IMLE is also a
   candidate second non-averaging model for H2.
4. **The CVPR 2027 deadline is not yet published.** The site 404s. CVPR 2027
   itself is Jun 20–24, in Seattle. Treat the deadline as Nov 2026 ± 2 weeks
   and re-check in September.
5. ⚠ **Open anomaly. Do not trust the high-step numbers yet.** Our Shortcut
   128-step FID is 40.1, but the repo publishes 15.5. We ruled out both the
   reference-statistics explanation and the CFG explanation. The released
   checkpoint may not be the one behind the README table. Recall comparisons
   are unaffected. One more item: the §4 claim that MeanFlow, iMF, and pMF
   weights are "on HF" did not survive a first search. Verify it before weeks
   2–4 depend on it.

   Full record: `code/nfe1/runs/week1_check/20260806/`.

### Week-2 preparation result (2026-08-08; read before locking)

1. **The 128-step FID anomaly is RESOLVED, and our number was right.** The
   repo's 15.5 is a guided figure — the paper states that CFG is used only at
   the smallest step size. Our 40.1 is unguided, and unguided DiT-B baselines
   land exactly there. An independent paper reran the same checkpoint with
   guidance and got 15.0. The high-step numbers are un-quarantined, pending one
   10-minute regression run. **Consequence for H1:** the week-1 rise of recall
   with steps is confounded by guidance, because guidance is baked into the
   low-step weights and absent at 128 steps. So H1 must be read at matched
   precision only, and its wording should say so at lock.
2. **Model roster corrections.** MeanFlow's flagship ImageNet weights were
   never released, so §4 must drop that claim. **iMF and pMF weights DO
   exist**: [iMF](https://huggingface.co/Lyy0725/iMF) and
   [pMF](https://huggingface.co/Lyy0725/pMF). Drifting, iMF, and pMF share a
   byte-identical FID reference file, so the JAX path gives cross-family
   comparability for free. **ROMS-IMLE is out.** It has no code and no weights,
   and its published recall of 0.50 is LOWER than its own baselines; the week-1
   note that treated it as a helpful second non-averaging model was backwards.
   **Verified replacements:** [IMM](https://huggingface.co/lumaai/imm), which
   samples at 1, 2, 4, and 8 steps from ONE checkpoint — exactly the H2
   control — and whose averaging-versus-distributional loss is a single knob,
   making a cleaner H3 than training from scratch; and AFM from ByteDance,
   which ships 50k sample packs, so recall costs nothing to compute.
3. Reference-feature caching and the parallel loader are implemented. They must
   reproduce the week-1 metrics bit-for-bit before any sweep uses them. That is
   a 15-minute GPU check.

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

There are two possible explanations, and each needs a different fix.

- **Intrinsic to one step:** a single network call must make every choice at
  once. Losing diversity is the price of 1-NFE, and no change to the training
  goal can remove it.
- **Caused by averaging:** most 1-NFE methods train with mean squared error
  (MSE) on average velocity. MSE can make the model learn a
  *frequency-weighted average over sub-modes*, which is called averaging
  distortion. Under this explanation, the training goal causes the collapse,
  and a goal that does not average can avoid it.

Nobody has run the experiment that tells these two explanations apart.

## 2. What recent work has shown

- **SubFlow** ([arXiv 2604.12273](https://arxiv.org/abs/2604.12273), Apr 2026) named the mechanism. It says
  that "when trained with MSE objectives, class-conditional flows learn a
  frequency-weighted mean over intra-class sub-modes." It uses sub-mode
  conditioning as a fix. **However, it tests only the averaging family:**
  MeanFlow and Shortcut. Its code is announced but not released.
- **Drifting** from the He group ([arXiv 2602.04770](https://arxiv.org/abs/2602.04770)) reaches 1-NFE
  **without an averaging objective**. It "evolves the pushforward distribution
  during training." That makes it the natural model that could disprove the
  averaging explanation, but nobody has measured its diversity. We verified one
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
  averaging and distillation family. The **comparison across objective families
  has not been run**, based on our Aug 2026 search.

## 3. What is new in our study

**Compare different step counts inside the same model family.** A simple
Drifting-versus-Shortcut recall comparison would be unfair, because their
quality is different: Drifting FID is 1.53, while Shortcut 1-step FID is 10.6.
That difference would be a hidden factor. Instead, each family acts as its own
control.

- For a family that supports several numbers of function evaluations, such as
  Shortcut at 1, 4, or 128 steps from one checkpoint, and the MeanFlow-family
  variants, the main measurement is the **slope of recall versus NFE at matched
  precision**.
- Drifting is built for 1-NFE. Compare it with a multi-step reference matched
  for precision. Then compare its recall *shortfall* with the averaging
  family's shortfall from 128 steps to 1 step.
- Train a **matched-budget pair from scratch** at B/4 size on ImageNet-64 or
  CIFAR. One member uses an averaging goal and the other uses the Drifting
  goal. This keeps model capacity, data, and compute the same. The released
  checkpoints differ in all three, so they cannot provide this control.

The predictions are clear. If collapse is **intrinsic to 1-NFE**, recall will
fall toward 1-NFE in every family, including Drifting. If collapse is
**specific to averaging**, only that family will show the drop, and Drifting
will keep its recall at 1-NFE. Either answer is useful.

Checked novelty: this is the first cross-family measurement of 1-NFE diversity,
the first diversity result of any kind for Drifting, and the first test of
SubFlow's explanation outside the family it was built on.

## 4. Exact experiment plan

**Models:** the released Drifting latent and pixel B & L checkpoints from
[`Goodeat/drifting`](https://huggingface.co/Goodeat/drifting); Shortcut DiT-B/XL; the MeanFlow, iMF, and pMF HF
weights; a many-step standard flow or diffusion model with matched architecture,
to serve as the precision-matched recall ceiling; and the pair we train from
scratch at B/4.

**Measurements:** precision and recall using [Kynkäänniemi k-NN](https://arxiv.org/abs/1904.06991), with k
fixed before the run. Also coverage and density. Also recall for each
ImageNet-256 class, because class-conditional coverage of sub-modes is where
averaging distortion should appear. Also distance to the nearest training
image, so that a model does not look "diverse" merely by copying. Match
precision through guidance and temperature sweeps. Use the same sample count,
50k, and fixed random seeds.

**Predictions, fixed before the run:**

- **H1:** recall decreases steadily as NFE goes to 1 inside the averaging
  family. That confirms SubFlow's starting claim in the family it studied.
- **H2, the deciding test:** Drifting's 1-NFE recall shortfall from its
  precision-matched multi-step reference is **less than half** the averaging
  family's shortfall from 128 steps to 1 step. We predict TRUE, which would
  support the averaging-specific explanation.
- **H3:** the matched pair trained from scratch shows the same ordering as H2.
  We predict TRUE. If it is FALSE, then differences between the released
  checkpoints caused H2, and we must honestly weaken the claim.
- **H4:** SubFlow-style sub-mode conditioning narrows the averaging family's
  shortfall but does not change Drifting's. We predict TRUE. If SubFlow's code
  is still unavailable, reimplement only its published recipe.

**Decision rules:** use bootstrap confidence intervals over sample splits, and
the Holm correction across H1–H4. Fix the precision-matching tolerance at
±0.02. Drop H4 without penalty if our implementation cannot reproduce
SubFlow's main published result within tolerance. That is our rule for checking
that a test model is usable: we do not publish a negative result that our own
failed reimplementation caused. H4 is only a confirmation test.

**Claims we will not make:** claims about text-to-image or video; claims about
one-step distillation methods, which use a different mechanism; and claims that
any released model is "bad." We study the class of training goals, not one
checkpoint.

## 5. What each possible result means

- **Averaging-specific, H2 TRUE:** the main message is "1-NFE does not force
  diversity collapse; the objective does." That supports training methods which
  do not average. It also shows exactly where SubFlow's fix is needed. The
  method this opens is better guidance for choosing one-step training
  objectives.
- **Intrinsic, H2 FALSE:** the main message is "one call, fewer modes: the
  diversity cost of 1-NFE does not depend on the objective." That opens a way
  to budget diversity by NFE. Measure how many steps buy how much recall, and
  recommend that uses which need broad coverage should not default to 1-NFE.
- In either case, release the cross-family diversity test harness: matched
  precision, precision and recall, coverage, and memorization checks. That is
  the common measuring tool the area currently lacks.

## 6. Resources and schedule

**Cost:** 150–350 GPU-h. Run the inference sweeps on OrangeGrid A100. Run the
from-scratch pair, about 200 GPU-h, on Anvil A100 or Delta H200. **We must
check that JAX works on H200 before we depend on Delta**, because of an earlier
hardware lesson.

Schedule for about 14 weeks before CVPR: week 1 check → weeks 2–4 checkpoint
sweeps → weeks 5–8 from-scratch pair → weeks 9–10 H4 → weeks 11–13 analysis and
writing.

## 7. Risks and competing work to watch

- The main way someone could claim this first is for SubFlow to release code
  and cross-family results. Watch versions of [`2604.12273`](https://arxiv.org/abs/2604.12273) and search again
  every 6 weeks.
- The He group could measure recall for its own Drifting model, because the
  code already sits in its `inference.py`. Move quickly: the first sweep takes
  days.
- Some checkpoint pairs may share no precision range. If that happens, report
  the reachable range honestly and rely on the matched from-scratch pair.

## 8. Week-1 check before locking the plan

1. Run Drifting and Shortcut recall once, from beginning to end. This verifies
   for ourselves that recall can be calculated now.
2. Check the JAX software on H200 and A100.
3. Repeat the literature search, and name the most recent 8 weeks directly.
4. Confirm the exact CVPR 2027 deadline.
5. Get professor approval, then mark the page LOCKED and record the git hash.

## Related

[[Unified-Direction-Ranking-2026-08]] · the removed SigLIP-2 ladder draft (git history) ·
[[GPU-Resources-Across-Clusters]]

# Pre-registration: Is One-Step Generative Diversity Collapse Intrinsic, or an Averaging Artifact?

Status: **DRAFT v1, 2026-08-03 — for professor sign-off.** This plan locks
after the week-1 check in §8. Target venue: **CVPR 2027**. The deadline is
around Nov 13, 2026 in our deadlines table; confirm the exact date before lock.

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

# Does the Capacity Gap Remain After Tuning Temperature for Each Teacher?

*Updated 2026-08-02 for the general research wiki.*

**Knowledge distillation (KD)** trains a smaller student model to copy a larger
teacher. A **capacity gap** happens when a larger, more accurate teacher gives a
student worse results than a smaller teacher. **Temperature** controls how sharp
or spread out the teacher's predicted probabilities are.

Status: **CHECK OF EARLIER WORK FAILED 2026-07-25 (run by the owner). Do not run
this plan.** A larger published study already included the main control, and
its result goes against this plan's main prediction.

## What the earlier-work check found

**[Asymmetric Temperature Scaling](https://arxiv.org/abs/2210.04427) Makes Larger Networks Teach Well Again**
(NeurIPS 2022) already ran almost the same control at a larger scale:

- It tested **18 teachers** from three model families: six ResNets, six
  WideResNets, and six ResNeXts. This plan proposed only 10 teachers. It also
  tested three students.
- It searched symmetric temperature values `tau in {1, 2, 4, 8, 12, 16}`.
- It tested two student-temperature rules: student `tau` equals teacher `tau`,
  or student `tau` equals 1.
- Its supplement says that it changed hyperparameters and selected the best
  result when drawing the teacher-size curves. In practice, **each
  teacher-student point could use its best scalar-temperature setting.**

**Result: even after scalar temperature was tuned, larger teachers sometimes
still produced worse students.** Scalar temperature does not remove the
capacity gap. The paper's fix, ATS, uses a *higher* temperature for the
correct-class logit and a *lower* temperature for the wrong-class logits. This
reduces overconfidence while keeping useful differences among wrong answers.

The two statements below appeared in the old plan. They are **false and must
not be reused**:

> "Every paper in the knowledge-distillation literature uses one temperature
> for the entire teacher ladder."

> "The contribution is the control none of them ran."

Other papers also cover the proposed explanation:

- [Spherical KD](https://arxiv.org/abs/2010.07485) (2020) studies how larger teachers produce larger logits. It
  normalizes logits so the student learns their direction instead of their
  size.
- [Logit Standardization](https://arxiv.org/abs/2403.01427) (CVPR 2024) sets an effective temperature using the
  standard deviation of the logits. This covers the idea that the teacher and
  student have mismatched logit scales.
- [DTKD](https://arxiv.org/abs/2404.12711) (2024) sets a different teacher and student temperature for each
  example, based on how sharp their predictions are.
- *Exploring Dark Knowledge under Various Teacher Capacities* extends ATS to
  instance-specific ATS (ISATS). It plots scalar, ATS, and ISATS temperature
  rules across ResNet, WideResNet, and ResNeXt teacher-size ladders. This is
  almost the exact explanation planned here.
- [CIST](https://arxiv.org/abs/2605.20357) (May 2026) gives each example its own temperature and connects teacher
  entropy in theory to the ratio of maximum logit to temperature. This covers
  much of the planned **entropy-matched test (H5)**.

The evidence for H2 is not merely uncertain; it points the other way. Tuning
one scalar temperature per teacher may improve `rho` somewhat, but it probably
will **not** make most of the capacity gap disappear.

## A missing control in the old plan

The owner caught a design problem that the draft missed. Comparing only a fixed
`tau=4` with a temperature tuned separately for every teacher cannot tell these
two explanations apart:

1. temperature should change with teacher capacity, or
2. `4` is simply a poor setting for every teacher.

The study would need a **globally tuned condition**: choose the best single
temperature shared by all teachers. The comparison that isolates the effect of
teacher capacity is *best shared temperature versus best `per-teacher tuned`
temperature*, not *default `fixed tau=4` versus per-teacher temperature*.

ATS also selected the best setup at each point. That choice favors the idea
that tuning will work, because it can accidentally select lucky results. The
capacity gap still remained. A stricter plan that avoids this leak should
therefore find the gap **more** stable, not less. The likely outcome of a clean
repeat is: ATS was right and may even have understated the gap.

One small question may remain for a section, but not a full paper: no checked
source directly compares vanilla KD with temperature tuned per teacher against
fixed-temperature [DKD](https://arxiv.org/abs/2203.08679) on one teacher ladder. This was H3 below. Any report
must call itself an extension of ATS, Logit Standardization, ISATS, and
adaptive-temperature KD. It cannot claim to be the first temperature control.

---

## Original plan, kept only as a record — superseded

Everything below is historical. Its core claim and design are no longer valid.

## Original question

> Every paper in the knowledge-distillation literature uses **one temperature
> for the entire teacher ladder**. Does the capacity gap remain when temperature
> is tuned separately for each teacher?

## Why the plan originally seemed important

The literature's proposed explanation seemed to predict a hidden factor that
could change the result. That hidden factor was temperature.

- **[Logit Standardization](https://arxiv.org/abs/2403.01427)** (Sun et al., CVPR 2024) connects the capacity gap
  to a shared-temperature requirement that teacher and student logits match
  exactly in **range and variance**. Temperature directly controls that match.
- Published fixed values included [DKD](https://arxiv.org/abs/2203.08679) `tau=4`, Logit Standardization
  `tau=2`, and [DIST](https://arxiv.org/abs/2205.10536) `tau=4/1`.
- Frank & Davis ([`2603.02430`](https://arxiv.org/abs/2603.02430)) ran the broadest temperature study found. It
  crossed `tau` with KD method, optimizer, batch size, epochs,
  initialization, and dataset detail, but **did not change teacher capacity**.

The proposed explanation was simple. Larger teachers are often more confident:
they have lower prediction entropy and larger logits. At one fixed temperature,
the larger teacher's softened probability distribution can still be sharper.
That leaves less "dark knowledge," meaning less information about the wrong
classes. If this caused the capacity gap, the best temperature should
**increase as teacher capacity grows**, and matching the softness should remove
the effect.

This was intended as a claim that could be disproved with data, not just a way
to describe the result.

## Results already known when the old plan was written

These were background and were never going to be claimed as new:

- Logit Standardization, CVPR 2024, Table 5 already had a grid of 6 teachers by
  4 objectives on CIFAR-100. Vanilla KD was flat or non-monotonic, while [DKD](https://arxiv.org/abs/2203.08679)
  rose steadily.
- Earlier work had already shown the capacity gap: [Cho & Hariharan](https://arxiv.org/abs/1910.01348),
  [TAKD](https://arxiv.org/abs/1902.03393), and [Menon et al.](https://arxiv.org/abs/2005.10419).
- DKD already had the `(1 - p_t)` suppression explanation, and [DIST](https://arxiv.org/abs/2205.10536) had the
  exact-match explanation.
- Busbridge et al., ICML 2025, Appendix C.1.3 already had the U-shape theorem.

The old plan incorrectly said its new contribution was **the control none of
those papers had run**.

## Old study design

### Stage 1 — choose temperatures only

| Factor | Levels |
|---|---|
| Teacher | **>= 10** on one ordered ladder: WRN-16-1/2/4/8, WRN-28-1/2/4, WRN-40-1/2/4, plus ResNet50 and VGG13 |
| Temperature | `tau in {1, 2, 3, 4, 6, 8, 12, 16}` |
| Objective | vanilla KD only |
| Student | WRN-16-2 |
| Seeds | 3 |

The plan required at least 10 teacher models rather than the 6 used in earlier
work because a Spearman correlation from only 6 points is too noisy for the
main measurement.

Stage 1 would use a held-out validation split taken from the CIFAR-100 training
set: 45,000 training examples and 5,000 validation examples. It would never use
the test set. Its output would be one selected temperature, `tau*(t)`, for each
teacher.

### Stage 2 — fixed evaluation plan

| Factor | Levels |
|---|---|
| Teacher | the same `>= 10` ladder |
| **Temperature rule** | **(a) fixed `tau=4`**, the common default; **(b) tuned `tau*(t)`** from Stage 1; **(c) entropy-matched `tau_H(t)`** |
| Objective | vanilla KD and [DKD](https://arxiv.org/abs/2203.08679) |
| Student | WRN-16-2, plus one smaller model: ResNet-8 or MobileNetV2 |
| Seeds | 5, different from the Stage 1 seeds |

The entropy-matched condition was meant to be the useful new part. Choose
`tau_H(t)` so that the teacher's average softened prediction entropy stays the
same across the ladder. This is a one-line rule that needs **no tuning and no
validation split**. If it kept most of the gain from tuned `tau`, the paper
could offer a free fix instead of only pointing out a problem.

Every cell would use the same standard training schedule. The gate had removed
the extra training-budget condition because Busbridge et al. already argued
against lack of compute as the explanation at LLM scale. That condition would
have tripled the number of runs without adding enough value.

## Old measurements

The **main measurement** was Spearman `rho`, a rank-correlation number, between
**teacher accuracy** and **student accuracy** across the ladder. It would be
computed separately for each objective, temperature rule, and student. [DKD](https://arxiv.org/abs/2203.08679)
reports the same number—`rho ~ 0.26` for KD and `~0.94` for DKD—so
the results would be directly comparable.

A second measurement with more statistical power was the ordinary least
squares (OLS) slope of student accuracy versus teacher accuracy, with a
bootstrap confidence interval. `rho` from about 10 points remains noisy; the
slope also uses the sizes of the differences.

### Hypotheses and pass thresholds fixed in advance

- **H1, repeat the known effect.** `rho(KD, fixed tau=4) < 0.5`. If it were
  already high, the capacity-gap effect would not repeat on this ladder. The
  study would report that controlled non-replication and stop.
- **H2, main claim.** `Delta_rho = rho(KD, tuned) - rho(KD, fixed) >= 0.30`,
  and the seed-bootstrap confidence interval for `Delta_rho` must exclude zero.
- **H3, strongest claim.** `rho(KD, tuned)` must be within `0.15` of
  `rho(DKD, fixed)`. Then **much of DKD's advantage over KD across the teacher
  ladder could be called a temperature artifact.**
- **H4, explanation.** `tau*(t)` should rise with teacher capacity. Spearman
  correlation between `tau*` and teacher confidence—mean maximum probability
  or mean logit norm—should be `>= 0.60`. The gap left after temperature
  matching should also track the logit-scale mismatch left over.
- **H5, useful fix.** Entropy-matched `tau_H` should recover `>= 70%` of
  the `Delta_rho` from tuned `tau*`.

The analysis would bootstrap over seeds with 10,000 draws and random seed
`20260725`. It would report all effects both as raw points and as multiples of
the pooled standard deviation across seeds. Published work often reports only
point differences. For example, [CRD](https://arxiv.org/abs/1910.10699) Table 10 has a teacher that is 4.7
points better but moves the student by `-0.02` while `sigma = 0.32`. The report
would also have to give the fraction of teacher pairs whose difference is
smaller than the seed noise floor.

## Checks required before accepting any Stage 2 result

- **Repeat a published anchor.** For the Logit Standardization ladder subset
  VGG13, WRN-28-2, WRN-40-2, WRN-16-4, WRN-28-4, and ResNet50 into WRN-16-2,
  the `tau=4` KD column must match LS Table 5 within a limit chosen in advance.
  This checks the code against a published result before making a new claim.
- Measure teacher accuracy rather than assume it. Report both accuracy and
  model size, and state conclusions using whichever variable the data supports.
- Store checkpoint, settings, code, and evaluator hashes in one manifest, which
  is a record of the exact files and versions used. The analysis program must
  reject files from different manifests.
- Stage 1 and Stage 2 must use different random seeds, and Stage 1 must never
  use the test set.

## Old decision rules

- **Full paper:** H1 repeats and H2 passes. The gap exists at fixed temperature
  and mostly disappears after per-teacher tuning. H3 and H5 decide how strong
  the story can be.
- **Full paper with a different story:** H1 fails. That would be a controlled
  failure to repeat an effect claimed by about 45 papers and controlled by
  about 6.
- **Smaller paper:** H1 passes but H2 fails. The capacity gap survives
  temperature tuning, making the standard explanation stronger. It would be an
  honest controlled confirmation, but less valuable.
- **Stop:** every difference stays inside seed noise and no factor can be
  separated.

## Known problems in the old design

1. **Overfitting temperature selection.** Stage 1 chooses among 8 temperatures
   for 10 teachers using only 5,000 validation examples. The plan used separate
   seeds, a fixed rule—highest validation accuracy, with ties going to lower
   `tau`—and a report of the validation-to-test gap.
2. **Teacher accuracy and model size are not the same.** They do not perfectly
   track each other on a real ladder. Both must be reported. For comparison
   with [DKD](https://arxiv.org/abs/2203.08679), `rho` was defined using accuracy.
3. **About 10 points still give a noisy `rho`.** This is why the slope was also
   planned as a main measurement.
4. **CIFAR-100 results may not transfer.** A fixed extra check on Tiny-ImageNet
   would use 4 teachers, 2 temperature rules, and 1 objective. It would be a
   robustness check, not a main measurement.
5. **How to describe the paper.** It would be a control, not a discovery. The
   abstract must credit DKD, [DIST](https://arxiv.org/abs/2205.10536), and Logit Standardization. TMLR welcomes
   this kind of work. A main-track paper falsely sold as a discovery should be
   rejected.

## Items that originally blocked the plan

1. Search by task and older terms for any paper that tunes temperature per
   teacher across a capacity ladder; any entropy- or confidence-matching rule
   in calibration and label-smoothing work; and citations to Logit
   Standardization and Frank & Davis through 2026.
2. Manually check [DKD](https://arxiv.org/abs/2203.08679)'s appendix teacher-ladder table. It came from ar5iv
   because the CVPR OpenAccess PDF returned 403.
3. Fix the exact teacher checkpoints and confirm their accuracies are ordered
   before training students.
4. Choose the allowed error for reproducing LS Table 5 before running.
5. Decide whether [DIST](https://arxiv.org/abs/2205.10536) belongs among the main objectives or only the extra
   checks. **DIST and DKD had not been tested on one ladder**, a real but
   secondary comparison gap.
6. Choose a practically important `Delta_rho` in advance, so a tiny but
   statistically clear change cannot be called success.

## Old compute estimate

Stage 1: `10 teachers x 8 tau x 3 seeds = 240` student runs.

Stage 2: `10 x 3 policies x 2 objectives x 2 students x 5 seeds = 600` runs.

Teachers would be trained once and reused. At CIFAR-100 WRN-16-2 scale, the
total was about 600–700 GPU-hours, or several days to two weeks on the available
cluster. The Tiny-ImageNet check would add about 40 runs.

## Related

Capacity-Gap-Falsification-Preregistration (svib repo wiki) — an older version
of this plan and its check.
[[Direction-Gate-Results]] — the repeated failure pattern this plan tried to
avoid.
[[LLM-KD-Direction-Gates]] — the distillation survey that followed this check.

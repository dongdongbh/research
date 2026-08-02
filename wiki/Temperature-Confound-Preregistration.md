# Does the Capacity Gap Survive Per-Teacher Temperature Tuning?

Status: **PRIOR-ART GATE FAILED 2026-07-25 (owner-run). Do not run this
protocol.** The central control was already performed at larger scale, and its
result contradicts this protocol's primary hypothesis.

## Gate result — the premise was false

**Asymmetric Temperature Scaling Makes Larger Networks Teach Well Again**
(NeurIPS 2022) ran essentially this control, at larger scale:

- **18 teachers** across three families (six ResNets, six WideResNets, six
  ResNeXts) — versus the 10 proposed here — and three students.
- Symmetric temperature searched over `tau in {1, 2, 4, 8, 12, 16}`.
- Two student-temperature policies (student `tau` = teacher `tau`; student
  `tau` = 1).
- Per the supplement, hyperparameters were adjusted and the best result selected
  when drawing the teacher-ladder curves — i.e. **each teacher-student point was
  effectively allowed its best scalar-temperature configuration.**

**Their finding: even after tuning scalar temperature, larger teachers still
sometimes produced worse students.** Scalar temperature tuning does not remove
the capacity gap. Their proposed fix, ATS, uses a *higher* temperature on the
correct-class logit and a *lower* one on the wrong-class logits, to reduce
overconfidence while preserving discrimination among wrong classes.

Two claims in the draft below are therefore **false and must not be reused**:

> "Every paper in the knowledge-distillation literature uses one temperature for
> the entire teacher ladder."

> "The contribution is the control none of them ran."

**The rest of the mechanism is also well covered.** Spherical KD (2020) studies
the larger-teacher-to-larger-logit-magnitude mechanism and normalizes logits so
the student learns direction rather than magnitude. Logit Standardization
(CVPR 2024) sets effective temperature from logit standard deviation — the
logit-scale-mismatch account. DTKD (2024) computes sample-specific teacher and
student temperatures from prediction sharpness. *Exploring Dark Knowledge under
Various Teacher Capacities* extends ATS to instance-specific ATS (ISATS) and
**plots scalar, ATS and ISATS temperature policies across ResNet, WideResNet and
ResNeXt capacity ladders** — nearly the exact scientific mechanism proposed
here. CIST (May 2026) assigns sample-wise temperatures and theoretically links
teacher entropy to the max-logit-to-temperature ratio, which substantially
covers the **entropy-matched arm (H5)**.

**H2's prior is not neutral, it is negative.** Existing evidence predicts that
per-teacher scalar tuning will improve `rho` somewhat but will *not* make the
capacity gap substantially disappear.

**A design flaw the owner caught that this draft missed.** Comparing only
`fixed tau=4` against `per-teacher tuned` cannot separate two explanations: that
capacity-dependent temperature matters, or that `4` was simply a poor default
for everyone. A **globally tuned arm** — one temperature shared across all
teachers, itself tuned — is required, and the contrast that isolates
capacity-dependence is *global-tuned versus per-teacher-tuned*, not
*default versus per-teacher-tuned*.

**One observation that cuts against a replication being interesting.** ATS
selected the best configuration per point, which is a selection bias *in favour
of* the tuning hypothesis. They still found the gap survived. A stricter,
leakage-resistant protocol would therefore be expected to find the gap **more**
robust, not less — so the expected outcome of a clean replication is "ATS was
right, and if anything understated it."

**What remains, at section rather than paper scale:** per-teacher-tuned vanilla
KD versus fixed-temperature DKD on one ladder (the draft's H3) does not appear
in the primary sources checked. Any writeup must be framed as an extension of
ATS, Logit Standardization, ISATS and adaptive-temperature KD — never as a first
temperature control.

---

## Original protocol, retained for the record. Superseded.

## The question

> Every paper in the knowledge-distillation literature uses **one temperature
> for the entire teacher ladder**. Does the capacity gap survive when
> temperature is tuned per teacher?

## Why this is a real gap and not a procedural nitpick

The literature's own stated mechanism predicts its own confound.

- **Logit Standardization** (Sun et al., CVPR 2024) attributes the capacity gap
  to the shared-temperature exact-match constraint on teacher/student logit
  **range and variance**. Temperature is precisely the knob that controls that.
- Fixed temperatures across the whole ladder: DKD `tau=4`, LS `tau=2`,
  DIST `tau=4/1`.
- The most systematic temperature study (Frank & Davis, `2603.02430`) crosses
  `tau` against KD method, optimizer, batch size, epochs, initialization and
  dataset granularity — and **explicitly does not vary teacher capacity.**

**The mechanistic reason to expect a confound.** Higher-capacity teachers are
typically more confident: lower predictive entropy, larger logit magnitudes. At
a *fixed* temperature, a more confident teacher's softened distribution remains
peakier, so the "dark knowledge" signal shrinks as capacity grows — through no
fault of capacity itself. If that is what drives the gap, then the optimal
temperature should **increase with teacher capacity**, and matching effective
softness should remove the effect.

That is a falsifiable mediation claim, not a rhetorical one.

## What is already published, and therefore not claimed

Cited as background, never as findings:

- The crossed 6-teacher x 4-objective grid on CIFAR-100 (**LS, CVPR 2024,
  Table 5**), showing vanilla KD flat/non-monotone while DKD rises monotonically.
- The capacity gap itself (Cho & Hariharan; TAKD; Menon et al.).
- DKD's `(1 - p_t)` suppression account and DIST's exact-match account.
- The U-shape theorem (Busbridge et al., ICML 2025, App. C.1.3).

The contribution is **the control none of them ran.**

## Design

### Stage 1 — temperature sweep (selection only)

| Factor | Levels |
|---|---|
| Teacher | **>= 10** on one ordered ladder (WRN-16-1/2/4/8, WRN-28-1/2/4, WRN-40-1/2/4, plus ResNet50, VGG13) |
| Temperature | `tau in {1, 2, 3, 4, 6, 8, 12, 16}` |
| Objective | vanilla KD only |
| Student | WRN-16-2 |
| Seeds | 3 |

`>= 10` ladder points, not the literature's 6, because Spearman correlation over
6 points is too noisy to carry a primary endpoint.

Selection uses a **held-out validation split carved from CIFAR-100 train**
(45k/5k). The test set is never touched in Stage 1. Output: `tau*(t)` per
teacher.

### Stage 2 — locked evaluation

| Factor | Levels |
|---|---|
| Teacher | the same `>= 10` ladder |
| **Temperature policy** | **(a) fixed `tau=4`** (literature default) · **(b) tuned `tau*(t)`** from Stage 1 · **(c) entropy-matched `tau_H(t)`** |
| Objective | vanilla KD, DKD |
| Student | WRN-16-2, plus one smaller (ResNet-8 or MobileNetV2) |
| Seeds | 5, disjoint from Stage 1 |

**The entropy-matched arm is the constructive contribution.** Choose `tau_H(t)`
so that the mean softened-teacher predictive entropy is constant across the
ladder — a one-line rule requiring **no tuning and no validation split**. If it
recovers most of the tuned-`tau` benefit, the paper ships a free fix rather than
only a criticism.

Training budget is **fixed** at the standard schedule for all cells. The gate
prescribed cutting the budget arm: Busbridge et al. already argue against the
compute-insufficiency account at LLM scale, and it is not worth 3x the cells.

## Endpoints

**Primary.** Spearman `rho` between **teacher accuracy** and **student
accuracy** across the ladder, computed per (objective, temperature policy,
student). This is the statistic DKD reports (`rho ~ 0.26` for KD, `~0.94` for
DKD), so it is directly comparable to published work.

**Secondary, higher power.** OLS slope of student accuracy on teacher accuracy
with bootstrap CI — `rho` over ~10 points is still noisy, and the slope uses the
magnitudes.

**Pre-registered hypotheses and thresholds.**

- **H1 (replication).** `rho(KD, fixed tau=4) < 0.5`. If it is already high, the
  effect does not replicate on this ladder and **the study reports that and
  stops** — a controlled non-replication of a widely asserted effect.
- **H2 (primary).** `Delta_rho = rho(KD, tuned) - rho(KD, fixed) >= 0.30`, with
  the seed-bootstrap CI on `Delta_rho` excluding zero.
- **H3 (the sharp claim).** `rho(KD, tuned)` lies within `0.15` of
  `rho(DKD, fixed)`. If so, **DKD's advantage over KD on the teacher ladder is
  substantially a temperature artifact.**
- **H4 (mechanism / mediation).** `tau*(t)` increases with teacher capacity:
  Spearman `>= 0.60` between `tau*` and teacher predictive confidence (mean
  max-probability or mean logit norm). Additionally, the residual gap after
  `tau`-matching should correlate with residual logit-scale mismatch.
- **H5 (constructive).** Entropy-matched `tau_H` recovers `>= 70%` of the
  `Delta_rho` achieved by tuned `tau*`.

**Statistics.** Seed bootstrap, 10,000 draws, seed `20260725`. All effects
reported in units of **pooled seed standard deviation** as well as raw points —
the literature reports bare point differences, and CRD's own Table 10 has a
4.7-point-better teacher moving the student by `-0.02` against `sigma = 0.32`.
Fraction of ladder pairs whose difference is below the seed noise floor is a
required reported quantity.

## Validity gates before any Stage 2 number is accepted

- **Anchor reproduction:** the `tau=4` KD column on the LS ladder subset
  (VGG13, WRN-28-2, WRN-40-2, WRN-16-4, WRN-28-4, ResNet50 → WRN-16-2) must
  reproduce LS Table 5 within a pre-declared tolerance. This validates the
  harness against published numbers before anything new is claimed.
- Teacher accuracies measured, not assumed, and reported alongside capacity;
  conclusions stated in terms of whichever variable the data supports.
- Checkpoint, config, code, and evaluator hashes in one manifest; the analyzer
  refuses mixed-manifest artifacts.
- Stage 1 and Stage 2 seeds disjoint, and the test set untouched in Stage 1.

## Decision gate

**Full paper** if H1 replicates and H2 holds — the gap exists at fixed
temperature and substantially dissolves under per-teacher tuning. H3 and H5
determine how strong the framing can be.

**Full paper, different framing,** if H1 fails — a controlled non-replication
of an effect asserted by ~45 papers and controlled by ~6.

**Reduced scope** if H1 holds and H2 fails: the capacity gap survives
temperature tuning, which *strengthens* the standard account. Report as a
controlled confirmation, and note the mechanism claim survived a test it had
never faced. Worth less, still honest, still publishable as a short paper.

**Stop** if the whole surface sits inside seed noise and no factor resolves.

## Known hazards

1. **Selection overfitting in Stage 1.** 10 teachers x 8 temperatures on a 5k
   validation split. Mitigated by disjoint seeds, a locked selection rule
   (highest validation accuracy, ties to lower `tau`), and reporting the
   validation-to-test gap.
2. **Teacher accuracy and teacher capacity are different variables** and are not
   perfectly correlated on any real ladder. Both are reported; `rho` is defined
   against accuracy for comparability with DKD.
3. **`rho` over ~10 points remains noisy** — hence the slope as a co-primary.
4. **CIFAR-100 may not transfer.** A robustness arm on Tiny-ImageNet with 4
   teachers x 2 policies x 1 objective is pre-declared as a check, not a
   primary endpoint.
5. **Positioning.** This is a control, not a discovery. DKD, DIST and LS must be
   credited in the abstract. TMLR's scope invites exactly this; a main-track
   submission framed as discovery would be correctly rejected.

## Open items blocking lock

1. **Final prior-art gate**, searched by task and older vocabulary: whether any
   paper tunes temperature per teacher across a capacity ladder; whether
   entropy-matched or confidence-matched temperature already exists as a
   proposed rule (check the calibration and label-smoothing literatures, where
   this idea would plausibly live under different words); and forward citations
   of LS and Frank & Davis through 2026.
2. **Manually verify DKD's appendix teacher-ladder table** — it was extracted
   via ar5iv and the CVPR OpenAccess PDF returned 403.
3. Fix the exact teacher checkpoint set and confirm accuracies are ordered
   before any student training.
4. Pre-declare the anchor-reproduction tolerance for the LS Table 5 check.
5. Decide whether DIST joins the primary objectives or the robustness arm. Note
   the gate found **DIST versus DKD has never been run on one ladder** — a real
   but secondary benchmarking gap.
6. Pre-declare the practically meaningful `Delta_rho`, so a statistically
   resolved but negligible change is not reported as success.

## Compute

Stage 1: `10 teachers x 8 tau x 3 seeds = 240` student runs.
Stage 2: `10 x 3 policies x 2 objectives x 2 students x 5 seeds = 600` runs.
Teachers trained once and reused. At CIFAR-100 WRN-16-2 scale this is roughly
600-700 GPU-hours total, days to two weeks on the available cluster. The
robustness arm adds ~40 runs.

## Related

[[Capacity-Gap-Falsification-Preregistration]] — the superseded version and its
gate.
[[Direction-Gate-Results]] — the recurring failure mode this protocol was
designed around.

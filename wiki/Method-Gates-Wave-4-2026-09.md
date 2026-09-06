# Method Gates — Wave 4, 2026-09-01

Four directions from [[Direction-Audit-2026-09-01]] went through a
pre-registration gate today: the owner's method filter, a second independent
scoop pass (the prereg workflow's two-pass rule), live verification of every
load-bearing fact, and a week-1 decisive step with a kill number fixed before
any run. One Opus agent per gate. Full records, with every search and quote:
[[Gate-2026-09-01-canonicalization]] · [[Gate-2026-09-01-horizon]] ·
[[Gate-2026-09-01-dreamgen]] · [[Gate-2026-09-01-sparsity]].

**All four survive; every one lost half a star or more to something the first
pass missed.** That is the two-pass rule doing its job.

| Gate | Verdict | Level | Week-1 cost | Owner decision needed |
|---|---|---|---|---|
| 1. Role-preserving canonicalization | SURVIVES ★★★★ conditional | 4 | under 2 GPU-h, one day | approve the week-1 step |
| 2. Horizon-calibrated world-model policy evaluator | SURVIVES ★★★½ conditional | 3 (bottom) | 30–60 GPU-h, one to two days | approve the week-1 step |
| 3. Recorded-outcome anchor for generated-video judges | SURVIVES ★★★, as a section of the ICLR paper | 3 | 10–20 GPU-h, one day | approve as an ICLR arm, not a paper |
| 4. Sparsity-premise instrument | SURVIVES ★★★ conditional | 4 (premise) / 3 (rule) | ~10 GPU-h pilot, ~81 total | accept or reject one sentence |

## Gate 1 — Role-preserving canonicalization

**What it is.** Gupta, Isola and co-authors show one orthogonal map Q (a pure
rotation) fitted on a few hundred image embeddings aligns two independently
trained image-text models ([2602.17584](https://arxiv.org/abs/2602.17584), one
citation ever). We predict Q keeps category information and loses role
information, then repair it with a role-anchored fit. Route 1 (new mechanism).

**What the second pass found.** A rotation cannot destroy anything a linear
probe can see: rotate the data and the probe rotates with it. So the whole
claim rests on the map's residual, the part the fit gets wrong, being lopsided:
small along category directions, large along role directions. And the test must
read the text side, because our own probe shows the pooled image vector holds
no linearly decodable role information to lose (0.52–0.54 on all three
backbones). Measure transfer: train the probe in the target model's space, test
it on source embeddings carried across by Q. Neighbours to cite, not
competitors: latent-space translation and model stitching (Moschella et al.
ICLR 2023, Maiorca et al. NeurIPS 2023), backward-compatible embeddings, and
[Plato's Cave 2604.18572](https://arxiv.org/abs/2604.18572). The source repo
has no licence: reimplement, never fork.

**Week-1 step (under 2 GPU-h).** Pair CLIP ViT-B/32 with OpenCLIP LAION-2B
(both 512-wide, a true rotation); second pair CLIP into SigLIP 2 (rectangular
fit). Fit Q by orthogonal Procrustes on 1,000 image anchors, sweep anchors in
{128, 256, 1000, 2360}. Measure role decodability by transfer on both strata,
CIFAR-100 zero-shot, image-to-text Recall@1, SugarCrepe++ and Winoground through
the locked evaluator. Let D be the drop in spatial-stratum role decodability
from the target's own ceiling to the transfer number, with a paired bootstrap
interval. **Dead if the whole interval for D is below 2 points. Note only if D
is 2–10 points. Alive only if D ≥ 10 with the interval excluding 2 AND
CIFAR-100 and Recall@1 each drop by under 2 points.** Kill arm for the method
half: if refitting Q on every image shrinks D below 2, there is no method, only
"use more anchors".

## Gate 2 — Horizon-calibrated world-model policy evaluator

**What it is.** Borrow horizon selection from off-policy evaluation (Thomas and
Brunskill 2016; the curse-of-horizon papers) and ship a tool that returns the
rollout length H* at which a world model's policy ranking best matches the
simulator, a bias-corrected ranking, and a "do not trust past H*" interval.
Route 3. It is the method paper the RoboJudge audit is required to name.

**What the second pass found.** [SC3-Eval 2606.18610](https://arxiv.org/abs/2606.18610)
already stops drifting rollouts early and shows the horizon effect on ranking
is real (Pearson 0.871 → 0.929). We must never write "nobody truncates
rollouts". What is empty: the rank-agreement-versus-horizon curve, the estimator
that picks H*, and any interval on it.

**Week-1 step (30–60 GPU-h on OrangeGrid; the audit's 20–40 was optimistic
because the simulator side dominates).** On [jepa-wms](https://github.com/facebookresearch/jepa-wms)
(CC-BY-NC 4.0, academic use only), 8–12 "policies" from planner settings and
checkpoints, four environments (Push-T, PointMaze, Wall, Metaworld). Score
inside the model at H = 1…K and in the simulator; Bradley–Terry both sides;
τ(H) with the session bootstrap; report MMRV too. Two traps pre-registered:
choose H* on a validation split and report only on held-out episodes; recompute
under both L1 and L2 objectives and, on Push-T, in pixel space through the
released decoder. **Kill: if the interval for τ(H*) − τ(H_max) contains zero in
all four environments, the direction dies that day.** Code reuse:
`robojudge/src/robojudge/ranking.py` and the paired agreement bootstrap
unchanged; `tau_rows` must be parameterised away from `N_POLICIES`.

## Gate 3 — Recorded-outcome anchor for generated-video judges

**What it is.** Port the RoboJudge validity protocol to judges of generated
robot video, where no recorded outcome exists, by running the judge unchanged
on real episodes whose success is recorded. Route 3. Shipped as fields of the
`RoboJudgeAudit` card the prereg already promises.

**What the second pass found.** Three misses. The name "Judge Card" is taken
([2605.06161](https://arxiv.org/abs/2605.06161)); running a video judge on real
footage as a control is published ([Physion-Eval 2603.19607](https://arxiv.org/abs/2603.19607),
whose section 4.1 benchmarks ten judges, correcting the audit's claim that it
validates none); the aggregation complaint is in print
([RoboGaze 2606.28385](https://arxiv.org/abs/2606.28385)). Ours: the
recorded-outcome anchor, the d′/criterion split, the blind arm, the
prompt-disagreement rate, and Fisher-z intervals on DreamGen's eight published
correlations (already computed in the gate). Rename the deliverable.

**Week-1 step (10–20 GPU-h, one day).** Run DreamGen's own judge script
unchanged on our 5,106 RoboArena episodes: default prompt (A), harsh prompt (B),
default prompt through our frame pipeline (C), blind arm with one unrelated
frame (D), and our existing same-checkpoint run as baseline (E, d′ 0.840, on
disk). Per arm: d′, criterion, tie rate, ROC area, paired bootstrap clustered on
session. **Dies as a direction if A reaches d′ ≥ 0.60 with lower bound above
0.40, A and B agree on ≥ 90% of episodes, and D's interval includes zero.**
Narrows if only the first holds. Strong if A's upper bound is below 0.40 or A
and B disagree on more than 25%. Guard: if C lands far from A, it is a
frame-sampling effect, not a prompt effect; say so.

## Gate 4 — Sparsity-premise instrument

**What it is.** Our un-run `sparsityprem` design: a loss-band sampler,
coordinated-failure estimator and curvature probe that measure the "argued
sparsity" premise of [2606.29657](https://arxiv.org/abs/2606.29657) v2. The
premise test is Level 4 twice over (one citer worldwide, no empirical
follow-up, 76 papers in the ten-week pass, none relevant). The harness runs
(50/50 band samples in band, verified today).

**What the second pass found.** The item passes the method filter only through
the training rule it unlocks (measure the mutual information between each
training-visible feature and the safety-critical query set; decorrelate above
the measured threshold). That rule has a published, code-released neighbour,
[LARF 2507.18631](https://arxiv.org/abs/2507.18631) (EMNLP 2025, 31 citations),
so the rule is Level 3 and LARF is a mandatory baseline. And Arm C runs at one
correlation strength, so it cannot produce the dose-response curve the rule
needs: run four strengths, +300 models, +11 GPU-h (~81 total). The measured
threshold is a procedure, not a portable number; that goes in "what we will
not claim".

**The condition, one sentence to accept or reject:** "I approve the
sparsity-premise sweep on the condition that, before the pilot runs, the design
is edited to (a) name the feature-admission training rule as the method paper
it unlocks, (b) run Arm C at four correlation strengths, raising the budget from
~70 to ~80 GPU-h, (c) carry LARF as a mandatory baseline for that rule, and (d)
record the source version strings 2606.29657v2 and 2607.07538v2 in DESIGN.md;
and I accept that the study can only refute the premise, never confirm it."
Rejecting it fails the method filter.

## Owner decision and week-1 runs (2026-09-05)

Owner, 2026-09-05, verbatim: "continue running more detailed gates, and may
be pre-registrations. and also devide directions to diagnostic and mothod.
for conditional, run the condition and write back results to wiki. I approve
the week-1 step". Read as: all four week-1 steps approved; Gate 4's condition
sentence accepted, so its design is edited first and then its pilot runs.

Launched the same day, one Opus agent per gate, all on OrangeGrid (both L40S
cards were idle): Gate 1 on card 0 (under 2 GPU-h), Gate 3 on card 1 (10–20
GPU-h), Gate 4 on card 0 after Gate 1 finishes (~10 GPU-h pilot), Gate 2 on
whichever card frees first after that (30–60 GPU-h). Each gate page gets its
own "Week-1 result" section when its run ends. The diagnostic-versus-method
split the owner asked for is at [[Direction-Classification-2026-09-05]]:
gates 1 and 2 are methods; gates 3 and 4 are diagnostics with named unlocks.

### Week-1 results as they land

- **Gate 1 (2026-09-05): DEAD. The direction closes.** In the one direction
  where the fitted rotation works well (LAION-2B into CLIP ViT-B/32: CIFAR-100
  rises 4.18 points, Recall@1 falls 1.23), spatial role decodability drops
  only 2.25 points at the headline anchor count, and refitting on all 16,654
  images drives the drop to −0.87 [−1.65, −0.08]. The whole interval sits
  below the 2-point line, which is §7.4's DEAD condition and §7.5's kill arm
  at once. The large drop in the other direction (18.71) comes with a 23.83
  point CIFAR-100 loss and is within 1.45 points of a random rotation, so it
  is a bad fit, not a boundary. A second finding kills the design itself: the
  map degrades the text tower in every pair and direction (9.48 to 33.76
  CIFAR-100 points when the text side is mapped), so no configuration carries
  categories well and roles badly, the lopsided residual §5.1 required.
  Measurement validated: self-map drop 0.00; probes reproduce the numbers of
  record exactly. Cost 12 GPU-minutes. Rating ★★ per §11. Full record:
  [[Gate-2026-09-01-canonicalization]], Week-1 result section.

- **Gate 2 (2026-09-06): DIES, provisionally.** On the pre-registered L2
  objective, the interval for τ(H*) − τ(H_max) contains zero in all three
  environments that could run: Push-T +0.200 [−0.311, +0.400], Wall +0.135
  [−0.267, +0.222], PointMaze +0.000 [0, 0]. Provisional only because the
  rule is written over four environments and Metaworld stayed blocked on the
  dataset licence gate. Two facts change the meaning without changing the
  verdict. First, the horizon effect is real but sits at the short end: on
  Wall, rank agreement goes from −0.449 at H = 1 to +0.719 at H = 5 (MMRV
  0.494 → 0.086), so the best horizon is near the long end and "stop early"
  cannot beat "run to the end"; a contrast chosen after seeing that is the
  selection the gate forbids. Second, with 36 held-out episodes the intervals
  are 0.5–0.7 wide against differences of 0.0–0.2, so this null is weak
  evidence, flagged before the run. Controls: L1 and Push-T pixel space do
  not reproduce the L2 shape. 1,800 rows, no duplicates, 30 pool jobs, about
  7 h wall. Rating ★★ by condition 1. Coordinator reading: the data says the
  untrustworthy horizons are the short ones, which is what SC3-Eval's drift
  truncation already handles, so a powered rerun of the same contrast is not
  recommended; the owner decides. Full record: [[Gate-2026-09-01-horizon]].
- **Gate 3 (2026-09-05): STRONG, with the frame guard fired.** The DreamGen
  judge exactly as shipped cannot tell success from failure on our 5,106
  RoboArena episodes: d′ 0.035 [0.003, 0.069]. The same weights and prompt
  fed through our video pipeline reach d′ 0.743 [0.648, 0.839]; our baseline
  run reproduces at 0.840. So 88% of the gap is frame handling (DreamGen
  rescales frames to 30%, 49 frames in 929 tokens versus 40 frames in 3,698)
  and 12% is the prompt. The blind arm is at zero (d′ −0.003, ties 99.9%).
  Two carry-forward findings: the shipped script silently drops its own
  `--zeroshot` flag, so every published harsh-prompt run used the neutral
  prompt; and the harsh prompt is more generous, not stricter. Cost 2.84
  GPU-h. The claim must be rewritten from "the prompt" to "the frame
  pipeline". Full record: [[Gate-2026-09-01-dreamgen]], Week-1 result section.

## Recommendation

Run Gate 1's week-1 step first (one day, under 2 GPU-h, everything cached; a
clean kill or a live method by tomorrow). Run Gate 3's arms inside the ICLR
timeline if the Cosmos rerun leaves the OrangeGrid card free before 2026-09-10.
Gate 4 starts the moment the condition sentence is accepted. Gate 2 waits for
Gate 1's answer and a free card; it is the largest and the most exposed to the
CVPR world-model clock.

## Related

[[Direction-Audit-2026-09-01]] · [[Method-Gates-Wave-3-2026-08]] ·
[[Unified-Direction-Ranking-2026-08]] · [[Prereg-RoboJudge-Audit]]

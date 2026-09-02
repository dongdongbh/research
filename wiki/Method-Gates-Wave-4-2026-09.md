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

# Method Gate — Horizon-calibrated world-model policy evaluator (2026-09-01)

This is the pre-registration gate for rank 2 of
[[Direction-Audit-2026-09-01]]. The audit evidence for the item is in
`research/.orchestrator/audit-20260901/standalone-b.md`, item 3. This page is
the **second, independent pass** that the prereg workflow requires. Every link
below was opened while writing this page.

## Words used on this page

- **World model** — a learned model that predicts what the world will look
  like after an action. Here it is used as a stand-in for a robot simulator.
- **Policy** — the program that chooses the robot's actions.
- **Policy evaluator** — using a world model to score and rank several
  policies, instead of running them on a real robot.
- **Rollout** — letting the model imagine one step after another. The
  **rollout length** or **horizon** `H` is how many steps it imagines before
  we score the result.
- **Rank correlation** — a number from −1 to +1 saying whether two lists put
  the same items in the same order. **Kendall's tau (τ)** is the version we
  use. τ = 1 means the two orders agree exactly.
- **Bootstrap** — re-draw the same data many times to see how much a number
  wobbles. It gives a **confidence interval**, the range the number would take
  if we ran the study again.
- **Off-policy evaluation (OPE)** — an older field that estimates how good a
  policy is from data collected by a *different* policy.
- **Scoop level** — how much of the idea is already published. Level 5 means
  no overlap. Level 1 means fully published. An item must beat Level 2.
- **GPU-h** — one hour of one graphics card.

## Headline

**SURVIVES, ★★★½ conditional. Level 3, at the bottom of Level 3.**

The premise is now confirmed by an outside paper rather than merely assumed:
[SC3-Eval](https://arxiv.org/abs/2606.18610) shows that cutting rollouts short
raises the agreement between a world-model leaderboard and the real one, from
Pearson r = 0.871 to r = 0.929 and Mean Maximum Rank Violation 0.151 → 0.119.
So there is a real horizon effect on ranking, and it is worth money.

**Loud problem, stated once.** That same paper is the closest competitor, and
the audit missed it. SC3-Eval already stops a rollout early when it drifts. Our
delta is therefore narrower than the audit believed, and we must never write
the sentence "nobody truncates rollouts in world-model evaluators." What is
still empty is the *curve* and the *estimator*: nobody plots rank agreement
against rollout length, nobody picks the length that maximises it, and nobody
puts a confidence interval on the answer.

## 1. The owner's method filter

**Route 3 — an old method used on a new problem.** The rating below is for the
reframed item, as the audit already established.

- **The old method** is horizon selection from off-policy evaluation. That
  field learned that trusting a model further into the future trades bias
  against variance and that there is a best place to stop. The reference
  mechanism is the MAGIC estimator of
  [Thomas and Brunskill (2016)](https://arxiv.org/abs/1604.00923), which
  introduces "a new way to mix between model based estimates and importance
  sampling based estimates." The growth of error with horizon is described in
  [Breaking the Curse of Horizon](https://arxiv.org/abs/1810.12429) (Liu, Li,
  Tang and Zhou, NeurIPS 2018) and in
  [Understanding the Curse of Horizon in Off-Policy Evaluation via Conditional Importance Sampling](https://arxiv.org/abs/1910.06508)
  (Liu, Bacon and Brunskill, ICML 2020).
- **The new problem** is modern latent and video world models used to rank
  robot policies. The horizon-selection machinery has never been carried
  across, and it has never been aimed at **rank** fidelity instead of value
  error.

**The shipped tool, named as the standing rule demands.** A small released
package, `horizoncal`, that takes a world model, a set of policies, and a set
of real anchor rollouts, and returns four things:

1. the curve of rank agreement τ against rollout length H;
2. `H*`, the rollout length where the model's policy ranking best matches
   reality, chosen on a validation split;
3. a bias-corrected ranking read off at `H*`;
4. a bootstrap confidence interval and a **"do not trust past H*"** flag.

That is an estimator people will run, not a diagnosis. It is also the **method
paper the RoboJudge audit is required to name**: §3 of
[[Prereg-RoboJudge-Audit]] promises an "evaluator report card with rank-flip
rate, blind floor, and minimum n, as a common standard." `H*` and the
"do not trust past" flag become two more fields of that same report card, and
one released tool serves both papers.

## 2. The second scoop pass

Six searches ran through the paper-search tool across Semantic Scholar,
OpenAlex, arXiv and Crossref: the two phrases the brief named
("rank correlation versus rollout horizon", "world model policy selection
horizon truncation"), the borrowed machinery ("off-policy evaluation truncated
horizon bias variance estimator selection"), the adjacent community's own words
("offline policy selection rank correlation benchmark estimator regret"), a
most-recent pass restricted to 2026, and a targeted pass on horizon-chosen-for-
ranking. A **citation-graph pass** pulled all 41 papers citing WorldGym.

**The OpenReview connector was unavailable** (`openreview-py` is not installed
in the search environment), so that source was covered by a web search of
openreview.net instead. State this limitation if the gate is ever challenged.

### What the audit missed

- **[SC3-Eval](https://arxiv.org/abs/2606.18610)** (2026-06-17; Tseng, Hussein,
  Dong, Ren, Shi, Wang, **Sergey Levine**, Li). **The closest prior work, and
  the reason the star rating dropped.** Deep-read from the PDF. It "reuses the
  inverse dynamics mode at inference as a per-action-chunk uncertainty signal
  that terminates rollouts whose generated frames drift away from the requested
  actions." Its ablation table is the decisive passage: removing early
  termination moves Pearson r from 0.929 to 0.871 and MMRV from 0.119 to 0.151.
  **What it does not do:** it never reports agreement as a function of rollout
  length, it reports no confidence interval on either number, its stopping rule
  is per-rollout and learned rather than a single horizon chosen to maximise
  rank agreement, and the mechanism only exists inside a world model they
  retrained with forward–inverse consistency. Its own limitations section says
  the evaluator "is also trained and validated on short-horizon manipulation,
  with a single table-bussing episode lasting roughly 20 seconds," and expects
  "accumulated drift" at longer horizons. That sentence is our opening.
- **[MMRV](https://arxiv.org/abs/2405.05941)** (Mean Maximum Rank Violation,
  from SIMPLER; Li, Hsu, Gu, Pertsch, Mees, Walke and others). A rank-agreement
  measure that already exists in this exact field. **It must be a reported
  metric in our tool alongside Kendall's τ**, or a reviewer will ask why we
  invented our own.
- **[WorldArena](https://arxiv.org/abs/2602.08971)** (Tsinghua FIB Lab; code at
  [tsinghua-fib-lab/WorldArena](https://github.com/tsinghua-fib-lab/WorldArena),
  leaderboard at
  [WorldArena/WorldArena](https://huggingface.co/spaces/WorldArena/WorldArena)).
  Deep-read from the PDF. Its "Embodied Policy Evaluator" task trains **five**
  policy models and reports a **Pearson r with no confidence interval** (the
  figure gives r = 0.986 for one model and r = 0.483 for another). No rollout
  length is varied. This is the same weakness our RoboJudge audit documents,
  now sitting in a public leaderboard's own metric.
- **Correction to the audit's clock.** The audit says there is a live CVPR 2026
  World Model Track. The precise fact is that the **WorldArena Challenge at
  CVPR 2026 closed on 2026-06-30**; its submission page says the challenge "has
  officially concluded" and points to a coming **WorldArena Challenge 2.0**. The
  leaderboard stays open for local evaluation. So the clock is real but it is
  the *next* challenge, not this one.
- **A second framing paper.**
  [How Should World Models Be Evaluated for Embodied Decision-Making?](https://arxiv.org/abs/2606.15032)
  proposes an L0–L7 ladder listing "policy-ranking agreement", "closed-loop
  rollout validity" and "uncertainty calibration" as things to report. It joins
  [Validate the Dream Before You Trust Its Verdict](https://arxiv.org/abs/2607.07196)
  and its L0–L4 ladder. **Two independent ladder papers now exist.** The
  framing is firmly taken; a checklist is not an estimator, but we must present
  our work as filling a named rung, never as noticing the problem first.
- **The same group is still moving.** The authors of the L0–L4 ladder published
  [Imagined Rollouts are Kinematic, Not Dynamic](https://arxiv.org/abs/2607.05966)
  on 2026-07-07, a per-step diagnostic of long-horizon world-model failure. Two
  papers in three months from one group on horizon-dependent world-model
  trust. They are the most likely people to publish our curve.
- **Nothing in the 41 WorldGym citers scoops the estimator.** The nearest are
  [StressDream](https://arxiv.org/abs/2606.00267) (steers imagination toward
  high-impact futures; a sampling method, not a horizon method),
  [WorldSimProbe](https://arxiv.org/abs/2608.09298) (five diagnostic suites for
  action fidelity; no ranking, no horizon), and
  [SimVerity](https://arxiv.org/abs/2608.25067) (simulation-to-deployment
  verdict transfer for smart-home agents; different domain).

### The premise re-verified against the source paper

The audit's load-bearing claim is that WorldGym asserts rankings are preserved
and never tests it against horizon. **I re-checked this myself from the PDF and
it holds, with one sharpening.**

- The claim: "we show that WorldGym is able to preserve relative policy
  rankings across different policy versions, sizes, and training checkpoints."
- The evidence is qualitative across three policies (RT-1-X, Octo, OpenVLA)
  plus checkpoint sequences: "the relative performance rankings between RT-1-X,
  Octo, and OpenVLA are the same." The single correlation reported is
  **Pearson r = 0.78, with no confidence interval**.
- **Sharpening.** WorldGym does use the word "horizon", but it means the
  diffusion *prediction* horizon, the number of frames denoised in parallel,
  and it is set equal to the policy's action-chunk size for speed. The paper's
  only horizon sweep is a timing table. The rollout length is a separate
  required input, `N_rollout` in its Algorithm 1, and it is **fixed and never
  varied**. That fixed input is exactly the free parameter our tool calibrates.

### Verdict of the second pass

**Level 3 — medium overlap, at the bottom of the band.** Against SC3-Eval, two
of four axes match (problem framing: a world model ranking robot policies;
application domain: real manipulation policies) and two differ (core mechanism:
a global horizon chosen to maximise rank agreement, post hoc on a frozen model,
versus a learned per-rollout drift threshold inside a retrained model; key
insight: a bias–variance trade-off in rank space, versus off-manifold drift
detection).

**Delta, one sentence.** Unlike SC3-Eval, which terminates each imagined
rollout when a learned action-consistency signal says it has drifted, inside a
world model it retrains for that purpose, our work measures rank agreement as a
function of rollout length on any frozen world model, picks the single length
that maximises it under validation-locked selection, and returns the ranking
with a bootstrap interval and an explicit "do not trust past H*" flag.

## 3. Live verification

Everything load-bearing was checked against the live source on 2026-09-01, not
recalled.

| Fact | Status | Evidence |
|---|---|---|
| jepa-wms repo and licence | **Confirmed** | [facebookresearch/jepa-wms](https://github.com/facebookresearch/jepa-wms), 468 stars, **CC-BY-NC 4.0**. The Hugging Face model card carries `license: cc-by-nc-4.0`. **Non-commercial, academic use only — this sentence must appear in any write-up.** |
| Which checkpoints actually download | **Confirmed, with sizes** | [facebook/jepa-wms](https://huggingface.co/facebook/jepa-wms): `jepa_wm_pusht` / `pointmaze` / `wall` 212 MB each, `jepa_wm_metaworld` 212 MB, `jepa_wm_droid` 2.75 GB. DINO-WM baselines 262–281 MB each. V-JEPA-2-AC baselines 3.66 GB and 5.27 GB. |
| Which environments | **Confirmed** | Push-T, PointMaze, Wall, Metaworld (DINOv2 ViT-S/14 encoder, predictor depth 6) and DROID/RoboCasa (DINOv3 ViT-L/16, depth 12). |
| Source paper | **Confirmed** | [What Drives Success in Physical Planning with Joint-Embedding Predictive World Models?](https://arxiv.org/abs/2512.24497) (Terver, Yang, Ponce, Bardes, **Yann LeCun**). |
| Datasets downloadable | **Confirmed** | [facebook/jepa-wms datasets](https://huggingface.co/datasets/facebook/jepa-wms) holds `pusht`, `point_maze`, `wall`, `metaworld`, `robocasa`, `franka_custom`; `src/scripts/download_data.py` fetches them. Raw DROID is 5.6–8.7 TB and **we will not pull it**. |
| A real ground-truth score exists | **Confirmed** | The planning evaluation runs the policy in the actual simulator and reports Success Rate over `cfg.meta.eval_episodes`, plus end-of-episode goal distance. Grid command: `python -m evals.simu_env_planning.run_eval_grid`. |
| What the model-side score is | **Confirmed** | The planner minimises the L2 distance between the goal embedding and the predicted embeddings across the planning horizon; L1 is configurable; planners are CEM and NeverGrad. |
| WorldGym released model size | **Confirmed** | `mixed_openx_9robots_20frames_0p1actiondropout_580ksteps.pt`, **about 9 GB**, served from Google Drive, not Hugging Face. Runners for OpenVLA, Octo, SpatialVLA and RT-1-X. |
| WorldGym licence | **Confirmed, and it is a problem** | The GitHub API reports **`license: null`** for [world-model-eval/world-model-eval](https://github.com/world-model-eval/world-model-eval) (107 stars, last push 2025-11-06). **No licence means evaluate it, do not fork it** — the same rule Gate 1 applied to CLIPSelf. |
| Does any of this run on an L40S | **Yes, comfortably** | Our OrangeGrid L40S has 48 GB (about 44 GB usable), 2 per node. The four small jepa-wms models are ~212 MB of weights each; the largest artifact in the whole set is 5.27 GB. WorldGym's 9 GB file is stored at full precision and needs roughly 4.5 GB of weights in bfloat16. All fit many times over. **Memory is not the constraint; rollout throughput is.** |

## 4. The week-1 decisive step

Written exactly as the audit sketched it, with the cost corrected and the two
statistical traps named.

**The step.** On jepa-wms, assemble **8–12 policies** by varying the planner
settings (CEM sample count, iterations, planning horizon, L1 versus L2
objective) and the checkpoint (`jepa_wm_*`, `dino_wm_*`). Score every policy
twice in each of the four small environments — Push-T, PointMaze, Wall,
Metaworld:

1. **Inside the world model**, at rollout lengths H = 1 … K, scoring by the
   planner's own objective, the latent distance to the goal embedding.
2. **In the real simulator**, using `run_eval_grid`, scoring by Success Rate.

Turn both sides into pairwise comparisons over matched episodes, fit both
rankings with Bradley–Terry, and plot Kendall's τ against H with our
session-level bootstrap. Report MMRV alongside τ.

**Kill criterion, a number fixed before the run.** If the bootstrap confidence
interval for **τ(H\*) − τ(H_max) contains 0 in all four environments**, there is
no horizon effect to calibrate and the direction dies that day.

**Two traps that would sink the result at review.**

- **Selection on the same data.** `H*` is picked as the best point on the
  curve, so the difference τ(H\*) − τ(H_max) is biased upward by construction.
  Apply the house rule from the lab's `honest-eval-stats` rules —
  **validation-locked selection**: choose `H*` on a
  fixed validation split of episodes, and report the difference only on the
  held-out split. Fix the split before running.
- **The cost function, not the model, may drive the curve.** The model-side
  score is a latent distance, and a flat τ(H) could mean the distance
  saturates rather than that the model is horizon-robust. Pre-register two
  controls: recompute the curve under both the L1 and the L2 objective, and on
  Push-T recompute it with the released VM2M decoder head
  (`dinov2_vits_224`) so the score is checked in pixel space.

**Cost — I am revising the audit's number upward and saying why.** The audit
says 20–40 GPU-h. The model side is cheap: the four small predictors are about
50 M parameters and latent rollouts cost almost nothing. The expensive term is
the **real-simulator side**, because each of the 8–12 "policies" is a planner
and every planning step queries the world model thousands of times. For four
environments × 10 policies × about 50 episodes, budget **30–60 GPU-h on
OrangeGrid L40S**, free, with 20–40 as the optimistic end. Time to a first
readable curve: one to two days.

**Code we reuse from robojudge**, named exactly:

- `src/robojudge/ranking.py` — `build_comparisons`, `win_matrix`, `bt_fit`,
  `bootstrap_bt` (session-level bootstrap, 10,000 draws, per-draw convergence
  and separation bookkeeping), `strength_cis`, `rank_flip_matrix`,
  `ridge_sensitivity`. Reused unchanged.
- `src/robojudge/judge/aggregate.py` — `paired_bootstrap_agreement`, which
  refits *both* rankings on the same resample "so the interval carries the
  correlation between the two sides instead of pretending they are
  independent." This is the exact statistic we need, and the τ(H) version is a
  small extension: resample once, refit the real and the model ranking at every
  H, and take the differences within the draw.
- `src/robojudge/review/core.py` — `tau_rows` and `spearman_rows`, which
  compute correlations per bootstrap draw from Bradley–Terry log-strengths.
  **One concrete code task:** `tau_rows` is hard-wired to the RoboJudge policy
  count through the module constant `N_POLICIES`, so it must be parameterised
  before it accepts 8–12 policies.
- `AGREEMENT_STATS` in the same aggregate module already holds Kendall,
  Spearman and Pearson. **Add MMRV** so we speak the field's own language.

New code needed: the world-model rollout-and-score loop at a settable H, the
`H*` selector with the validation split, and the MMRV function. That is a small
package, which is the point.

## 5. Baselines, venue, risks, and what it unlocks

**Systems we must compare against.**

- **No calibration** — score at the fixed rollout length the source repo uses.
  This is what WorldGym, WorldArena and RoboWorld all do.
- **SC3-Eval's early termination**, reimplemented as a drift threshold on the
  frozen model. We cannot use their retrained model, so we compare against the
  *idea* and say so plainly.
- **Longest horizon**, H_max, the naive "more imagination is better" choice.
- **An oracle horizon** picked on the test split, as an upper bound that shows
  how much validation-locked selection costs us.
- **MMRV and Pearson r without an interval**, the current field default, so the
  paper shows what the field would have concluded.

**Venue.** This is a method paper, not an audit, so it does not compete with
our ICLR slot, which [[Prereg-RoboJudge-Audit]] holds. Target **CVPR 2027 or
ICML 2027**. I did not verify either deadline; check before committing.
**WorldArena Challenge 2.0** is the nearer fixed point and is worth entering
with the tool, because the challenge's own policy-evaluator metric is an
interval-free Pearson r over five policies.

**Risks, loudest first.**

1. **The delta is one axis wide.** SC3-Eval already truncates rollouts and has
   Levine's name on it. If they add a horizon sweep in their camera-ready, our
   contribution shrinks to the interval. **48-hour re-gate on any new paper
   from that group or from the Betz group.**
2. **Famous artifacts on both sides.** LeCun is on the jepa-wms paper, Percy
   Liang and Sherry Yang on WorldGym, Levine on SC3-Eval, NVIDIA on StressDream.
   Our own standing lesson says ideas rooted in famous artifacts die in months.
3. **The curve may be flat.** That is the kill criterion, and it fires in one
   week for under 60 free GPU-h, which is the whole reason to run it.
4. **Non-commercial licence.** jepa-wms is CC-BY-NC 4.0 and WorldGym has no
   licence at all. Our released tool must be independent of both: it takes a
   world model as an argument and ships no weights.
5. **The owner's veto next door.** Part 3 of [[Unified-Direction-Ranking-2026-08]]
   records an **owner veto** on "seed-noise and variance of agent leaderboards"
   for being not significant, no barrier, and not novel. This item is a
   different shape and the difference must be stated up front: the deliverable
   is not a variance report but an estimator that changes a decision — it
   returns a horizon to use and a ranking to trust. If the owner reads it as
   the vetoed shape, the gate fails on the spot.
6. **Do not double-fund the asset.** Rank 7 of the audit, planning-budget
   allocation on jepa-wms, is benched and uses the same checkpoints. Keep them
   separate.

**What it unlocks.**

- It is **the method paper the RoboJudge audit is required to name**, closing
  the standing rule that every audit names the method it unlocks.
- One released tool serves both papers, and `H*` plus the "do not trust past"
  flag become fields of the evaluator report card already promised in
  [[Prereg-RoboJudge-Audit]] §3.
- It gives the lab a second evaluator-validation result on a completely
  different modality, which is the argument that our statistics work is a
  programme rather than one paper.

## 6. Rating

**★★★½, conditional. Write the pre-registration.**

The audit rated this ★★★★. I am lowering it half a star because the second
pass found SC3-Eval, which narrows the mechanism delta and puts Sergey Levine
in the lane, and because the two ladder papers close off the framing.

**The conditions, in order.**

1. **Week 1 must show the effect.** The rating rises back to ★★★★ if the
   bootstrap interval for τ(H\*) − τ(H_max) excludes 0 in **at least two of the
   four** environments, on the held-out split. It falls to **★★, kill**, if it
   contains 0 in all four.
2. **The write-up must name SC3-Eval as the closest prior work in the
   abstract**, and must never claim to be first to shorten a rollout.
3. **MMRV must be reported** next to Kendall's τ.
4. **The prereg must state the validation-locked selection rule for `H*`
   before any curve is plotted.**

## Related

[[Direction-Audit-2026-09-01]] · [[Prereg-RoboJudge-Audit]] ·
[[Unified-Direction-Ranking-2026-08]] · [[Method-Gates-2026-08]] ·
[[Method-Gates-Wave-3-2026-08]] · [[GPU-Resources-Across-Clusters]]

## Week-1 result (run 2026-09-05)

**Authorization.** Owner, 2026-09-05, verbatim: "continue running more detailed
gates, and may be pre-registrations ... I approve the week-1 step". That
authorizes §4 of this page as written.

**Licence, as §3 requires.**
[jepa-wms](https://github.com/facebookresearch/jepa-wms) is CC-BY-NC 4.0.
**Non-commercial, academic use only.**

### Headline

**DIES, provisionally. The kill number fires in all three environments that
ran.** The interval for `tau(H*) - tau(H_max)` contains zero everywhere on the
pre-registered L2 objective: Push-T +0.200 [-0.311, +0.400], Wall +0.135
[-0.267, +0.222], PointMaze +0.000 [+0.000, +0.000]. By condition 1 of §6 that
sends the rating to **★★**.

Provisional for one reason: §4 fixes the number over **four** environments and
Metaworld could not run, because the jepa-wms dataset on Hugging Face is behind
a licence gate only a person can accept. So this is three of four.

**Two things must be read alongside that verdict, and neither rescues it.**

First, **the curves are not flat**. On Wall rank agreement swings from **-0.449
at H=1 to +0.719 at H=5**, with Mean Maximum Rank Violation falling from 0.494
to 0.086 over the same stretch. That is a large horizon effect. It is simply not
the one §4 aimed at: the damage sits at **short** rollouts, not long ones, and
after H=5 the curve decays only gently to +0.494 at H=12. `tau(H*) - tau(H_max)`
asks whether stopping early beats running to the end, and when the best horizon
already sits near the long end that difference is small however dramatic the
curve. Swapping in a better contrast now would be the exact selection the gate's
own traps forbid, so the number stands as written.

Second, **the run could not have resolved the difference it tests**. With 36
held-out episodes the intervals are 0.5 to 0.7 wide while the differences under
test are 0.0 to 0.2. This was flagged before the run, not after. A null here is
weak evidence of no effect, not evidence of none.

The full numbers, the three curves, and the L1 and pixel-space controls are in
the task record at
`research/.orchestrator/tasks/gate2-horizon-wk1-20260905-01/result.md`.

### What ran

1,800 scored episodes: 3 environments x 10 policies x 60 episodes, every cell
complete, no duplicates, and the matched-episode check passing across thirty
independently-run jobs. `H*` was chosen on 24 validation episodes and every
reported number comes from the other 36, which is the validation-locked rule
condition 4 of §6 demands.

The two pre-registered controls behaved as controls should. Neither the L1
objective nor Push-T in pixel space reproduces the L2 shape: L1 gives negative
or zero differences everywhere and pixel space is flat at +0.018. So the L2
point estimates are not an artefact of a saturating latent distance, but they
are not corroborated either, which is a further reason not to read them as an
effect.

### Words used in this section

- **Gated dataset** — a download that is public to read about but blocked until
  a person accepts its terms on the website once.
- **Marker file** — an empty file one job writes to tell another job that it has
  finished with a graphics card.
- **Smoke test** — a small run whose only job is to prove the code executes.
- **Separation** — when one policy beats another every single time, so the
  fitted strength wants to run off to infinity and the number you get comes from
  the penalty rather than from the data.

### What is ready

The code is a new small package at
`/anvil/projects/x-cis261253/code/wmhorizon/`, mirrored on OrangeGrid at
`/home/dli160/wmhorizon/`. It has three parts: the statistics, the driver that
scores policies on the graphics card, and the report that turns rows into the
verdict.

The statistics reuse RoboJudge unchanged, as §4 requires, with the one change
§4 asked for. `tau_rows` used to be wired to RoboJudge's own policy count; it now
takes that count as an argument, keeps the old value as the default so no
existing caller changes, and refuses rather than miscomputes if the width is
wrong. It was checked against `scipy.stats.kendalltau` at 7, 8, 10 and 12
policies.

The self-check passes, six tests. Two of them are the ones that matter: it
plants a horizon effect in fake data and demands the gate find it, and it plants
a flat curve and demands the gate kill it. A third checks that the interval for
the difference between two horizons is tighter than it would be if the two
horizons were treated as independent, which is the whole reason both rankings
are refit inside one resample.

Mean Maximum Rank Violation is implemented as
[SIMPLER](https://arxiv.org/abs/2405.05941) defines it and is reported next to
Kendall's tau, as condition 3 of §6 requires.

All three runnable environments build end to end on the processor: the dataset
loads, the simulator starts, the shapes are right. The model-side rollout was
also smoke-tested on the processor in all three, with fixed actions instead of a
planner. That check exists because the easiest thing to get silently wrong is
the bookkeeping around the rollout, not the rollout itself: how many frames of
context to carry, which rows of the unrolled sequence are predictions rather
than context, and whether the per-step score lines up with horizon `H`. All
three came back correct, with twelve finite scores.

### One statistical worry, checked and cleared

The simulator score is Success Rate, which is either 0 or 1. With ten policies
and a few dozen episodes that could easily have produced **separation**, and
then the intervals would have been set by the ridge penalty rather than by the
data, exactly the case RoboJudge's `estimand_label` exists to label.

Checked on simulated data of the same shape and size. The largest fitted
log-strength is **1.17**, against RoboJudge's separation threshold of **5**, and
**27 to 30 of every 30** bootstrap fits converge at the default ridge of 1e-6.
So the pre-registered default is kept, no deviation is needed, and the interval
will be an ordinary Bradley-Terry interval.

### The choices that were mine, and why

**Ten policies**, the same set in all environments: two world models
(`jepa-wm` and `dino-wm`), two planners (CEM and NeverGrad), four search budgets
from 300 samples for 30 iterations down to 30 samples for 5, and both the L1 and
the L2 objective. The search budget is the axis that does the real work. A
planner given a tenth of the search is genuinely worse, and that is what spreads
the simulator success rates apart. Without a spread the simulator ranking is
nearly a tie and a rank correlation against a tie measures nothing.

The planner's own planning horizon is deliberately **held fixed** at the
released value of 6. The repo uses the word horizon for two different things,
the planner's planning horizon and the rollout length, and only the second is
what this gate calibrates. Keeping them apart is worth more than one extra axis.

**K = 12 everywhere.** An episode is 30 simulator steps and the world model
advances 5 simulator steps at a time, so an episode is 6 model steps. K = 12
covers that and the same distance again beyond it, which is where drift should
start costing rank fidelity. The extra horizons are almost free: one imagined
rollout of length 12 is scored at every H from 1 to 12 off the same latents.

**60 episodes per environment, 24 for choosing and 36 for reporting.** The split
is drawn once from a fixed seed before any curve is plotted, and `H*` is chosen
only on the 24, which is the validation-locked rule condition 4 of §6 demands.
The split is uneven on purpose: the episode is the independent unit of the
bootstrap, and the number that gets judged comes from the reporting half, so the
power belongs there.

This was briefly cut to 30 when the measured cost put the design near 100
GPU-hours against the 30 to 60 §4 budgeted. That was the wrong call and was
overruled: the budget is a **cost** limit, and OrangeGrid pool time is free, so
the real limit is wall time. The answer is to spread the work over more idle
cards, not to measure fewer episodes. The 30 episodes already scored count
toward the 60, so nothing was thrown away.

### What is blocked, stated loudly

**The two marker-gated cards turned out to be unusable, and the run was moved.**
GPU 1 was released on schedule. Started there, the run made no progress at all:
the connection into holder job 1081798 is torn down after two to three minutes
of full-utilisation work, which is less than a single episode takes, and four
automatic restarts produced zero scored episodes. Detaching inside that job does
not help; both `setsid` and `tmux` were tried there and both die with the
connection. This is the lab's own standing rule that long GPU work on OrangeGrid
must be submitted rather than run inside the holder job.

The run is therefore three submitted jobs, one per environment. The submit file
asks the pool for L40S cards and **excludes the holder node**, so it cannot take
the two cards the marker protocol governs away from the other two gates. That
costs nothing: a dozen L40S nodes were sitting idle with two free cards each.

**The cost is higher than §4 estimated, and it changed the design.** Measured on
completed episodes of the most expensive policy: Push-T takes **258 seconds
inside the world model and 129 seconds in the simulator**, so about 6.4 minutes
for one policy on one episode, and Wall is the same. That says the planner
dominates and the pixel-space decoder is a small part of it. PointMaze is about
half, even with software rendering.

Averaged over the ten policies, whose search budgets differ by a factor of
sixty, the three-environment design would be roughly **100 GPU-hours at 60
episodes**, against the 30 to 60 this page budgeted. The episode count was cut
to 30, which brings it to about **45**. That is a real loss of power, not a free
saving.

**PointMaze hit a second, separate wall, and is now over it.** It is the one
environment that renders through mujoco-py, which needs an offscreen graphics
context. These compute nodes carry a **compute-only NVIDIA driver**:
`libnvidia-ml` and `libnvidia-cfg` are present, but there is no `libEGL_nvidia`
and no `libGL` anywhere, so hardware rendering is impossible and every episode
died with "Failed to initialize OpenGL". Mesa's software EGL does not rescue it
either, because mujoco-py's graphics-card path has to enumerate a rendering
device and mesa does not present one.

The fix is OSMesa, pure software rendering, which is ample for a 224x224 maze.
The catch is that the current conda mesa has dropped OSMesa; version 23.3.4
still ships it. With that, mujoco-py's processor build compiles and a real frame
comes back. PointMaze is resubmitted and running.

**Anvil was tried as a second site and abandoned.** The owner asked for longer
runs there, so Push-T and Wall were submitted to account `cis261253-ai` on the
`ai` partition. At submission every one of the 80 H100 cards across all 20 nodes
was allocated, **361 jobs were pending against 66 running**, and the scheduler's
own estimate for our job was **two weeks out**; shorter wall clocks did not move
it at all. Both jobs were cancelled. The environment there is built and verified
anyway, so the site can be picked up later without redoing the setup.

PointMaze was never submitted on Anvil. It is the only environment needing
mujoco-py and OSMesa, Anvil has no conda to install them from, and it already
runs on OrangeGrid, so building that stack twice buys nothing.

**Metaworld is out, and only a person can put it back in.** The jepa-wms
[dataset repository](https://huggingface.co/datasets/facebook/jepa-wms) is
**gated**. Our token is valid and authenticates, but every file returns 403 until
someone accepts the terms once on that page. That was not done here, because it
means agreeing to a non-commercial licence on the owner's behalf.

Push-T, PointMaze and Wall were saved by a mirror. jepa-wms states that those
three are re-hosted from [DINO-WM](https://github.com/gaoyuezhou/dino_wm)
unmodified, and DINO-WM's own archive is ungated at
[OSF](https://osf.io/bmw48/?view_only=a56a296ce3b24cceaf408383a175ce28). The
file sizes match the gated copies exactly, to the byte. Metaworld has no such
mirror, and the numbers it needs to scale actions and proprioception live only
in that dataset, not in the released checkpoint. The model checkpoints
themselves were never a problem: they are ungated on `dl.fbaipublicfiles.com`.

**So the kill number needs restating for three environments.** §4 fixes it at
"the interval contains 0 in all four". With three, a result that contains zero in
all three is a **provisional** kill and must be labelled that way until Metaworld
runs. Two of three excluding zero still meets condition 1 of §6 on its own terms.

**Nothing here weakens the scoop position.** [SC3-Eval](https://arxiv.org/abs/2606.18610)
remains the closest prior work and the write-up must still say so. Nothing on
this page claims to be first to shorten a rollout.

### How to finish it

The thirty jobs are already running and resume from their own output, so nothing
has to be restarted unless they are killed. To relaunch:

```
cd /home/dli160/wmhorizon && condor_submit og/shards.sub
```

Each job writes `runs/rows/<policy>/<env>.jsonl`, one line per episode, kept
separate because ten jobs appending to a single file on shared storage would
interleave lines. A complete run is 600 rows per environment, being 10 policies
times 60 episodes. Merge before analysis:

```
mkdir -p runs/merged
for e in pusht wall pointmaze; do cat runs/rows/*/$e.jsonl > runs/merged/$e.jsonl; done
```

The analysis keeps only episodes that every policy finished, and each job works
in blocks of five episodes, so a partial run is still readable.

Then, from `/anvil/projects/x-cis261253/code/wmhorizon`:

```
uv run python -m wmhorizon.report --rows <dir> --out runs/report --draws 10000
```

Run it in the background and split it by environment and objective with the
`--envs` and `--scores` flags, because those pairs are independent. Each
bootstrap draw costs about 0.23 seconds, so one environment and one objective is
roughly 75 minutes at 10,000 draws. Do not let numpy open a thread per core on
these 10-by-10 matrices; it made a draw four times slower, and `report.py` now
pins the maths libraries to one thread itself.

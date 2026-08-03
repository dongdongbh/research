# Pre-registration: Do Robot Policy Evaluators Recover the Human Ranking?

Status: **DRAFT v1, 2026-08-03 — for professor sign-off.** Locks after the
week-1 go/no-go (§8). Once locked, hypotheses, arms, metrics and decision
rules do not change; any deviation is reported as a deviation.

Paper type: **diagnostic** (with a constructive method half per standing
rule 5). Target venue: ICLR 2027 (abstracts Sep 18, full Sep 25). Companion
method paper: [[Prereg-Crop-Consistency-Distillation]].

---

## 1. The problem, in plain language

Evaluating a robot policy properly requires running it on a real robot many
times — slow, expensive, and impossible for most labs. So the field is
racing to replace real evaluation with **automated evaluators**: reward
models that score episodes from video, VLM "judges" that grade rollouts, and
world models that simulate rollouts so no robot is needed at all.

Every important decision downstream — which policy to deploy, which training
recipe won, which paper's method is better — is starting to flow through
these evaluators. **If the evaluators are wrong, the field's rankings are
wrong**, and nobody would know, because the evaluators are graded by the
same groups that build them.

## 2. Current research state

Eleven evaluator systems appeared in roughly ten months ([WorldEval](https://arxiv.org/abs/2505.19017),
[WorldGym](https://arxiv.org/abs/2506.00613), [Ctrl-World](https://arxiv.org/abs/2510.10125), [RobotArena∞](https://arxiv.org/abs/2510.23571), [PolaRiS](https://arxiv.org/abs/2512.16881), [SC3-Eval](https://arxiv.org/abs/2606.18610), [RoboWorld](https://arxiv.org/abs/2607.01060), [dWorldEval](https://arxiv.org/abs/2604.22152),
[PiL-World](https://arxiv.org/abs/2606.05773), [GigaWorld-1](https://arxiv.org/abs/2607.02642), [RoboReward](https://arxiv.org/abs/2601.00675)). Each validates itself, on tiny policy
sets: SC3-Eval reports "Pearson correlation of 0.929" on **seven** policies;
RoboWorld reports "Pearson's r = 0.989" similarly. **No independent audit of
any of them exists** (lane-wide sweep, Aug 2026).

Three facts define the opening:

1. A position paper ([arXiv 2606.15032](https://arxiv.org/abs/2606.15032)) names exactly the missing checks —
   "policy-ranking agreement," "model exploitability," "uncertainty
   calibration" — and runs none of them ("zero experiments").
2. The audit *genre* is proven publishable in the text/code domain: a
   tool-calling-benchmark validity audit ([2607.02577](https://arxiv.org/abs/2607.02577)) found an 18.5%
   evaluator–human misalignment rate. Ours is the embodied instance.
3. **The ground truth already exists in public.** [RoboArena](https://arxiv.org/abs/2506.18123) released its raw
   evaluation dumps ([`RoboArena/DataDump_02-03-2026`](https://huggingface.co/datasets/RoboArena/DataDump_02-03-2026), MIT): 3,284 double-blind
   pairwise human comparisons, 9,589 real-robot episodes across 15 policies,
   with per-episode `binary_success`, `partial_success`, videos, and
   free-text rationales. **Corrected against the parsed dump (2026-08-02,
   pre-lock): exactly 7 policies have ≥600 episodes (1,068–1,431); the 8th,
   `pi05_droid`, has 564. The earlier "8 policies with 600–1,650" was wrong.
   The ≥600 RULE stands; the count is corrected — we do not bend thresholds
   to recover expected counts.** Builders cannot
   credibly audit themselves; we can, without a single robot.

## 3. Our method and novelty

**One pre-registered protocol, applied to multiple evaluators, against human
ground truth.** Novelty claims — **repositioned 2026-08-02 pre-lock**: the
confirmatory pass found **[2606.01036](https://arxiv.org/abs/2606.01036) (ICML 2026, Tian/Wu/Bajcsy)** already
uses RoboArena human pairwise labels as ground truth for three reward models
(rollout-pair agreement 0.72–0.77 on easy tasks → 0.52–0.62 on Tool Use)
— so "first to use the dump as evaluator ground truth" is CEDED; it becomes
motivating prior evidence, cited as such. Everything below remains verified
unclaimed (their study has no policy leaderboard, no RoboReward/VLM-judge
family, no blind floor, no CIs, no injection, no judge-swap; RoboArena's 68
and RoboReward's 37 citing papers contain zero independent audits):

- **First policy-level rank-validity audit** of robot policy evaluators:
  does the evaluator's induced policy RANKING (leaderboard, not per-pair
  agreement) match the human preference ranking, with honest uncertainty at
  realistic n?
- **The blind floor:** how much of the reported agreement is recoverable by
  an evaluator that never sees the pixels (language/duration priors only)?
  Nobody has measured this for any embodied evaluator.
- **The n-sensitivity result:** bootstrap subsampling to n=7 policies from
  our n=8-with-600-episodes to compute the CI that the field's r≈0.93-style
  claims actually carry. This single figure recalibrates every evaluator
  paper in §2 at once.
- **Method half:** a rank-calibrated evaluator — recalibrate RoboReward's
  episode scores against the ranking objective on held-out policies — plus
  the "evaluator report card" (rank-flip rate, blind floor, minimum-n)
  as a releasable standard.

## 4. Pre-registered design

**Fixed corpus:** RoboArena dump; primary set = the policies with ≥600
episodes (measured 2026-08-02: exactly **7** — see §2 correction); the human
ranking is computed once from pairwise preferences via Bradley–Terry
(ties = half-win each side; session-level bootstrap, 10,000 draws), before
any evaluator is run, and frozen. **Frozen 2026-08-02** (regenerated post-review with corrected flip
semantics; θ bitwise identical): robojudge `runs/ranking_freeze/2026-08-02/`
— top-1 `pi0_fast_droid` stable in **94.0%** of draws, bottom-1 in 100%;
adjacent-pair flip probabilities 2↔3 = 0.38, 3↔4 = 0.44, 4↔5 = 0.22 —
the middle of the human ranking is statistically indistinguishable.

**Evaluator arms:**
- A1: RoboReward-4B and -8B (released weights, `teetone/*`).
- A2: off-the-shelf VLM judges — Qwen2.5-VL-7B/72B, InternVL3-8B, plus a
  SigLIP2-similarity scorer from our cached-feature stack. Fixed prompt
  template, temperature 0, 3 sampled frames/view unless the model ingests
  video natively.
- A3 (confound battery): language-only (no pixels), first-frame-only,
  shuffled-frames, instruction-swapped, duration-only regression.
- A4 (degenerate-episode injection, constructed offline from the dump):
  instruction–video mismatch pairs; no-progress segments extracted from
  failed episodes presented as complete episodes. An evaluator should score
  these at the bottom.
- A5 ([DreamGen](https://arxiv.org/abs/2505.12705) component): regenerate DreamGen Bench videos for the 4
  published video models; score under 5 judges (A2 set + Cosmos-Reason);
  hold the published downstream policy-success vector fixed; test whether
  the "strong correlation" claim survives judge swap.

**Primary metrics:** Kendall τ and Spearman ρ vs the human ranking with
bootstrap CIs; MMRV (introduced by
[SIMPLER (2405.05941)](https://arxiv.org/abs/2405.05941), used by SC3-Eval
— for comparability; attribution corrected 2026-08-02); top-1 flip
rate under evaluator swap; blind floor = τ(A3-language-only)/τ(A1).

**Hypotheses (directional, locked):**
- **H1:** the top-1 policy is NOT invariant across evaluator arms (predict:
  at least one flip among A1–A2).
- **H2:** blind floor ≥ 0.5 (predict TRUE — priors carry most agreement).
- **H3:** the n=7 bootstrap CI on Pearson r contains 0.5 (predict TRUE).
- **H4:** ≥1 evaluator scores ≥1 degenerate episode class above the median
  real episode (predict TRUE).
- **H5:** [RoboRewardBench](https://crfm.stanford.edu/helm/robo-reward-bench) benchmark accuracy does not rank the arms by
  rank-fidelity τ (predict TRUE — benchmark accuracy ≠ ranking validity).
- **H6 (DreamGen):** the video-model ranking is not judge-invariant, and the
  downstream correlation across judges spans ≥0.4 width.

**Decision rules:** each H is scored by its pre-stated statistic with 95%
bootstrap CIs; Holm correction across H1–H6. **A full null (evaluators are
rank-valid, floors low, no degenerate failures) is the publishable positive
result** — the first independent confirmation the field would have.

**What we will NOT claim:** anything about evaluators/policies not tested;
anything about closed-loop deployment; causal claims about *why* an
evaluator fails beyond the confound battery's factors.

## 5. Expected outcomes

- **Likely (per H1–H4):** rankings partially evaluator-dependent; a sizable
  blind floor; wide n=7 CIs; at least one degenerate failure → headline:
  "current robot-evaluator agreement numbers are substantially prior-driven
  and statistically under-powered," plus the report card + calibrated
  evaluator that fixes the measurable part.
- **Null branch:** evaluators robustly recover the human ranking → headline:
  first independent validation; report card still ships; the n-sensitivity
  figure still corrects the field's reporting practice.
- Either way the artifact (`RoboJudgeAudit` harness + report card + the
  Bradley–Terry human-ranking reference) is reusable by every future
  evaluator paper.

## 6. Resources and timeline

Cost: **250–500 GPU-h, inference-only** (A5 adds 60–120 GPU-h of video
generation). Cluster: OrangeGrid (2×A100/L40S, free, no time limits); Anvil
A100 for the 72B judge if needed. Storage: RoboArena dump size **unverified**
— this is the week-1 go/no-go; stage on `$SCRATCH`, manifests per
[[Data-Transfer-Between-Clusters]].

Wk1 gate + ranking freeze → Wk2–3 A1/A2 → Wk4 A3/A4 → Wk5 A5 → Wk6 analysis
+ packaging → Wk7 write-up. Abstract (Sep 18) needs only the frozen protocol
+ the H3 n-sensitivity figure, computable in week 1 from the dump alone.

## 7. Risks and scoop watch

- RoboArena dump too large / labels insufficient for Bradley–Terry → fall
  back to the 2025-08 dump (7,513 files) or per-episode binary success.
- The evaluator authors' orbit (Levine/Finn/Pertsch/Liang students) could
  self-audit — 6-week re-gate clock; watch citations of [2606.15032](https://arxiv.org/abs/2606.15032) and the
  tool-calling audit.
- Coverage caveat from the gate: venue-native proceedings under-swept; run
  one clean confirmatory search before locking (§8).

## 8. Week-1 go/no-go checklist (locks the prereg)

1. ~~Verify dump size + hydration; parse one session end-to-end.~~ DONE
   2026-08-02: 18.5 GB verified via API pre-download; 3,284/3,284 sessions
   parse cleanly (0 failures); preferences exactly {A:1404, B:1401, TIE:479}.
2. ~~Compute the frozen human ranking + bootstrap CIs; produce the H3
   figure.~~ DONE 2026-08-02 (robojudge `runs/ranking_freeze/2026-08-02/`,
   `runs/h3_figure/2026-08-02/`; regenerated post-review with
   unit-variance normal scores). H3 n=7 bands: raw-strength scale
   **[0.51, 0.96]** for r*=0.929 (does NOT contain 0.5 — the earlier
   [0.40, 0.95] used under-dispersed scores); copula scale [0.86, 0.99].
   Abstract phrasing: "a true r*=0.929 yields measured r anywhere in
   [0.51, 0.96] at n=7."
3. **PIN BEFORE LOCK: the H3/H3-test Pearson scale** — raw-strength
   (attenuated by the binning-droid outlier; prereg prediction holds) vs
   copula/normal-scores (unbiased; prediction flips). Both computed and
   stored in `h3_draws.npz`. Decide with the professor; record the choice
   and rationale here.
4. **PIN BEFORE LOCK:** keep `paligemma_binning_droid` in the primary set
   (it clears ≥600 but is a near-degenerate outlier, 17W/511L, θ=−2.47,
   which dominates the strength marginal) — default: keep, per the rule.
5. ~~Confirmatory literature pass (recent 8 weeks explicitly).~~ DONE
   2026-08-02: partial threat [2606.01036](https://arxiv.org/abs/2606.01036) → repositioned (§3); RoboWorld v4
   now claims r=0.989 (n-sensitivity figure sharpened); ArmnetBench
   ([2607.24481](https://arxiv.org/abs/2607.24481), 3,118 human-scored episodes) is a scoop-enabler — window
   argument strengthened. **Caveat: OpenReview rate-limited during the
   sweep — run one manual OpenReview pass (ICLR'27/NeurIPS'26 submissions)
   before lock.**
6. Professor sign-off → mark this page LOCKED with date + git hash.

## Related

[[Unified-Direction-Ranking-2026-08]] · the removed autoresearch accept-rule draft (git history) ·
[[GPU-Resources-Across-Clusters]] · [[Data-Transfer-Between-Clusters]]

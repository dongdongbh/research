# Pre-registration: Do Robot Policy Evaluators Recover the Human Ranking?

Status: **DRAFT v1, 2026-08-03 — for professor sign-off.** Locks after the
week-1 go/no-go (§8). Once locked, hypotheses, arms, metrics and decision
rules do not change; any deviation is reported as a deviation.

Paper type: **diagnostic** (with a constructive method half per standing
rule 5). Target venue: ICLR 2027 (abstracts Sep 18, full Sep 25). Companion
paper: [[Prereg-Autoresearch-Accept-Rule]] (method).

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

Eleven evaluator systems appeared in roughly ten months (WorldEval,
WorldGym, Ctrl-World, RobotArena∞, PolaRiS, SC3-Eval, RoboWorld, dWorldEval,
PiL-World, GigaWorld-1, RoboReward). Each validates itself, on tiny policy
sets: SC3-Eval reports "Pearson correlation of 0.929" on **seven** policies;
RoboWorld reports "Pearson's r = 0.989" similarly. **No independent audit of
any of them exists** (lane-wide sweep, Aug 2026).

Three facts define the opening:

1. A position paper (arXiv 2606.15032) names exactly the missing checks —
   "policy-ranking agreement," "model exploitability," "uncertainty
   calibration" — and runs none of them ("zero experiments").
2. The audit *genre* is proven publishable in the text/code domain: a
   tool-calling-benchmark validity audit (2607.02577) found an 18.5%
   evaluator–human misalignment rate. Ours is the embodied instance.
3. **The ground truth already exists in public.** RoboArena released its raw
   evaluation dumps (`RoboArena/DataDump_02-03-2026`, MIT): 3,284 double-blind
   pairwise human comparisons, 9,589 real-robot episodes across 15 policies
   (8 policies with 600–1,650 episodes), with per-episode `binary_success`,
   `partial_success`, videos, and free-text rationales. Builders cannot
   credibly audit themselves; we can, without a single robot.

## 3. Our method and novelty

**One pre-registered protocol, applied to multiple evaluators, against human
ground truth.** Novelty claims (each verified unclaimed in the Aug 2026
lane sweep):

- **First independent rank-validity audit** of any robot policy evaluator:
  does the evaluator's induced policy ranking match the human preference
  ranking, with honest uncertainty at realistic n?
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

**Fixed corpus:** RoboArena dump; primary set = the 8 policies with ≥600
episodes; the human ranking is computed once from pairwise preferences via
Bradley–Terry, with bootstrap CIs, before any evaluator is run, and frozen.

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
- A5 (DreamGen component): regenerate DreamGen Bench videos for the 4
  published video models; score under 5 judges (A2 set + Cosmos-Reason);
  hold the published downstream policy-success vector fixed; test whether
  the "strong correlation" claim survives judge swap.

**Primary metrics:** Kendall τ and Spearman ρ vs the human ranking with
bootstrap CIs; MMRV (SC3-Eval's own metric, for comparability); top-1 flip
rate under evaluator swap; blind floor = τ(A3-language-only)/τ(A1).

**Hypotheses (directional, locked):**
- **H1:** the top-1 policy is NOT invariant across evaluator arms (predict:
  at least one flip among A1–A2).
- **H2:** blind floor ≥ 0.5 (predict TRUE — priors carry most agreement).
- **H3:** the n=7 bootstrap CI on Pearson r contains 0.5 (predict TRUE).
- **H4:** ≥1 evaluator scores ≥1 degenerate episode class above the median
  real episode (predict TRUE).
- **H5:** RoboRewardBench benchmark accuracy does not rank the arms by
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
  self-audit — 6-week re-gate clock; watch citations of 2606.15032 and the
  tool-calling audit.
- Coverage caveat from the gate: venue-native proceedings under-swept; run
  one clean confirmatory search before locking (§8).

## 8. Week-1 go/no-go checklist (locks the prereg)

1. Verify dump size + hydration; parse one session end-to-end.
2. Compute the frozen human ranking + bootstrap CIs; produce the H3 figure.
3. Confirmatory literature pass (recent 8 weeks explicitly).
4. Professor sign-off → mark this page LOCKED with date + git hash.

## Related

[[Unified-Direction-Ranking-2026-08]] · [[Prereg-Autoresearch-Accept-Rule]] ·
[[GPU-Resources-Across-Clusters]] · [[Data-Transfer-Between-Clusters]]

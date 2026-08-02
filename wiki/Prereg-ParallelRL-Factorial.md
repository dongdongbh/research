# Pre-registration: Which Ingredients of Massively-Parallel Off-Policy RL Actually Matter?

Status: **DRAFT v1, 2026-08-03 — for professor sign-off.** Replaces
Prereg-Autoresearch-Accept-Rule as the ICLR method flagship (owner decision:
statistical-hygiene studies read as engineering, not research — recorded in
[[Prereg-Autoresearch-Accept-Rule]]). Locks after the week-1 reproduction
gate (§8). Target venue: **ICLR 2027** (abstracts Sep 18).

Paper type: **mechanism arbitration whose deliverable is a method** (the
minimal recipe). Companion diagnostic: [[Prereg-RoboJudge-Audit]].

---

## 1. The problem, in plain language

A year ago, training a humanoid controller took days. Now it takes minutes:
FastTD3 solves HumanoidBench tasks "in under 3 hours on a single A100," and
its successor trains locomotion "in just 15 minutes with a single RTX 4090."
This speed is transforming robot learning — every sim-RL group is adopting
these recipes.

But **why** it works is unresolved, because the recipes ship as bundles.
FastTD3's own abstract lists its ingredients as "parallel simulation,
large-batch updates, a distributional critic, and carefully tuned
hyperparameters" — four confounded factors, one of which is literally
"tuning." And the field's two mechanism papers **contradict each other**:

- **BRC** (2505.23150) says high-**capacity** critics are the lever.
- **Compute-Optimal Scaling** (2508.14881) says the opposite knob matters:
  "increasing the batch quickly harms Q-function accuracy for **small**
  models, but this effect is absent in large models" ("TD-overfitting").

FastTD3 ships **large batches with a small critic — the exact cell the two
theories disagree about**, in production, adopted everywhere. Nobody has
crossed the factors.

## 2. Current research state

Five overlapping causal claims, published within 12 months by overlapping
author sets, none tested against the others under one protocol: FastTD3's
four bundled ingredients; minimalist reward functions (2512.01996); BRC's
capacity claim; TD-overfitting's batch×capacity interaction; and the
replay-volume regime (2605.10236: non-uniform sampling matters only at low
replay volume). The group is walking these axes **serially, one per paper**
(Nauman co-authors BRC and the replay paper) — which is precisely why the
crossed design remains open and why its window shrinks with every month.

**Substrate (verified Aug 2026):** `younggyoseo/FastTD3` (456★, pushed
2026-05-16, reference configs published); `mujocolab/mjlab` (2,757★, pushed
within 24h); `google-deepmind/mujoco_playground` (2,125★, pushed within
24h); `carlosferrazza/humanoid-bench` (779★, 11 months stale — the one cold
piece, used as an eval suite only). All MuJoCo-family — **no Isaac, so no
RT-core restriction**; every run is single-GPU.

## 3. Our method and novelty

A **pre-registered crossed factorial around the authors' own published
center point.** Novelty: the first design that puts the five claims in one
protocol; the first direct test of the BRC-vs-TD-overfitting disagreement;
and the **minimal FastTD3 recipe** — the reduced config that keeps the
speed with the inert ingredients removed — as the method deliverable.

Why our lack of an RL track record is survivable (gate analysis): every
cell is a *documented perturbation of the authors' own released config*, so
"you tuned the baselines wrong" has no purchase; and the week-1
reproduction table (our re-run of each source paper's headline, with
deltas) converts the credibility deficit into evidence of care.

## 4. Pre-registered design

**Stage 1 — screening (resolution-IV fractional factorial).** Five factors:
critic capacity {FastTD3 default, BRC-large}; batch size {small, FastTD3-
large}; critic type {distributional, scalar}; replay volume {low, high};
reward {minimalist, shaped}. 16 cells × 4 tasks (2 HumanoidBench, 1
Playground, 1 mjlab) × 3 seeds ≈ 192 runs.

**Stage 2 — the arbitration.** Full 3×3 **capacity × batch** cross on the
two most sensitive tasks × 5 seeds, centered on FastTD3's shipped config.

**Hypotheses (directional, locked):**
- **H1 (the headline):** the capacity×batch interaction is positive — the
  benefit of large batches increases with critic capacity (TD-overfitting's
  prediction; BRC's story predicts capacity helps independent of batch).
- **H2:** at FastTD3's shipped critic size, the large-batch arm is NOT
  better than the small-batch arm at matched compute (predict TRUE per
  TD-overfitting; FastTD3's adoption implies the field assumes FALSE).
- **H3:** at least two of FastTD3's four bundled ingredients contribute less
  than seed noise on ≥3 of 4 tasks (a minimal recipe exists).
- **H4:** the reward factor does not interact with the architecture factors
  (minimalist rewards are separable, per 2512.01996's framing).
- **H5:** the surviving-recipe ranking is consistent across the three sim
  suites (if FALSE: recipes are suite-specific — publishable, changes the
  method's scope).

**Decision rules:** effect sizes with bootstrap CIs over seeds; Holm across
H1–H5; "contributes less than seed noise" pre-defined as |effect| < 1
pooled seed-σ. Matched compute per cell (same env-steps × updates budget).
All configs published (repo, tagged) **before** Stage-1 results exist.

**Kill criteria:** (i) week-1 reproduction of FastTD3's headline fails
beyond 3σ of its reported variance → stop, diagnose, and report; do not
proceed to the factorial on a base we cannot reproduce. (ii) A crossed
capacity×batch study on these suites appears before lock → re-gate within
48h (the standing competitor-re-identification rule).

**What we will NOT claim:** anything about Isaac-based or GPU-vectorized-
physics recipes (different systems regime); anything about on-policy PPO
pipelines; that FastTD3's authors' reported numbers are wrong (we reproduce
them first, and the factorial varies *their* config).

## 5. Expected outcomes

- **H1/H2 as predicted:** headline — "fast humanoid RL works despite, not
  because of, its large batches at shipped capacity; here is the
  interaction map and the cheaper minimal recipe." BRC-vs-TD-overfitting
  resolved in TD-overfitting's favor at deployment scale.
- **H1 reversed:** BRC's capacity story wins — equally publishable, and the
  minimal recipe changes accordingly.
- **H3 FALSE (all ingredients matter):** the bundle is vindicated — the
  first controlled confirmation, plus the interaction map the field lacks.
- Deliverables regardless: the minimal recipe + config, the reproduction
  table, and the released factorial harness with every config pre-published.

## 6. Resources and timeline

**Cost:** ~400–650 GPU-h, all single-GPU MuJoCo runs → **OrangeGrid**
(free, no time limits; 8–10 concurrent jobs ≈ 4–5 days wall-clock per
stage). No Anvil/Delta credits needed.
**Timeline:** Wk1 reproduction gate + config publication → Wk2–4 Stage 1 →
Wk4–5 Stage 2 → Wk6 analysis → Wk7 write-up. Abstract needs the
reproduction table + Stage-1 screening, in hand by Sep 18.

## 7. Risks and scoop watch

- **Nauman/Seo orbit publishing the cross** is the main risk and it grows
  monthly — this paper is now-or-never (recorded in the unified ranking).
  Watch arXiv listings weekly; 48h re-gate on any hit.
- HumanoidBench staleness → pin versions; Playground/mjlab carry the load.
- Reviewer "why should we trust your tuning" → answered structurally (§3);
  additionally, no per-cell retuning is allowed anywhere (rule stated in
  the paper).

## 8. Lock checklist

1. Professor sign-off on §4 (factors, H1–H5, noise threshold, no-retune rule).
2. Week-1 reproduction table complete; configs tagged and public.
3. Confirmatory literature pass, most recent 8 weeks explicit.
4. → LOCKED + git hash; deviations logged below this line.

## Related

[[Unified-Direction-Ranking-2026-08]] · [[Prereg-RoboJudge-Audit]] ·
[[Prereg-Autoresearch-Accept-Rule]] (declined predecessor) ·
[[GPU-Resources-Across-Clusters]]

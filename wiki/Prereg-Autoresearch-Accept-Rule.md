# Pre-registration: How Much of Autonomous ML Research Is Noise — and the Accept Rule That Fixes It

Status: **DRAFT v1, 2026-08-03 — for professor sign-off.** The noise-floor
protocol (§4, Act 1) must be locked BEFORE any campaign runs; the FDR
estimator and bake-off arms lock with it. Deviations reported as deviations.

Paper type: **method** (the accept rule is the deliverable), with the
diagnosis embedded as its motivation — the Aug 2026 lane gate showed neither
half stands alone. Target venue: ICLR 2027. Companion paper (diagnostic):
[[Prereg-RoboJudge-Audit]].

---

## 1. The problem, in plain language

AI agents now run machine-learning research autonomously: they edit training
code, run an experiment, and keep the change if the number improves. The
most popular such system — Karpathy's `autoresearch` (92.8k stars, 13.2k
forks) — accepts a change when **one single training run** improves
validation loss:

> "8. If val_bpb improved (lower), you 'advance' the branch, keeping the git
> commit
> 9. If val_bpb is equal or worse, you git reset back to where you started"

No seeds. No repeats. No threshold. But training runs are noisy — the same
code with a different random seed gives a different number. So some fraction
of every overnight run's ~100 "discoveries" are luck. **Nobody knows what
fraction.** Thousands of forks are running this loop; the discoveries feed
speedrun leaderboards and follow-on papers.

This is the classic multiple-comparisons problem — sequential testing
without error control — appearing at the heart of AI-run science.

## 2. Current research state

- **The noise is admitted but never quantified.** Karpathy's own logs judge
  "in the noise" by eye four separate times ("Essentially identical. The
  0.0002 bpb improvement is almost noise"; "All variants within noise") and
  the leaderboard doc warns "the CORE metric can be a little bit noisy."
  A second, unnamed noise source: the budget is fixed **wall-clock** (5
  minutes), so step count itself is a random variable.
- **The community asked for the fix and upstream declined it.** Issues #251,
  #278, #303, #466 complain about seed-luck; **#560 proposed a noise-floor
  accept rule and was closed unmerged.** The slot is documented-vacant.
- **Nearest prior art, verified at method level (Aug 2026 gate):**
  2606.05186 runs a seeded factorial screen on the same substrate but
  **never runs the agent loop** — no accept-rule FDR; AIRA_2 (2603.26499)
  states qualitatively that prior "overfitting" was "evaluation noise" —
  no number; 2602.07150 quantifies noise in *benchmark scoring* of agents,
  not in a research loop; Regimes (2606.10241) self-flags "seed 5 unadjusted
  for its sequential promotion structure" — names the hole, doesn't fill it;
  2511.19794 supplies paired-bootstrap machinery for human papers, never
  wired into an agent. **Nothing measures a research loop's noise floor,
  estimates the greedy rule's false-discovery rate, or proposes a
  statistically sound accept rule for LLM-driven experimentation.**

## 3. Our method and novelty

Three acts, one paper:

1. **Measure the noise floor** of the loop's objective — the first variance
   decomposition of an agentic-research reward (seed vs wall-clock-step
   vs hardware components).
2. **Estimate the false-discovery rate of the greedy accept rule** by
   replaying an agent's *actual accepted commits* (and a matched sample of
   rejections — false negatives too) with k seeds. First FDR number for
   autonomous ML research.
3. **The method: a sequential, variance-calibrated accept rule** (k-seed
   racing / SPRT with the pre-measured σ) that maximizes
   **discoveries-that-survive-held-out-replication per GPU-hour** — a new
   objective for agentic science — shipped as a drop-in `program.md` +
   harness for the 13.2k forks.

Novelty check (lane sweep, Aug 2026): all three components verified
unclaimed; the merged shape (diagnose→quantify→fix on the same substrate)
exists nowhere.

## 4. Pre-registered design

**Substrates (two, non-negotiable):**
- S1: `autoresearch` + nanochat (frozen upstream 2026-03-26 = stationary
  target), objective val_bpb, 5-minute single-GPU budget.
- S2: a GRPO tuning loop (TRL) on Countdown with a binary verifier at
  0.5–1.5B, objective eval pass-rate — same accept-rule question, different
  reward family. (Default; may be swapped for an AIDE/MLE-bench-style loop
  before lock, never after.)

**Act 1 — noise floor (locks first, runs first).** 30 identical repeats of
the unmodified baseline per substrate, crossed with {OG L40S, OG A100,
Anvil A100} and {wall-clock budget, fixed-step budget}. Estimands: σ_seed,
σ_step-count, σ_hardware via variance decomposition; report the full empirical
distribution, not just σ. **Structural coordinate registered up front**
(Arora rule): accepted-delta magnitude vs σ.

**Act 2 — greedy-rule FDR.** 8 campaigns × 2 agent backends (Claude Code,
Codex) × S1, upstream `program.md` verbatim; log every accept/reject. Replay
every accepted commit and a size-matched random sample of rejected commits
with k=5 seeds at fixed-step budget.
- **FDR estimand:** fraction of accepted commits whose k-seed mean Δ ≤ 0.
- **FNR estimand:** fraction of rejected commits with k-seed mean Δ > 2σ.
- Week-1 sanity gate: reproduce the upstream baseline number; report the
  reproduction delta in the paper.

**Act 3 — accept-rule bake-off at matched total GPU budget.** Arms:
- R0 greedy n=1 (upstream);
- R1 fixed k=3 mean;
- R2 sequential racing (successive elimination, σ-calibrated);
- R3 SPRT with pre-declared minimum effect δ = 1σ;
- R4 batch promotion with Benjamini–Hochberg.
Each arm: 3 campaigns × 2 substrates, identical total GPU budget.
**Primary endpoint:** discoveries surviving held-out k=5 replication, per
GPU-hour. **Secondary:** final objective at budget; campaign-level regret;
wall-clock overhead of each rule.

**Hypotheses (directional, locked):**
- **H1:** the median accepted delta in Act 2 is < 2σ_total (predict TRUE).
- **H2:** greedy FDR ≥ 25% (predict TRUE; report CI regardless).
- **H3:** wall-clock budgeting contributes ≥ 25% of total variance (predict
  TRUE; this is the free result nobody has named).
- **H4:** R2 or R3 dominates R0 on the primary endpoint in both substrates
  (predict TRUE).
- **H5:** the rule ranking is consistent across S1 and S2 (predict TRUE;
  if FALSE, the honest conclusion is that accept rules are
  substrate-specific — still publishable, changes the method's framing).

**Decision rules:** bootstrap CIs on all estimands; Holm across H1–H5; the
FDR threshold (Δ≤0) and δ=1σ are fixed now and cannot be tuned post hoc.

**Kill criteria:** (i) Act 1 shows σ negligible relative to typical accepted
deltas → premise dead; publish a short honest negative ("the greedy rule is
fine at this budget") and stop. (ii) Week-1 baseline reproduction fails
beyond 3σ → pivot to diagnosing that, do not proceed to Act 3.

**What we will NOT claim:** anything about frontier-lab internal loops;
anything about substrates not tested; that any specific published speedrun
result is false (we estimate rates, we do not audit individuals' entries).

## 5. Expected outcomes

- **Central expectation (H1–H4):** a large fraction of single-run
  "discoveries" don't replicate; the calibrated sequential rule finds fewer,
  realer improvements per GPU-hour → headline number in the abstract, plus
  a drop-in fix with the largest possible distribution channel.
- **Negative branch:** FDR low → "autonomous greedy search is surprisingly
  sound at 5-minute budgets," with the noise floor and variance
  decomposition as standing reference — still a paper, per kill-criterion
  (i)'s framing.
- Deliverables regardless: results database (every run, every seed),
  variance/power module (shared with the judge-audit paper and the collapse
  program), racing harness, `program.md` drop-in.

## 6. Resources and timeline

Cost: Act 1 ≈ 10 GPU-h · Act 2 ≈ 160 (campaigns) + 150 (replays) · Act 3 ≈
250–350 → **500–800 GPU-h total**, essentially all **OrangeGrid** (5-min
single-GPU jobs; persistent agent processes need OG's no-time-limit nodes).
API cost for agent backends ≈ $300–600.

Wk1: pre-register + Act 1 + baseline reproduction → Wk2–3: Act 2 campaigns +
replays → Wk4–5: Act 3 → Wk6: analysis + artifact packaging → Wk7: write-up.
Abstract needs Act 1 + a preliminary Act 2 FDR, both in hand by Sep 18.

## 7. Risks and scoop watch

- One motivated fork could publish #560's idea — start Act 1 immediately;
  the pre-registration timestamp is the defense.
- Framing risk: must read as a general result about agentic science under
  noisy objectives (autoresearch + S2 as instances), never as a repo
  takedown; tone follows the SVIB disclosure playbook.
- Scope collision resolved at the program level: the "statistical stopping
  rule" claim is OURS here and struck from the collapse direction's scope.
- Confirmatory literature pass (recent 8 weeks explicit) before lock.

## 8. Lock checklist

1. Professor sign-off on §4 (especially FDR threshold, δ, arms, S2 choice).
2. Act 1 protocol frozen; repeats launched.
3. Confirmatory sweep clean.
4. Mark LOCKED with date + git hash; deviations logged below this line.

## Related

[[Unified-Direction-Ranking-2026-08]] · [[Prereg-RoboJudge-Audit]] ·
[[GPU-Resources-Across-Clusters]] · [[Research-Automation-Tools]]

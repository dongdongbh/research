# Execution Plan for the Three Locked Projects (2026-08-08)

The slate's three deadline projects, each with: what is next, which
machine, what resources, and the week-by-week plan. Companion items
(SVIB→TMLR writing, sparsity design, audit boundary note) are listed at
the end. Total GPU need across the three: **~550–650 H100-hours**, which
under the card-scaling rule means multi-card allocations from the start.

## Machines — the BALANCED placement decision (revised Aug 8, owner request)

Assign each project to the cluster whose shape fits it, instead of
spending Anvil credits on everything:

| Project | Cluster | Why |
|---|---|---|
| RoboJudge | **OrangeGrid** (free) | Judge inference is one independent job per episode batch — HTCondor's exact profile; 7–8B judges fit L40S 48 GB; no wall limits; saves ~150 h of credits. Data: one 28 GB transfer. |
| 1-NFE | **Anvil H100** (credits) | Both codebases are JAX; JAX is verified ONLY on H100. The right place to spend credits because there is no alternative yet. |
| Contextualization | **Anvil (rewriting) + Delta H200 (the eight training runs), if Delta verifies** | Rewriting needs the vLLM recipe proven on Anvil; the training campaign hates 48-h walls and fits Delta's 30-day wall in one allocation. |

**Verification gates (cheap, run first):** (V1) JAX on OrangeGrid
cards — pass ⇒ 1-NFE sweeps may overflow to OG free; (V2) VLM-judge
throughput on one L40S — confirms RoboJudge's home (fallback: Anvil
tranche); (V3) Delta H200 environment (torch + data staging via
Globus) — **needs the owner once** (password+Duo SSH). Until V3 passes,
Delta is fallback-only and training runs stay on Anvil.

Original single-cluster rationale kept below for the record.

- **Anvil H100 (ai partition) — original primary rationale.**
  Reasons: all staged data lives on Anvil (`/anvil/projects/.../datasets`:
  RoboArena 28 GB, the 2B-token contextualization mixture, ImageNet
  reference stats, all checkpoints and caches); JAX is verified on H100
  (741 TF/s) but NOT on OrangeGrid A100 or Delta H200; and the vLLM
  repair recipe for these nodes is documented (`code/ctxprereg/README`:
  remove nvidia-cutlass-dsl, set CC to spack gcc, venv bin on PATH).
- **OrangeGrid** takes the overflow it is actually good at: the
  sparsity-premise sweep (300 short independent jobs) when approved, and
  1-NFE checkpoint sweeps IF JAX-on-OG verifies (a free check, queued).
- **Delta** stays out unless the from-scratch pair needs it; JAX-on-H200
  must be verified before anything depends on it.
- Standing rules apply: interactive salloc in tmux, MAX wall (48 h),
  `--gres=gpu:N` with N from ceil(queued-GPU-h ÷ 48), two push watchers,
  drain before release, per-worker env isolation.

## Project 1 — RoboJudge audit (ICLR: abstract Sep 18, paper Sep 25)

**State:** week 1 done — data parsed (3,284/3,284), human ranking frozen
with bootstrap uncertainty, four pinned choices awaiting the formal lock.
The Fisher-z motivation table (40/74 published intervals include zero) is
computed and banked.

**Resources:** ~100–150 H100-hours (judge inference over recorded
episodes; the blind-floor arms are text-only and nearly free). Data
already on Anvil.

**Plan:**
- **Week of Aug 11 — judge-arm bring-up (start now):** harness that
  feeds each evaluator the same episodes + task text and maps outputs to
  a policy ranking by the pre-stated rule; evaluators = RoboReward,
  Cosmos-Reason2, and 2–3 general VLM judges, each per its published
  recipe; blind-floor arm (no video) for every judge; smoke on a handful
  of episodes per judge.
- **Weeks Aug 18–29:** full judge runs; agreement analysis with CIs
  against the frozen human ranking (including the
  should-be-uncertain-where-humans-are test); robustness cuts.
- **Sep 1–17:** writing (motivation = Fisher-z table), boundary note vs
  the uncertainty audit, freeze, abstract by Sep 18.
- **Owner action:** the formal prereg LOCK (approve the four pinned
  choices, record the git hash) — needed before full runs start.

## Project 2 — 1-NFE diversity (CVPR ~Nov 13; deadline unconfirmed ±2 wk)

**State:** week-1 check passed; pipeline is ours end-to-end; Drifting
recall (unpublished anywhere) measured at reduced n.

**Resources:** sweeps ~100–150 H100-h; from-scratch matched pair ~200
H100-h (prereg §6 corrected: Anvil A100 does not exist for us — the pair
runs on Anvil H100, or Delta only after a JAX check).

**Plan:**
- **Week of Aug 11 (mostly CPU + small GPU):** resolve the Shortcut
  128-step FID anomaly (checkpoint provenance — test the alternate
  checkpoints, ask the author if needed; high-step numbers stay
  quarantined until resolved); verify whether MeanFlow/iMF/pMF ImageNet
  weights actually exist (the prereg's claim failed a first search);
  stage ROMS-IMLE as the second non-averaging model; implement the two
  measured speedups (cache reference features, parallel loader).
- **Weeks Aug 18 – Sep 5:** precision-matched recall sweeps at 50k
  samples across families (guidance/temperature matching to ±0.02);
  H1/H2 read-out per the pre-registered rules.
- **Sep 8 onward:** from-scratch matched pair (H3), then H4; analysis
  and writing through October. Re-check the CVPR date and SubFlow's
  repo in early September (standing risk).
- **Owner action:** the formal LOCK (the two caveats are recorded in the
  prereg's week-1 block).

## Project 3 — Epistemic contextualization (ICML ~Jan 28)

**State:** pipeline built, smoke-verified, 2B-token mixture staged and
contamination-clean.

**Resources:** ~165 H100-h for the eight training runs + 85–190 h
rewriting + ~8 h pilots ≈ **260–360 H100-hours**, the biggest single
consumer. Rewriting throughput is measured (13k tok/s at high
concurrency), so the schedule is real, not guessed.

**Three design choices — recommended defaults (professor can override
at the meeting; only #2 blocks the rewriting pass):**
1. Accept the revised budget (165 vs 100 GPU-h). *Recommended: yes* —
   it is measured, not estimated.
2. Rewriter yield 42–46%: *recommended: run the built `align` step* so
   long, name-dense documents are repaired rather than dropped (the
   drops are biased, which threatens the mixture's balance), and report
   both retained-and-aligned counts.
3. Probability-mass secondary metric for H1: *recommended: yes* — it was
   added precisely because the 1B base is at chance on ConflictBank.
- **Week of Aug 11:** the ~10 h of named setup (rewrite-prompt
  improvement round, the pre-registered fact-accuracy audit, LR check on
  validation) + the ~8 GPU-h pilots.
- **Weeks Aug 18 – Sep 12:** the C1/C3 rewriting passes (the long pole —
  schedule them to drain otherwise-idle GPU time behind the other two
  projects).
- **Sep 15 – Oct 24:** the eight training runs + eval; analysis and
  writing with a wide margin before the January deadline.

## Immediate actions (today)

1. Submit the Anvil allocation NOW and let it pend: **2×H100, 48 h**
   (the first ~96 GPU-h tranche; the queue refills it under the drain
   rule). Watchers attached.
2. Start the 0-GPU work in parallel: RoboJudge judge-arm harness,
   1-NFE anomaly/weights checks, contextualization setup round.
3. When the node lands: GPU 0 → RoboJudge judge smokes then full runs;
   GPU 1 → contextualization pilots, then 1-NFE sweeps; rewriting
   passes drain idle time.
4. Owner/professor: the two LOCKs (RoboJudge, 1-NFE) and a yes/override
   on the three contextualization defaults.

## Companion items (not gated on the above)

- **SVIB→TMLR writing** (~2–3 weeks, 0 GPU) — can start any time; the
  binding evidence and its §10 context section are ready to fold in.
- **Sparsity-premise sweep** — on professor approval, runs on OrangeGrid
  (~70 GPU-h), independent of the Anvil queue.
- **Boundary note** RoboJudge vs uncertainty audit — part of RoboJudge
  week-3 writing.

## Related

[[Unified-Direction-Ranking-2026-08]] · [[Prereg-RoboJudge-Audit]] ·
[[Prereg-1NFE-Diversity]] · [[Prereg-Epistemic-Contextualization]] ·
[[Anvil-Interactive-GPU-Workflow]]

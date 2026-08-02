# Research Wiki — Home

General research knowledge shared across projects. Project-specific detail
stays in each project's own repo/wiki (`svib` for the NeurIPS paper and its
evidence base, `svib2` for the distillation line). Pages here were seeded
from those wikis on 2026-08-02; this repo is now the canonical home for the
general pages, and the copies left behind in `svib`/`svib2` are frozen.

The `wiki/` directory syncs to the GitHub Wiki automatically on push
(`.github/workflows/wiki-sync.yml`). Results and analysis code live in the
repo proper (see `results/`), not in the wiki.

## Current strategy (read these first)

- **[[Unified-Direction-Ranking-2026-08]] — CURRENT RANKING, 2026-08-03.**
  Full 8-lane gating sweep of every scan candidate: three flagships
  (autoresearch FDR ★★★★★, judge-audit program ★★★★½, parallel-RL
  factorial ★★★★½), a deep ★★★★ bench, T1 demoted to ★★★ (three cells
  published at top venues), ten candidates killed with evidence, and the
  week-one zero-cost gates.
- **Pre-registrations (DRAFT, awaiting sign-off):**
  ICLR 2027 — [[Prereg-RoboJudge-Audit]] (diagnostic) ·
  [[Prereg-ParallelRL-Factorial]] (method; replaces the declined
  [[Prereg-Autoresearch-Accept-Rule]]).
  Next cycle — [[Prereg-1NFE-Diversity]] (CVPR 2027) ·
  [[Prereg-SigLIP2-Ingredient-Ladder]] (ICML 2027, engineering starts now).
  Each: problem statement, research state, novelty, locked
  hypotheses/arms/metrics, kill criteria, timeline.
- [[Top-Researcher-Scan-2026-08]] — 26 leading researchers profiled:
  convergence map (verification bottleneck, diversity collapse, unarbitrated
  mechanism claims), ranked opportunities, method directions, operating
  lessons.
- [[Direction-Reevaluation-2026-08]] — all eight candidate directions
  re-gated by remaining opportunity (not crowding); current star ranking.
- [[Status-And-Survivors]] — plain-language orientation: every surviving
  direction with cost, pros/cons, and star priorities (star table superseded
  by the re-evaluation above; SVIB part now lives in the svib repo).

## Direction surveys and gates (history of what we checked)

- [[Method-Opportunities]] — method directions with baselines and numbers to
  beat (T1–T4 and clusters).
- [[Live-Research-Opportunities]] — evaluation-side directions.
- [[Self-Improving-AI-Survey]] — 772-paper sweep; verdict superseded by the
  re-evaluation, taxonomy still valid.
- [[Field-Scouting-Survey]] · [[Math-Grounded-Direction-Survey]] ·
  [[Next-Direction-Literature-Survey]] · [[Calibration-Opportunity-Survey]] ·
  [[Compression-Audit-Direction]]
- Gates and pre-registrations (closed): [[Direction-Gate-Results]] ·
  [[LLM-KD-Direction-Gates]] · [[KD-Noise-Floor-Stage1]] ·
  [[KD-Evidence-Audit-Gate]] · [[Calibration-Draw-Preregistration]] ·
  [[Calibration-Prior-Art-Gate]] · [[Temperature-Confound-Preregistration]]

## Cluster setup and best practice (from svib2)

- [[GPU-Resources-Across-Clusters]] — what GPUs we have where (OrangeGrid
  L40S/A100 no-credit no-time-limit; Anvil A100 + H100 "Anvil AI"; Delta),
  and what to run on which.
- [[Anvil-Interactive-GPU-Workflow]] — the default workflow for interactive
  GPU work on Anvil.
- [[Delta-Setup-and-Parallel-Workflow]] — NCSA Delta setup and running
  Anvil+Delta in parallel.
- [[Anvil-vs-Delta]] — cluster comparison across Anvil, Delta, DeltaAI, and
  Bridges-2 (hardware, partitions, quotas, queueing, accounting).
- [[CUDA-Compatibility-and-vLLM]] — CUDA/driver/vLLM compatibility notes.
- [[Data-Transfer-Between-Clusters]] — how to move large files between Anvil,
  Delta, and OrangeGrid (Globus / rsync / staging recipes; croc as fallback).
- [[Data-and-Caches]] — where datasets, feature caches, and artifacts live
  under `/anvil/projects/x-cis261253/`, and provenance rules.
- [[Research-Automation-Tools]] — installed automation/skills tooling.
- [[Anvil-H100-Qwen36-vLLM-Benchmark]] — H100 vLLM throughput reference.

Additional cluster facts recorded in [[Top-Researcher-Scan-2026-08]]
(operational notes): Isaac Sim needs RT cores — L40S only, not A100/H100/H200;
MuJoCo/SAPIEN stacks run everywhere.

## Standing process rules

1. **Prior-art gate before any experiment** — read method sections of the
   nearest papers and quote the sentence establishing the gap; never trust a
   paper's own novelty claim; re-run any gate older than ~6 weeks; **every
   search must explicitly cover the most recent 8 weeks** (a July gate missed
   two June papers that had already killed its direction).
2. **Re-gates must re-identify the competitor, not just re-check the claim** —
   T1's cells went to three top venues while we watched the wrong group.
3. **Lane-wide sweeps beat person-scoped checks** — four researcher-scan
   premises died on lane search that person-scoped checks had passed.
4. **Filter directions by remaining opportunity (saturation), never by
   crowding** — and an empty lane must explain why it is empty.
5. **Every diagnostic project names, at pre-registration time, the method
   paper it unlocks** — diagnosis first, then own the fix.
6. **Pair every arbitration with a released reusable artifact** (harness,
   benchmark, metric) — diagnostics buy credibility, artifacts buy citations.
7. **Substrate liveness gates outcomes, not just cost** — a null on your own
   reimplementation of unreleased code is not defensible; verify cited
   benchmarks/datasets actually exist and hydrate before rating a direction.
8. **Verify quoted numbers against primary artifacts** before they enter a
   design brief.
9. Ask "who changes their behavior if this result is true?" before committing
   compute.

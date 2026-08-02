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

- [[Anvil-Interactive-GPU-Workflow]] — the default workflow for interactive
  GPU work on Anvil.
- [[Delta-Setup-and-Parallel-Workflow]] — NCSA Delta setup and running
  Anvil+Delta in parallel.
- [[Anvil-vs-Delta-for-SVIB2]] — cluster comparison (partitions, quotas,
  queueing) — written for svib2 but generally applicable.
- [[CUDA-Compatibility-and-vLLM]] — CUDA/driver/vLLM compatibility notes.
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
   paper's own novelty claim; re-run any gate older than ~6 weeks.
2. **Filter directions by remaining opportunity (saturation), never by
   crowding** — and an empty lane must explain why it is empty.
3. **Every diagnostic project names, at pre-registration time, the method
   paper it unlocks** — diagnosis first, then own the fix.
4. **Pair every arbitration with a released reusable artifact** (harness,
   benchmark, metric) — diagnostics buy credibility, artifacts buy citations.
5. **Check substrate liveness** (maintainers, recent pushes, released
   checkpoints) before adopting any external codebase as a target.
6. Ask "who changes their behavior if this result is true?" before committing
   compute.

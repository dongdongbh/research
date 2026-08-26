# H3 on Delta — Session Handoff (2026-08-15)

This page starts a NEW Claude session on the Delta cluster. The owner
pulls two repos there and points the session here. Read this page top
to bottom, then the two authority documents, before touching a GPU.

## What this campaign is

H3 is the last locked experiment of the 1-NFE diversity study
([[Prereg-1NFE-Diversity]] — the pre-registration; its ledger governs).
One model (IMM) is trained three ways on CIFAR-10, changing one loss
setting, to test whether the "averaging" training goal costs variety
(recall) in a fully controlled setting. The measurement — recall at
matched precision for one-step generators — is verified empty
territory: none of the source papers measures recall at all.

## The two authority documents

1. `Prereg-1NFE-Diversity` in this wiki — the lock, the H3 design
   (three arms), the cost-pilot results, and the ledger. Any deviation
   is escalated to the coordinator/owner and logged HERE, never
   self-approved.
2. In the nfe1 repo: `DELTA-H3-HANDOFF.md` (the technical runbook) and
   `runs/week2_check/20260809/H3-LAUNCH-SPEC.md` (the design spec) and
   `runs/h3_pilot/20260814/` (measured costs, environment traps,
   NOTES.md).

## Delta facts (verified 2026-08-08, memory: delta-cluster-facts)

- Account **`bhvn-delta-gpu`** — 1,999 of 2,000 hours unused.
- Target partition **`gpuH200x8`**: 8 nodes of 8×H200-141GB. All 8
  nodes were busy at last check — **submit early and let it pend**.
  `gpuA100x4` (40 GB cards) works for probes; the pilot's peak was
  34.8 GB, which fits but is tight.
- **GPU walls are 2 days.** No longer partitions exist. The campaign
  must checkpoint and resume across windows (IMM snapshots every tick,
  so this is native).
- **torch must be pinned to cu128 wheels** (driver 570.x):
  `--index https://download.pytorch.org/whl/cu128`, and IMM needs
  **torch 2.8.0+cu128 exactly** (newer torch breaks IMM's Sampler).
- Delta bills the MAX fraction of GPU/CPU/RAM requested — a full-node
  request bills as a full node. That is fine here: we want the node.
- RH96 migration: migrated nodes need `--reservation=RH96` from
  dt-login04; the H200 nodes had NOT migrated as of Aug 8. Re-check.
- Only the owner can log in (password + Duo). The session runs there
  under the owner's account; agents never store those credentials.

## The plan, in order

1. **Check the stability verdict first — ANSWERED: STABLE
   (2026-08-15, nfe1 commit 1444dfe).** All three arms finished all
   20 ticks with zero non-finite weights and matching parameter
   norms; the verdict table and `stability.json` are in nfe1
   `runs/h3_pilot/20260814/` (RESULTS.md §7). The step-1 gate is
   cleared — pull nfe1, confirm you see commit 1444dfe, and proceed
   to the probes.
2. **Probe before sizing (hard rule).** Run the 3-tick timing probe on
   ONE H200 (and on an A100 if the H200 queue is long) exactly as the
   runbook says. Before submitting the probe to the busy H200 queue,
   smoke-test the same command on one cheap A100 first (rule of
   2026-08-14 in [[Delta-Setup-and-Parallel-Workflow]]) so a code bug
   cannot waste a long queue wait. The pilot measured 1,780 sec/tick on an L40S; the
   H200 number is unknown and this model is bandwidth-bound, so DO NOT
   assume a ratio. Also probe the multi-GPU scaling (1 vs 2 vs 4 GPUs
   on one arm, 3 ticks) — DDP efficiency at 32×32 is unknown.
3. **Size the campaign from the probe** and write the numbers into the
   prereg ledger (research repo) before the full submission. Baseline
   plan pending measurements: 3 arms × 500 ticks, fp32, IMM's shipped
   CIFAR recipe untouched, all arms at the SAME GPU count (summation
   order differs by card count, so arms must match). 500 ticks/arm is
   the owner-recommended size — confirm it is still the owner's choice
   at launch.
4. **Submit** per the runbook (the owner runs the actual sbatch/salloc;
   prepare everything so it is one command). Attach artifact-based
   watchers; report at tick milestones.
5. **Scoring comes after training** and reuses the nfe1 harness
   (matched-precision recall protocol, paired bootstrap; CIFAR needs
   its OWN frozen reference statistics and its own re-derived
   resolution limit — never inherit ImageNet's 0.020 figure).

## Standing rules (non-negotiable, from this program's history)

- Every number in any report traces to a file; nothing is invented.
- Deviations are escalated and ledgered, never self-approved. Check
  the git ledger — not your context window — before any retraction.
- Own isolated Python env per worker; never share a venv between
  concurrent workers.
- No git commits in evidence repos without the owner's word; the
  fresh campaign run dirs are append-only with manifests.
- Watchers watch artifacts and terminal states, not process liveness.
- NOTE-P1 covariate travels with the results: at equal ticks the
  tuple-matched arm takes 4× the optimizer steps of the others.

## Related

[[Prereg-1NFE-Diversity]] · [[GPU-Resources-Across-Clusters]] ·
[[Execution-Plan-Locked-Projects-2026-08]] (cross-cluster onboarding
section)

# Pre-registration: Is One-Step Generative Diversity Collapse Intrinsic, or an Averaging Artifact?

## Words used on this page

Read this list once. Every word below is used the same way everywhere on this
page.

- **1-NFE** — one network function evaluation. The model makes a whole image
  with one call to the network instead of dozens. NFE means "number of function
  evaluations," so 128-step means NFE = 128.
- **Checkpoint** — one saved copy of a trained model's weights.
- **Recall** — how much of the real data's variety the model covers. Higher is
  more diverse.
- **Precision** — how realistic the generated images look. We match precision
  between models so that a recall comparison is fair.
- **FID** (Fréchet Inception Distance) — the field's main image-quality score.
  Lower is better.
- **Guidance** (also written **cfg**) — a knob at sampling time that trades
  variety for image quality.
- **Shortfall** — how much recall a model loses when it drops from many steps
  to one step.
- **Cell** — one measurement condition in the sweep: one model, measured at one
  setting.
- **Arm** — one version of the experiment, run to answer one question.
- **Bootstrap** — a way to show how stable a result is. It re-samples the same
  data many times and re-computes the answer each time. Each re-computation is
  one **replicate**.
- **Confidence interval (CI)** — the range of values the data still allow. If
  the interval does not contain zero, chance alone is an unlikely explanation.
- **Holm correction** — a rule that guards against false positives when several
  hypotheses are tested at once.
- **Noise floor** — how much a number moves when nothing meaningful changes. A
  difference smaller than the noise floor means nothing.
- **Deviation** — any change made after the plan was locked. Every deviation is
  written down on this page.

## Sweep result (2026-08-10)

The full record is in nfe1
`runs/week2_check/20260809/RESULTS-week2-boundary.md`, plus 12 numbered notes.

**Where H1–H4 are defined:** in section 4 of this page, under
"Predictions, fixed before the run." Short version — **H1:** variety
(recall) falls as steps drop to one inside the averaging family.
**H2:** Drifting, trained without averaging, loses less than half of
what the averaging family loses. **H3:** a matched pair trained from
scratch shows the same ordering. **H4:** SubFlow's fix helps the
averaging family but not Drifting.

**H1 is confirmed in Shortcut and NOT confirmed in iMF.** In Shortcut every
matched contrast is positive: 6 of 6 for the B model and 7 of 7 for the XL
model, at +0.0917 at 50k samples. In iMF the same comparison gives +0.0004 on a
comparable grid. Both models train with an averaging objective. So the effect
depends on the individual model, and it is therefore not a property of the
averaging objective.

**H2 is true as an inequality, but the mechanism behind it is not supported.**

- The inequality: Drifting's 1-step recall EXCEEDS its matched 128-step
  reference by +0.2153 on the clean row of record. A twin row recovered after a
  crash agrees to within 0.0044, which validates feature salvage as a
  technique. The CI, recomputed on the primary row, is about [+0.205, +0.226].
- Why the mechanism fails: the comparison class is incoherent. Shortfalls
  within the averaging family span 0.0004–0.0969. And one averaging model and
  one non-averaging model are statistically indistinguishable at matched
  precision: +0.0053, CI [−0.0046, +0.0156]. That is an informative null,
  6.6× tighter than Drifting's advantage.

**Central finding: step-count shortfalls do not group by objective family. The
individual training recipe dominates.**

**What survives:** collapse is NOT intrinsic to one step. Drifting beats every
other 1-NFE model by +0.10–0.11 at matched precision. But the objective family
does not predict which models escape collapse.

**Two structural results:**

- Drifting's precision has a minimum at cfg 1.0, with a floor of 0.785. It
  shares no precision range with Shortcut-B at 1-NFE. That is the risk §7 named,
  now measured.
- Recall falls as the number of generated samples rises, by −0.04 to −0.10. The
  size of the fall depends on the model. The same-count rule is enforced.

**Cross-protocol comparability is established.** We reproduced the published
AFM, iMF, and Drifting numbers to within 0.03.

**Statistics used:** paired bootstrap, 2,000 replicates, Holm correction across
the hypotheses. Both adjusted p-values are below 0.001.

**Four errors that the worker found and corrected are logged in the ledger
amendments on this page.**

**Closing measurement, now COMPLETE.** The properly matched 50k 128-vs-1 pair
is **+0.1705, CI95 [+0.1609, +0.1802]** (paired bootstrap, 2,000 replicates).
The harness point estimate of +0.1699 agrees with it, within the documented
reimplementation difference. This SUPERSEDES the earlier +0.1365 interval. That
earlier interval belonged to a pair that sat 0.0006 outside the pre-registered
tolerance, so it is discarded, not averaged. H1 at 50k therefore reads: 128-vs-1
is +0.1705 [+0.1609, +0.1802], and 4-vs-1 is +0.0917. Both are resolved, and
they are ordered as predicted.

**Process note: two guards worked.** A stale cell reference nearly recomputed
the WRONG pair — the one outside tolerance — as if that were the correction.
The worker checked feature-file identity against the recorded rows before
running. Separately, the coordinator's garbled relay was refused by the sibling
worker.

**All GPU work is complete:** 39 cells. Both cards were handed to the
contextualization campaign under the drain rule.

**H3 (IMM M=1 vs M=4): the launch spec is delivered**, in nfe1
`runs/week2_check/20260809/H3-LAUNCH-SPEC.md`. It carries one design-critical
pre-declaration: the lock's "single knob" is CONFOUNDED. At a fixed batch size,
M also sets how many distinct time tuples each step uses, namely batch/M. So a
naive two-arm run would differ in the objective AND in time-sampling density at
the same time. The spec therefore prescribes:

- THREE arms, not two: M=4 at batch B; M=1 at batch B; and M=1 at batch B/4,
  which holds the tuple count equal.
- CIFAR scale, which sits inside the locked plan's reduced-scale clause.
- Fixed seeds and equal-tick stopping.
- Both guidance branches enumerated.
- CIFAR's own reference statistics and a re-derived resolution limit.
- An explicit "uninterpretable" outcome, declared in advance.
- A 20-tick pilot that MEASURES the cost before any allocation request, so no
  throughput is guessed.

Owner decision still needed: the compute window.

**Cost pilot RESULT (2026-08-14, measured on OrangeGrid L40S).** All
three arms cost the same: about 1,780–1,800 seconds per tick, which is
2.0 ticks per hour per card, within 1.4% of each other, with identical
peak memory. One rate prices the whole campaign. The shipped
10,000-tick length cannot be bought (~14,900 GPU-hours). The exchange
rate is simple: **100 ticks per arm = 149 GPU-hours across the three
arms.** The recommendation is 500 ticks per arm = **745 GPU-hours,
about 10.4 days on three free OrangeGrid cards** — 500 ticks trains a
real model, not a toy. Keep fp32: bf16 was measured (1.23× faster,
3.7 GB lighter) and declined, because the saving does not change what
is affordable and it deviates from the published recipe. The M=1
stability check finishes unattended overnight. One new caveat, named
NOTE-P1: at equal ticks, the tuple-matched arm takes 4× the optimizer
steps of the others; the main A-vs-B contrast is unaffected.

**Companion gate (2026-08-14): the "transplant the diversity
ingredient" METHOD idea is KILLED** (overlap Level 2).
[SubFlow](https://arxiv.org/abs/2604.12273) already ships a
plug-and-play diversity fix into Shortcut and MeanFlow and measures it
as recall; PacGAN owns the group-size knob. H3 itself is untouched:
Drifting, iMF and IMM never measure recall or precision anywhere in
their papers, so the matched-precision recall measurement is still an
empty slot that H3 takes. One ungated lead is recorded and NOT acted
on: whether train-time guidance-strength conditioning correlates with
diversity (must pass a gate before any GPU time). Records: nfe1
`runs/h3_pilot/20260814/`, tier2gates
`runs/h3ingredient-gate-20260814/`.

## Delta campaign preparation (2026-08-26): step-1 gate cleared, probes staged, numbers PENDING

**Who:** prepared by the Delta session on dt-login02; the owner submits every
job. Every number below traces to a named file or command.

**Step 1, the M=1 stability check, is cleared.** nfe1 commit `1444dfe`,
`runs/h3_pilot/20260814/RESULTS.md` §7 and `stability.json`: at tick 20
each of the three arms has 0 non-finite values among its 56,062,595 EMA
parameters (EMA = the slow-averaged copy of the weights). EMA L2 norms are
226.6004 (arm A), 226.6002 (arm B), 226.6850 (arm C). The runbook's
"if M=1 diverged, STOP" rule does not fire.

**Cluster facts re-checked 2026-08-26 09:15 CDT (`scontrol`, `sinfo`,
`squeue`, `accounts` on dt-login02):**

- The RH96 reservation no longer exists on Delta. `scontrol show reservation`
  lists only infinia, gpuc-fw-float, NDIF-H200 and NDIF-A40. No job needs a
  `--reservation` flag.
- H200 node gpue01 stays reserved for other projects (NDIF-H200) until
  2026-09-19, so 7 of 8 H200 nodes are open. H200 queue: 403 jobs pending,
  32 running. A100 queue: 1,658 pending, 235 running. Submit early.
- Balance: 1,999 of 2,000 service units (SU). One SU is one A100-GPU-hour.
  The H200 partition charges 3 SU per GPU-hour ([[Anvil-vs-Delta]], from the
  Delta queue table). So the balance buys at most **666 H200-GPU-hours.**
- Why that matters: at the L40S pilot rate (1,775.7 s/tick, arm A median,
  `cost_table.json`) 500 ticks × 3 arms costs 740 GPU-hours = **2,220 SU,
  which is more than the balance** (`parse_probe.py` on the pilot log). The
  campaign fits only if an H200 is faster than an L40S. The probes decide
  the affordable tick count; nothing is assumed.
- One H200 node has 96 CPUs, 2,015 GB memory and 8 GPUs, so 12 CPUs and
  250 GB per GPU. Every job requests exactly that share, so Delta's
  largest-fraction billing charges only the GPUs used.

**Three code facts found while preparing. Each changes the job scripts,
none changes the training recipe. Escalated for the owner's ratification,
not self-approved.**

1. IMM never writes its `training-state-latest.pt` file. The pilot log says
   `Failed to save the latest checkpoint: Missing key use_zero` at every
   save. The handoff's "snapshot every tick" was wrong: the shipped interval
   for numbered saves is 500 ticks (`configs/cifar10.yaml`). To resume
   across 48-hour walls the scripts set `training.snapshot_ticks=10` and
   `training.state_dump_ticks=10`. These settings only decide what is
   written to disk (about 0.9 GB per save and 45 GB per arm, estimated from
   the 56,062,595 parameters in fp32: weights, two RAdam moments, and the
   EMA copy). They do
   not touch any number in training.
2. IMM's stop rule: a run with `total_ticks=T` trains T−1 full ticks plus
   one optimizer step (`training_loop.py` line 504; this is why the pilot's
   tick 20 was partial). The scripts now pass T+1, so "500 ticks" means
   exactly 500 full ticks: 50,000 optimizer steps for arms A and B, and
   200,000 for arm C (NOTE-P1). The throw-away partial tick costs one step.
3. IMM's resume restores the weights, the EMA and the optimizer state
   (`train.py` lines 65–82, `training_loop.py` lines 288–311). It does NOT
   restore the position in the data order or the random-number streams;
   those restart from the seed. To keep the three arms identical in
   procedure, every 48-hour window stops all arms at the same tick
   (`WINDOW_TICKS`), so all arms resume at the same ticks. The list of
   resume ticks will be written here after the campaign.

**Probe and smoke jobs, all in nfe1 `delta/` (the owner submits; every job
asks for the 48-hour maximum wall per the owner's standing instruction):**

| Job | Card(s) | What it measures | Rough cost |
|---|---|---|---|
| `smoke_a100.sbatch` | 2 × A100 (factor 1) | The exact campaign code path: DDP, numbered saves, and a real resume across two launches (ticks 1–2, then ticks 3–4 from the tick-2 save). Pass = second launch prints `resuming from … (tick 2)` and its tick-3 line reads kimg 1228.8. | 2–4 SU (4 full ticks + 2 partial on 2 cards; range = perfect to no DDP scaling at the L40S rate) |
| `probeA_h200.sbatch` | 1 × H200 | Probe A: seconds per tick on one H200, arm-A settings, 3 full ticks (median of ticks 2–3). | 4.4 SU at the L40S rate (3 × 1,775.7 s × 3) |
| `probeB_h200.sbatch` | 4 × H200 (half node) | Probe B: seconds per tick at 2 and 4 GPUs, same settings. The 1-GPU row comes from Probe A. | 13–36 SU at the L40S rate (range = perfect to no DDP scaling) |

Outputs land in `/work/hdd/bhvn/dli26/h3/runs/<job>_<jobid>/`.
`parse_probe.py` reads the logs with the pilot parser unchanged (tick 1 and
partial ticks excluded) and prints the sizing table below.

**Sizing rule, fixed before the numbers exist:**

- Cost in SU for 500 ticks = 3 arms × n GPUs per arm × 500 × (seconds per
  tick at n GPUs) ÷ 3600 × 3.
- Ticks per window = floor(46 hours ÷ seconds per tick ÷ 10) × 10 (46 = the
  48-hour wall minus 2 hours for startup and the last save).
- Pick the smallest n whose cost fits 1,999 SU minus what the probes spent,
  with the fewest windows. All three arms use that same n. If 500 ticks does
  not fit at any n, the table shows the affordable tick count and the owner
  chooses the tick count.

**Smoke result 1 (job 21472348, gpua068, 2 × A100, 2026-08-26 09:29–09:50
CDT, COMPLETED 0:0):** startup, data, DDP over 2 cards, the fp32 banner, the
numbered saves and the sample export all worked on the first try. Steady
rate on 2 × A100: **611.9 s/tick** (tick 2; tick 1 was 622.7 s with setup);
peak GPU memory 34.80 GB, the same as the pilot. But the smoke caught a
bug in its own settings: with saves every 1 tick, the partial final tick's
save (tick 3) is a multiple of the interval, so the second launch resumed
from it and correctly said "nothing to do" — the resume path was NOT
exercised. Fix: `run_arm.sh` now refuses an interval below 2 and requires
the tick targets to be multiples of the interval; the smoke re-runs with
interval 2 and 4 full ticks (job 21472708). The campaign setting (interval
10, targets multiples of 10) never had this problem.

**Smoke result 2 (job 21472708, gpua071, 2 × A100, 09:52–10:13 CDT):** the
first launch ran ticks 1–2 (602.5 s/tick at tick 2) and saved at tick 2. The
second launch chose the right file (`training-state-000002.pt`, skipping the
partial tick-3 save) and then crashed while loading it:
`Unsupported global: torch_utils.persistence._reconstruct_persistent_obj`.
Cause: torch 2.6 and later refuse, by default, to load saved files that hold
whole Python objects (`weights_only=True`), and IMM's training-state file
holds the whole network object. IMM predates that change. Fix, without
editing IMM: `run_arm.sh` sets `TORCH_FORCE_NO_WEIGHTS_ONLY_LOAD=1`, the
official switch back to the old behaviour; the files are our own. Checked
on the login node: the tick-2 file loads (network, optimizer state, scaler
state). File sizes: 673 MB state + 225 MB snapshot = 0.9 GB per save, as
estimated. Job 21473085 re-runs only the resume launch against that
tick-2 save.

**Probe A RESULT (job 21472531, gpue03, one H200, 09:45–10:13 CDT;
`runs/probeA_h200_21472531/probe_A.json`, parsed with the pilot parser):**

- Seconds per tick, steady (ticks 2 and 3): 556.2 and 556.1 → median
  **556.15 s/tick**. Tick 1 with setup: 567.1 s. Peak GPU memory 34.82 GB,
  steady 33.50 GB (the pilot: 34.8 / 33.21 GB).
- One H200 is **3.19× an L40S** on this job (1,775.7 ÷ 556.15).
- At 1 GPU per arm: 500 ticks × 3 arms = **695 SU**, which fits the 1,999
  SU balance; each arm needs 77.2 hours = **2 windows of 48 hours** with
  290 ticks per window; 1,437 ticks per arm would be affordable at the
  balance. The 2- and 4-GPU rows (Probe B) decide whether more GPUs per
  arm buy fewer windows at an acceptable SU price.

**Incident during Probe A (recorded, no effect on the numbers):** the
session edited the shared `run_arm.sh` while this job was running it.
Bash reads a script piece by piece, so when training finished, bash
continued from a shifted position in the new file and started a SECOND
training run on the same GPU (`ngpu1/imm_cifar10/00001-…`). The job was
cancelled at 10:22 CDT before that run reached a tick, so the log holds
exactly the four tick lines of the real probe, and the parser (which
drops any tick whose image counter does not advance) saw only those.
Cost: about 9 minutes of one H200. Fix: every sbatch now copies
`run_arm.sh` into the job's output directory at start and runs that
frozen copy. Rule from now on: never edit a script while a job that
reads it is running.

**Smoke result 3 — RESUME VERIFIED (job 21473085, gpua048, 2 × A100,
10:20–10:41 CDT, COMPLETED 0:0; log `runs/smoke_a100_21472708/
ngpu2_job21473085_launch1.log`):** the launch chose `training-state-000002.pt`,
loaded it under the env-var fix, and its first tick line was `tick 3 kimg
1228.8` — the exact image count for three full ticks — then `tick 4 kimg
1638.4`, saved at tick 4, and ended on the partial tick 5. A further launch
found the tick-4 save and exited with "already at tick 4; nothing to do".
That is the campaign's per-window behaviour, proven on Delta. Steady rate
on 2 × A100 across the three smoke runs: 602.5–613.1 s/tick.

**Owner decisions (2026-08-26, about 13:25 CDT), verbatim:** "Both
decisions confirmed. (1) Resubmit Probe B with --time=04:00:00 — the probe
needs <3 h and the max-wall rule is for work jobs, not probes; log this as
an owner-approved exception. (2) I confirm the campaign: 500 ticks per arm,
1 GPU per arm. Set GPUS_PER_ARM=1, WINDOW_TICKS=290, flip the gate, and
submit today so the queue wait starts now. Probe B's result goes into the
ledger for the record but does not change this campaign — all arms keep
1 GPU for the whole run."

**Owner-approved exception:** Probe B runs with a 4-hour wall instead of
the 48-hour maximum. Reason: it is a probe, not a work job, and a shorter
wall lets the scheduler fit it into gaps (job 21472532, 48 h wall, had an
estimated start 27 hours out after 3.5 hours of waiting; cancelled and
resubmitted as **job 21477439** at 13:27 CDT).

**CAMPAIGN SUBMITTED: job 21477440, 2026-08-26 13:27 CDT.** Shape: 3 arms
side by side on one gpuH200x8 node, 1 H200 each (3 GPUs, 36 CPUs, 750 GB =
3/8 node), 48-hour wall, `TICKS=500`, `GPUS_PER_ARM=1`, `WINDOW_TICKS=290`,
`DUMP_K=10`, `GATE_CONFIRMED=yes`. Plan: window 1 runs every arm from
tick 0 to tick 290 (290 × 556.15 s = 44.8 h + startup), window 2 resumes
every arm at tick 290 and runs to tick 500. The same file is resubmitted
for window 2. Expected cost: 695 SU (`probe_A.json`). Run directory:
`/work/hdd/bhvn/dli26/h3/runs/campaign/<arm>/`; per-window logs
`<arm>_job<jobid>.log` in that directory. Arms: `m4_seed0` (A),
`m1_seed0` (B), `m1_tuplematched_seed0` (C), seed 0, batch_gpu 256, fp32,
`training.metrics=[]`, snapshot and state saves every 10 ticks, all other
settings the shipped `cifar10.yaml`.

**Campaign window 1 COMPLETE.** Job 21477440 ran on gpue03 from
2026-08-26 20:42 to 2026-08-28 19:58 CDT (1 day 23 h 17 min, exit code
0:0), 43 minutes inside the 48-hour wall. **All three arms reached full
tick 290** — the plan's target — so no arm resumes from a lower tick:

| Arm | Last full tick | Rate (s/tick) | Save failures | Loss NaN/Inf |
|---|---|---|---|---|
| A `m4_seed0` (M=4, batch 4096) | 290 | 555.9 | 0 | 0 |
| B `m1_seed0` (M=1, batch 4096) | 290 | 585.4 | 0 | 0 |
| C `m1_tuplematched_seed0` (M=1, batch 1024) | 290 | 583.1 | 0 | 0 |

Each arm holds 29 numbered saves, one at every multiple of 10 from 10 to
290, plus the partial tick 291 dump that the resume rule ignores (291 is
not a multiple of 10). Rates stayed flat to within 0.5% for two days. The
M=1 arms ran 5.3% slower than the M=4 arm throughout; in the L40S pilot,
where each arm had its own machine, the spread was 1.3%. This is a
throughput difference only — it does not touch the objective, the data,
or the number of optimizer steps.

**Campaign window 2 STARTED: 2026-08-30 07:19 CDT on gpue08** (job
21570100). It was submitted on 08-28 17:36 with
`--dependency=afterany:21477440`, so it collected queue priority while
window 1 finished but could never run at the same time; it then waited
1 day 11 hours in the ordinary queue. All three arms resumed exactly as
designed: each loaded `training-state-000290.pt`, skipped the partial
tick-291 dump (291 is not a multiple of 10), and set `resume_tick=290
target_tick=500 total_ticks=501`. No load errors. Arms are on GPUs 0, 1
and 2 as before, writing into the same run directory each arm used in
window 1. Remaining work: 210 full ticks, about 32 hours for arm A and
34 hours for arms B and C, inside a 48-hour wall that ends 2026-09-01
07:18 CDT.

**CAMPAIGN TRAINING COMPLETE: 2026-08-31 17:38 CDT.** Window 2 (job
21570100) ran 1 day 10 h 19 min and exited 0:0, about 14 hours inside its
wall. **All three arms reached full tick 500**, each with the identical
image count `kimg 204800.0` (500 × 409.6), each ending on the throw-away
partial tick 501. Every arm holds 52 numbered saves.

| Arm | Full ticks (window 1 + 2) | s/tick w1 | s/tick w2 | Final kimg | Exit | Save failures | Loss NaN/Inf |
|---|---|---|---|---|---|---|---|
| A `m4_seed0` | 290 + 210 = 500 | 555.9 | 566.1 | 204800.0 | rc=0 | 0 | 0 |
| B `m1_seed0` | 290 + 210 = 500 | 585.4 | 587.2 | 204800.0 | rc=0 | 0 | 0 |
| C `m1_tuplematched_seed0` | 290 + 210 = 500 | 583.1 | 585.5 | 204800.0 | rc=0 | 0 | 0 |

**Stability of the FINAL weights — all arms finite** (`campaign/FINAL/
stability_tick500.json`, produced by `delta/check_final.py`, which imports
the pilot's `summarize()` unmodified, so this is the same measurement as
the tick-20 pilot verdict). IMM's snapshot files carry only the EMA
("slow-averaged") copy of the network — keys `ema`, `augment_pipe`,
`dataset_kwargs` — so, as in the pilot, the EMA copy is what is measured:

| Arm | Non-finite params | EMA L2 norm | Max abs param | Ratio vs arm A |
|---|---|---|---|---|
| A `m4_seed0` | 0 of 56,062,595 | 260.6406 | 1.3964 | 1.0000 |
| B `m1_seed0` | 0 of 56,062,595 | 261.1315 | 1.5596 | 1.0019 |
| C `m1_tuplematched_seed0` | 0 of 56,062,595 | 383.0679 | 1.9433 | 1.4697 |

Zero non-finite values anywhere after 500 full ticks. Launch-spec risk 1
("M=1 may not train stably") is answered NO at full campaign length, not
just at the pilot's 20 ticks. Arms A and B — the pair that answers H3 —
end within 0.2% of each other in weight norm.

**Arm C's larger norm is the NOTE-P1 covariate, not a finding and not
instability.** Arm C runs 4× the optimizer steps per tick, so at tick 500
it has taken 200,000 steps against 50,000 for arms A and B. Its norm ratio
grew steadily with those steps: 1.331 at tick 300, 1.4697 at tick 500.
Nothing is non-finite and the norm is the same order of magnitude. This
number says nothing about diversity; it is reported so no later reader
mistakes it for a result.

**Cost, measured:** the campaign spent about 739 SU against the 695
projection, 6% over. The projection used Probe A's single-arm rate
(556.15 s/tick, one arm alone on a node); in the real campaign the two
M=1 arms ran about 5% slower than arm A with three jobs sharing a node.
Balance after training: 1,256 of 2,000 SU. Wall-clock total 81.6 hours
across two windows. Disk 132 GB.

**NOTHING ABOUT H3 IS ANSWERED YET.** These runs produced three trained
models; they did not measure anything about diversity. No samples have
been generated and no recall, precision or FID has been computed from
these checkpoints. The measurement phase is untouched, and it starts with
launch spec §9 step 1 — build and freeze CIFAR-10 reference statistics
and record the digest, which has never been done — then generation with
`run_imm.py` at 1/2/4/8 steps, scoring with `score_npz.py` at k=3,
matched precision at ±0.02 with BOTH guidance branches enumerated
(NOTE-12: IMM's precision is not monotone in guidance), paired bootstrap
intervals (never the naive generated-side bootstrap, biased −0.09), and
CIFAR's OWN re-derived resolution limit. Both pre-declared
"uninterpretable" outcomes remain live: arms A and B may share no
precision range, and arms B and C may disagree.

**Final weights for scoring:** `campaign/<arm>/imm_cifar10/00001-cifar10-
32x32-uncond-gpus1-cifar10_a1b4k15/network-snapshot-000500.pkl` (window-1
saves are under `00000-…`; window 2 opened a new numbered directory).

**Note on IMM's resume, restated for the record:** resuming restores the
network, the EMA copy and the optimizer state, but not the position in
the shuffled data order and not the random-number streams, which restart
from seed 0. All three arms were interrupted at the SAME tick (290) and
resumed the same way, so the arms stay procedurally identical to each
other — which is what the comparison requires.

Tick-2 rates in the campaign (three arms sharing one node, one H200
each): **A 556.8 s, B 586.8 s, C 584.3 s per tick**, peak memory 34.82 GB
each. The two M=1 arms run 5.4% slower than the M=4 arm here (1.3% in the
L40S pilot, where each arm had its own machine). Consequence for window 1:
arm B is projected to reach tick 290 at about 2026-08-28 19:58 CDT, 44
minutes before the 48-hour wall at 20:42. If the wall arrives first, arms
B and C resume from their tick-280 save while arm A resumes from 290; that
would be recorded here as a per-arm difference in the resume tick (the
arms still stop at the same final tick, 500).

**Probe B RESULT (job 21477439, gpue05, 20:58–21:26 CDT, COMPLETED;
`runs/probeB_h200_21477439/probe_B.json`, joined with Probe A by
`parse_probe.py`, pilot parser rules unchanged):**

| GPUs per arm | sec/tick (median of ticks 2–3) | speed-up vs 1 GPU | DDP efficiency | SU for 500 ticks × 3 arms | fits 1,999 SU? | ticks per 46 h window | 48 h windows (3 arms side by side) |
|---|---|---|---|---|---|---|---|
| 1 | 556.15 | 1.0 | 1.0 | 695 | yes | 290 | 2 |
| 2 | 278.75 | 1.995 | 0.998 | 697 | yes | 590 | 1 |
| 4 | 140.1 | 3.97 | 0.993 | 700 | yes | 1180 | n/a (>8 GPUs) |

Plain reading: DDP (data-parallel training across GPUs) scales almost
perfectly for this model at 32×32 — 2 GPUs are 1.995× as fast, 4 GPUs
3.97×. Peak memory per GPU stays 34.82 GB at every count. So 2 GPUs per
arm would have finished 500 ticks in ONE 48-hour window for 697 SU
instead of two windows for 695 SU. **For the record only:** the owner
confirmed 1 GPU per arm before this result, the campaign (job 21477440)
was already running, and all arms keep the same GPU count for the whole
run. The 2-GPU shape is the recommendation for any future rerun. This paragraph is replaced by the
`parse_probe.py` table (`probe_A.json`, `probe_B.json`) when the probes
finish. Nothing in this section is measured on an H200 yet. `sbatch
--test-only` on 2026-08-26 09:28 CDT accepted all four files and estimated
starts of about now (smoke), +1 h (Probe A) and +23 h (Probe B, campaign).

## Scoring phase, opened 2026-08-31: two findings that change the measurement plan

**ESCALATED, NOT SELF-APPROVED.** Both items below change what the H3
measurement can do. They are recorded here for the owner's ruling, as the
standing deviation rule requires.

### Finding 1: guidance does not exist for these arms, so the guidance sweep cannot run

The H3 arms are trained **unconditional**. `configs/cifar10.yaml` sets
`dataset.use_labels: false`, every run directory is named `...-uncond-...`,
and the network therefore has `label_dim = 0`. IMM's guidance path
(`training/preconds.py`, `cfg_forward`, lines 268-280) sets `class_labels`
to `None` whenever `label_dim == 0`, then builds its guided batch with
`torch.cat([torch.zeros_like(class_labels), class_labels])`. There is no
guidance knob on an unconditional model, and asking for one raises an error
rather than silently doing nothing.

What this breaks: the locked plan's matched-precision recipe says "Match
precision through guidance and temperature sweeps" (section 4), the H3 launch
spec requires "Both guidance branches enumerated", and the campaign shape
(section 9 step 5) budgets "a guidance sweep at 10k per arm to find the shared
precision band". **None of that is executable on CIFAR-10 as trained.** The
week-2 note NOTE-12 about IMM's precision being non-monotone in guidance also
does not apply here.

What remains as a precision knob: the **number of sampling steps** (NFE) and
the step discretization. Nothing else — `pushforward_generator_fn` has no
temperature argument. This makes launch-spec risk 3 ("matched precision may
not exist between the arms") materially more likely, because the search now
runs over a handful of discrete NFE values instead of a continuous knob.

Consequence for generation: sample packs are produced at **NFE 1, 2, 4 and
8** per arm, which is what launch spec section 6 step 1 names, so the Anvil
side has something to match precision over. A 1-NFE-only delivery would make
the matched-precision step impossible.

### Finding 2: the study's instrument is not on Delta, and scoring stays on Anvil

`src/score_npz.py` and `src/bootstrap.py` score through Drifting's JAX
InceptionV3 with Drifting's reference files, and they carry hard-coded
`/anvil/projects/x-cis261253/...` paths. On Delta: `repos/drifting` is absent,
`/anvil` is not mounted, and no JAX environment exists. IMM ships its own
PyTorch InceptionV3, but substituting it would be a NEW scorer, and the lab's
provenance rule is that a new or reimplemented scorer's numbers do not count
until it reproduces a frozen invariant of the established pipeline — which
needs the ImageNet reference files that live on Anvil.

**Owner ruling, 2026-08-31, verbatim:** "Do not rebuild or approximate the
scorer on Delta. Scoring happens on Anvil with the original instrument."
Delta's job is therefore generation and staging only.

### Anvil-side amendments (2026-09-01, written before any decisive row was read)

Recorded by the coordinator at 20:10 UTC with 2 of 14 scoring rows finished
(the two reference rows only). Reported to the owner in the same session.

1. **Host change, same instrument.** Anvil's `ai` partition quoted a 9-day
   start for a 1-hour job, so the harness runs on OrangeGrid's L40S instead.
   The code is the same checkout (`uv.lock` byte-identical), the same
   Drifting JAX InceptionV3, the same chunked feature path, k = 3. The
   CIFAR reference port is a sibling script (`src/score_npz_cifar.py`); the
   vendored scorer and `score_npz.py` are untouched. Self-tests: a real
   2,000-image slice against itself gives precision 1.000, recall 1.000,
   FID −8e−07; the 50k real pack against the 10k reference gives FID exactly
   0.000 and Inception Score 11.27 (published real-CIFAR value about 11.24).
2. **The real-versus-real "ceiling" row is replaced.** The 10k precision/recall
   reference is a class-stratified subset of the 50k training pack, so recall
   of the 50k pack is 1.0 by construction. The ceiling row now scores the
   40,000 training images that share no image with the reference.
3. **A fourth outcome is added to the H3 rule, in advance.** The arms are
   tick-500 checkpoints (5% of the shipped 10,000-tick schedule), and the
   first scored arm shows FID 212 and recall 0.022 at NFE 1. If every arm's
   NFE-1 recall lies below CIFAR's re-derived resolution limit, then the
   step-count shortfalls cannot differ by more than that limit for any reason,
   and the launch-spec rule would read "refuted" on the measurement floor
   alone. That is not evidence about the objective. So: if all three arms'
   NFE-1 recall is below the resolution limit, the verdict is **"unresolved at
   this training budget"**, reported beside the rule's literal outcome, and
   the campaign needs longer training before H3 can be read. This adds a
   category; it does not move the thresholds of the existing three.

### Scoring result (2026-09-01): H3 is not answered at tick 500

**Verdict in one sentence.** At tick 500 (5% of the shipped 10,000-tick
schedule) none of the three arms produces measurable diversity, so H3 is
**unresolved at this training budget**; read literally, the launch-spec rule
returns **uninterpretable** (arms B and C disagree), and no clause of it
produces support for H3.

Evidence: `nfe1/runs/h3_score/20260901/` (14 rows, one host, one GPU, one set
of Inception weights, `RUN-MANIFEST.json`), report in
`nfe1/.orchestrator/tasks/h3-score-20260901-01/result.md`. Instrument: the
study's own scorer (Drifting's JAX InceptionV3, k = 3, 10,000-image
class-stratified CIFAR-10 reference), run on OrangeGrid's L40S. Instrument
checks: real CIFAR-10 Inception Score 11.273 (published about 11.24); the real
50k pack against the reference gives FID exactly 0.000; the sibling bootstrap
agrees with the upstream scorer within 0.0001.

**The fourteen rows.** Recall is the share of real images that fall inside the
generated set's neighbourhood; precision is the share of generated images
that fall inside the real set's neighbourhood.

| row | precision | recall | FID |
|---|---|---|---|
| A (M=4, distributional), 1 step | 0.3878 | **0.0218** | 211.7 |
| A, 2 / 4 / 8 steps | 0.0991 / 0.0513 / 0.0371 | 0.0000 | 261 / 271 / 276 |
| B (M=1, averaging), 1 step | 0.1862 | **0.0005** | 249.0 |
| B, 2 / 4 / 8 steps | 0.0707 / 0.0445 / 0.0312 | 0.0000 | 276 / 282 / 286 |
| C (M=1, tuple-matched), 1 step | 0.3498 | **0.0157** | 195.0 |
| C, 2 / 4 / 8 steps | 0.0327 / 0.0217 / 0.0271 | 0.0000 | 280 / 295 / 290 |
| real 40k (disjoint from the reference) | 0.6851 | **0.6906** | 0.131 |

Three facts decide the reading.

1. **More steps make these models worse.** In every arm recall drops to
   exactly zero and FID rises as the step count goes from 1 to 8. The
   shortfall the hypothesis is written about (recall at 4 steps minus recall
   at 1 step, expected positive) is negative everywhere. These checkpoints do
   not behave like the trained models H1 and H2 were measured on.
2. **The 1-step rows cannot be matched.** They are the only rows with any
   recall, and their precisions sit 0.20 apart for A versus B (ten times the
   ±0.02 tolerance). All thirteen cross-arm pairs that do fall inside the
   tolerance are at 2, 4 or 8 steps, where both members have recall 0.0000,
   so every matched pair carries a recall difference of exactly 0.0000. This
   is launch-spec risk 3 realised.
3. **The largest recall any arm reaches, 0.0218, is 32 times below the real-data
   ceiling** of 0.6906 measured on the same instrument.

**The two readings, side by side.**

- *Launch-spec rule, literal:* **uninterpretable.** Arms B and C differ in their
  4-versus-1 shortfall by +0.0153 [+0.0129, +0.0178], which survives Holm and
  is three times the 0.0057 resolution limit at that recall level. The rule's
  second clause would also fire (the averaging arm shows the smaller
  shortfall, which it calls "refuted"), but the third clause governs: a B–C
  disagreement means the A–B difference cannot be attributed to the objective.
- *Amendment of 2026-09-01, literal:* **not triggered.** It requires all three
  arms' 1-step recall to sit below the resolution limit; arm A's 0.0218 clears
  the ceiling-level limit of 0.0181 by about 0.004. We report it as
  untriggered. Its intent applies: the measurement is uninformative about
  objectives.

**CIFAR's resolution limit is not one number.** Re-deriving it by NOTE-02's
method gives 0.0181 at the real-data ceiling (within a tenth of ImageNet's
0.020, as expected for a proportion over a fixed 10,000-image reference) and
0.0057 / 0.0048 / 0.0009 on the three 1-step rows, shrinking with recall.
Any threshold test must name the recall level its limit came from.

**Two corrections carried forward.** The specified "real 50k against real 10k"
ceiling row has recall 1.0000 by construction (the reference is a subset of
the pack); the honest ceiling is the disjoint 40k row above. And
`condor_ssh_to_job` sessions on OrangeGrid die at random and kill their
children, so scoring ran as a resumable per-row script behind a reconnect loop.

**What this means for H3.** The campaign produced three trained-but-far-from-
converged models and no diversity measurement. The lock's own fallback applies
(the from-scratch matched pair at reduced size), and longer training is the
only way to read the objective contrast. Measured rates: one H200 runs an arm
at 556 s/tick, one L40S at 1,776 s/tick. Reaching 2,000 ticks costs about 309
H200-GPU-h (927 Delta service units) per arm, so a two-arm pair does not fit
the remaining 1,256 service units; on OrangeGrid it is free but about 41 days
of wall-clock on two L40S cards. A sub-5-GPU-hour probe (score arm A's saves
at ticks 100, 300 and 500 at 1 and 8 steps) would show whether recall is rising
with training at all before anything larger is paid for. **Owner decision
pending.**

### What Delta produced

1. **Frozen CIFAR-10 reference IMAGE packs**, `/work/hdd/bhvn/dli26/h3/refstats/`
   (launch spec section 9 step 1, previously never done):
   `cifar10_ref_50k.npz`, all 50,000 training images, pixel SHA-256
   `f799e8d246a8e52c...`; and `cifar10_ref_10k.npz`, 1,000 per class under
   seed 0, pixel SHA-256 `1f5375d898bdd208...`. Full digests in that
   directory's `MANIFEST.md` and per-pack JSON.

   **A trap worth carrying forward:** `cifar10-32x32.zip` is fully
   class-grouped — 50,000 entries in 10 contiguous blocks of 5,000, with only
   9 label changes between neighbours. Taking "the first 10,000" as a
   reference would give a set holding 2 of the 10 classes and would wreck any
   diversity measurement. The 10k pack is class-stratified for that reason.

   Inception FEATURES were deliberately NOT computed here: features are
   instrument-specific and belong to the Anvil scorer.

2. **Sample packs — DONE 2026-09-01 06:26 CDT**, staged at
   `/work/hdd/bhvn/dli26/h3-samples/` (job 21672369, gpua061, COMPLETED
   31 min 56 s, exit 0:0). Twelve packs: 3 arms x NFE 1/2/4/8, 50,000
   samples each, `(50000, 32, 32, 3)` uint8 under key `arr_0`, seed 0 with
   identical latents across arms (a paired design), sampled from each arm's
   tick-500 snapshot with `pushforward_generator_fn`. The two CIFAR-10
   reference packs were copied in beside them; the originals in
   `h3/refstats/` stay canonical. All 14 `.npz` files pass
   `sha256sum -c SHA256SUMS.txt`. The directory also carries a per-pack JSON
   sidecar, `MANIFEST.md`, `MANIFEST-refstats.md`, and a frozen copy of
   `gen_samples.py`. Nothing was scored on Delta.

   1-NFE pack digests (the primary row per arm):
   arm A `m4_seed0_nfe1.npz` `d88f49dd173f21ff...`;
   arm B `m1_seed0_nfe1.npz` `67e90e43089233ca...`;
   arm C `m1_tuplematched_seed0_nfe1.npz` `7655a944c5e4120f...`.
   The full 14-row digest table is in that directory's `SHA256SUMS.txt`.

   **The guidance failure is now measured, not inferred.** The recorded
   attempt raised `TypeError: zeros_like(): argument 'input' (position 1)
   must be Tensor, not NoneType` from `cfg_forward`, exactly as Finding 1
   predicted. Evidence is in the job log,
   `runs/logs/h3-gen-anvil-21672369.out`.

   **One raw-pixel observation for the scorer, NOT a result:** mean pixel
   value is 129-131 for arm A's packs and 122-126 for arms B and C. If
   precision tracks that separation, arms A and B may occupy different
   precision ranges, which is launch-spec risk 3 and would make the contrast
   unreadable. Only the Anvil scorer can settle this; the numbers above are
   pixel statistics and say nothing about precision or recall.

## Status: LOCKED 2026-08-08

The professor signed off and adopted the recommended defaults. Lock hash:
`ad85987d4b5e13c2e59c2afd4aa557fc5178338a`. These amendments came from the
week-1 and week-2 blocks below and are part of the lock.

- H1 is restated to matched-precision comparisons only. The step-count trend is
  confounded by guidance, so raw step slopes carry no evidential weight.
- The model roster drops the unreleased MeanFlow flagship and drops ROMS-IMLE.
  It adds iMF, pMF, IMM, and AFM.
- H2's within-checkpoint control is IMM's 1/2/4/8-step sampling.
- **H3 is primarily IMM's single-knob averaging-vs-distributional contrast, M=1
  vs M=4.** The matched pair trained from scratch is the fallback, used if that
  knob proves confounded.
- The JAX measurement path is canonical, because it gives byte-identical FID
  references across families.
- The caching and loader speedups may be used only after the bit-for-bit
  regression check passes.
- Treat the CVPR deadline as Nov 2026 ± 2 weeks, and re-check in September.

## Deviation 1 (2026-08-09, coordinator-approved; owner-ratified 2026-08-10)

**Who decided:** the coordinator approved this deviation. The owner ratified it
on 2026-08-10, together with all of its amendments below. Owner ratification,
verbatim: "1-NFE deviations ratified".

**What went wrong:** the lock demanded a "bit-for-bit" regression check on the
speedups. That is unattainable on this hardware for ANY run. Unmodified week-1
code drifts by the same ~1e-4 in FID across repeats and across machines. The
measured three-repeat noise floor is 1e-4–4e-4 FID and about 1e-3 recall. That
is two orders of magnitude below the ~0.1-scale effects that H1 and H2 test.

**What replaced it.** We report exact numerical differences against the noise
floor. We also added two tests that are truly exact, which noise cannot reach:
the cached reference features must be equal under `np.array_equal`, and the
labels-only loader's label stream must be equal element by element. So the
check is stronger wherever the hardware allows it.

**Result of the check:** the speedups passed and are cleared. They are 6.18×
faster. The sped-up run is CLOSER to week-1 than the un-sped-up rerun is, which
is the signature of noise rather than bias.

**Also recorded at this gate:** direct measurement CONFIRMED the guidance
explanation for the 128-step anomaly. Guided FID is 19.57, against a pass rule
of ≤22. The high-step numbers are un-quarantined.

### Correction (appended 2026-08-09, superseded the same day by isolation experiment X1)

**What we first believed:** GPU co-tenancy, meaning other jobs sharing the
card, caused the drift.

**What is actually true:** the drift is a bimodal XLA-autotuning flip. Six
measurements of one configuration fall into two tight clusters. Spread inside a
cluster is ≤5.4e-5 FID. The gap between clusters is 7.3e-3. A run executed
ALONE landed in the second cluster, so co-tenancy is demonstrably not needed to
produce the flip.

**What changes:** a comparison must now beat the between-cluster figure, not
the within-cluster one. The corrected noise floors are FID 7.3e-3, precision
0.0047, and recall 0.0014. That recall floor is ten times the within-cluster
figure we first reported.

**What does not change:** no conclusion moves. The binding constraint
everywhere is still the 0.020 reference-side resolution limit, which is 14×
the corrected recall floor. H1's shortfalls run 29–106× the corrected floor.

**Two more results from the same experiment:** the feature-saving hook was
exonerated, and the held 50k rows were released. The exclusive-card rule stays
in place as hygiene, but it is explicitly NOT sufficient for reproducibility.

**Also recorded:** the naive with-replacement bootstrap on generated samples is
structurally biased for k-NN recall. The measured bias is −0.0895. Intervals
now use reference-side resampling and paired-difference resampling instead. The
biased variant is kept only as a labeled diagnostic, and the
finite-generated-sample limitation is stated.

### Second amendment (same day)

**What went wrong:** one of the two "noise-immune exact tests" was itself
overstated. The reference-feature computation runs the same nondeterministic
kernels, so cache-versus-recompute cannot be bitwise equal. The measured
difference is ≤0.0042, which is no larger than recompute-versus-recompute.

**The honest statement:** the cache PINS the reference side. That improves
comparability between rows, by the same logic as sharing a seed. The
decision-relevant test is the effect of cached-versus-fresh features on
precision and recall. That test is G0b, and we report it whatever it shows.

**Noise floors are per measurement path.** The Shortcut path is unimodal, with
a recall floor of 0.0004; H1's evidence rests at 70–370× that floor. The
Drifting path is bimodal, with a recall floor of 0.0014.

### Third amendment (2026-08-10)

**What went wrong:** the week-1 claim that a 10k recall is a "lower bound" on
the 50k value was BACKWARDS. Recall FALLS as the number of generated samples
rises. The reason: k-nearest-neighbour radii shrink as the points get denser.
The measured fall is −0.04 to −0.10, and it depends on the model, so no single
correction can be applied to all of them.

**The rule now in force, and enforced in code:** every recall figure carries
its sample count. Never compare two recall figures unless their counts match.

**Effect on results:** none. Every contrast we reported was already
same-count.

Changes after this point are deviations and must be logged. The original draft
header follows.

## Previous status: DRAFT v1, 2026-08-03

This plan locks after the week-1 check in §8. Target venue: **CVPR 2027**. Our
deadlines table puts the deadline around Nov 13, 2026; confirm the exact date
before lock.

### Week-1 check result (2026-08-06; §8 steps 1–4 done, step 5 is the owner's)

1. **The pipeline is ours.** We ran both families end to end on our own H100,
   with 10,000 samples each and identical class labels and real reference.
   **Drifting's recall has never been published anywhere. It is 0.72 for
   latent L and 0.69 for pixel L at 1 step.** Shortcut recall rises with step
   count: 0.48 → 0.53 → 0.62 at 1, 4, and 128 steps. That is the direction H1
   predicts. These rows are NOT precision-matched, so they say nothing about H2
   yet.
2. **JAX works on our H100**, at 741 TFLOP/s of real GPU compute. Two questions
   stay open: the A100 on OrangeGrid, and the H200 on Delta. **Correction to
   §6: Anvil's A100 partition is not available to our account at all.** So the
   from-scratch pair must use the H100 partition or Delta. Note that both repos
   are JAX.
3. **Novelty holds.** We searched an 8-week window. SubFlow still has no code,
   and nobody has measured Drifting's diversity. One softening: ROMS-IMLE
   ([2607.19332](https://arxiv.org/abs/2607.19332)) publishes recall for a
   non-averaging one-step model. So "nobody has measured this" must become "no
   one has measured it across objective families." ROMS-IMLE is also a
   candidate second non-averaging model for H2.
4. **The CVPR 2027 deadline is not yet published.** The site 404s. CVPR 2027
   itself is Jun 20–24, in Seattle. Treat the deadline as Nov 2026 ± 2 weeks
   and re-check in September.
5. ⚠ **Open anomaly. Do not trust the high-step numbers yet.** Our Shortcut
   128-step FID is 40.1, but the repo publishes 15.5. We ruled out both the
   reference-statistics explanation and the CFG explanation. The released
   checkpoint may not be the one behind the README table. Recall comparisons
   are unaffected. One more item: the §4 claim that MeanFlow, iMF, and pMF
   weights are "on HF" did not survive a first search. Verify it before weeks
   2–4 depend on it.

   Full record: `code/nfe1/runs/week1_check/20260806/`.

### Week-2 preparation result (2026-08-08; read before locking)

1. **The 128-step FID anomaly is RESOLVED, and our number was right.** The
   repo's 15.5 is a guided figure — the paper states that CFG is used only at
   the smallest step size. Our 40.1 is unguided, and unguided DiT-B baselines
   land exactly there. An independent paper reran the same checkpoint with
   guidance and got 15.0. The high-step numbers are un-quarantined, pending one
   10-minute regression run. **Consequence for H1:** the week-1 rise of recall
   with steps is confounded by guidance, because guidance is baked into the
   low-step weights and absent at 128 steps. So H1 must be read at matched
   precision only, and its wording should say so at lock.
2. **Model roster corrections.** MeanFlow's flagship ImageNet weights were
   never released, so §4 must drop that claim. **iMF and pMF weights DO
   exist**: [iMF](https://huggingface.co/Lyy0725/iMF) and
   [pMF](https://huggingface.co/Lyy0725/pMF). Drifting, iMF, and pMF share a
   byte-identical FID reference file, so the JAX path gives cross-family
   comparability for free. **ROMS-IMLE is out.** It has no code and no weights,
   and its published recall of 0.50 is LOWER than its own baselines; the week-1
   note that treated it as a helpful second non-averaging model was backwards.
   **Verified replacements:** [IMM](https://huggingface.co/lumaai/imm), which
   samples at 1, 2, 4, and 8 steps from ONE checkpoint — exactly the H2
   control — and whose averaging-versus-distributional loss is a single knob,
   making a cleaner H3 than training from scratch; and AFM from ByteDance,
   which ships 50k sample packs, so recall costs nothing to compute.
3. Reference-feature caching and the parallel loader are implemented. They must
   reproduce the week-1 metrics bit-for-bit before any sweep uses them. That is
   a 15-minute GPU check.

   Ten open lock questions: `code/nfe1/runs/week2_prep/20260808/`.

Paper type: **a study that decides between two explanations, with a named
method it could make possible** (standing rules 5–6). Related plans:
[[Prereg-RoboJudge-Audit]] · [[Prereg-Crop-Consistency-Distillation]] ·
[[Prereg-Epistemic-Contextualization]].

---

## 1. The problem

One-step, or "1-NFE," image generators create a full image with one network
call instead of dozens. Examples include [MeanFlow](https://arxiv.org/abs/2505.13447), [Shortcut models](https://arxiv.org/abs/2410.12557), and now
Drifting. They make generation about 50× cheaper. Their main quality score,
Fréchet Inception Distance (FID), is now close to that of multi-step diffusion
models. However, several groups report that these one-step models are **less
diverse**: they represent fewer of the different patterns in the data.

There are two possible explanations, and each needs a different fix.

- **Intrinsic to one step:** a single network call must make every choice at
  once. Losing diversity is the price of 1-NFE, and no change to the training
  goal can remove it.
- **Caused by averaging:** most 1-NFE methods train with mean squared error
  (MSE) on average velocity. MSE can make the model learn a
  *frequency-weighted average over sub-modes*, which is called averaging
  distortion. Under this explanation, the training goal causes the collapse,
  and a goal that does not average can avoid it.

Nobody has run the experiment that tells these two explanations apart.

## 2. What recent work has shown

- **SubFlow** ([arXiv 2604.12273](https://arxiv.org/abs/2604.12273), Apr 2026) named the mechanism. It says
  that "when trained with MSE objectives, class-conditional flows learn a
  frequency-weighted mean over intra-class sub-modes." It uses sub-mode
  conditioning as a fix. **However, it tests only the averaging family:**
  MeanFlow and Shortcut. Its code is announced but not released.
- **Drifting** from the He group ([arXiv 2602.04770](https://arxiv.org/abs/2602.04770)) reaches 1-NFE
  **without an averaging objective**. It "evolves the pushforward distribution
  during training." That makes it the natural model that could disprove the
  averaging explanation, but nobody has measured its diversity. We verified one
  important practical fact in Aug 2026: the released `inference.py` in
  [`lambertae/drifting`](https://github.com/lambertae/drifting) (485★) **already calculates FID, Inception Score
  (IS), precision, and recall. The paper reports only FID and IS. We can
  calculate its unreported recall now.**
- For a matched model from the averaging family,
  [`kvfrans/shortcut-models`](https://github.com/kvfrans/shortcut-models) (762★) releases ImageNet-256 weights that can
  sample in **1, 4, or 128 steps from the same checkpoint**. MeanFlow, iMF, and
  pMF weights are also on Hugging Face (HF).
- Nearby work studies a different question. Diversity fixes for distillation,
  including [1.x-Distill](https://arxiv.org/abs/2604.04018), [Diversity-Preserved DMD](https://arxiv.org/abs/2602.03139),
  [Data-Forcing](https://arxiv.org/abs/2606.18478), and [Don't-Settle-at-the-Mode](https://arxiv.org/abs/2606.27371), stay inside the
  averaging and distillation family. The **comparison across objective families
  has not been run**, based on our Aug 2026 search.

## 3. What is new in our study

**Compare different step counts inside the same model family.** A simple
Drifting-versus-Shortcut recall comparison would be unfair, because their
quality is different: Drifting FID is 1.53, while Shortcut 1-step FID is 10.6.
That difference would be a hidden factor. Instead, each family acts as its own
control.

- For a family that supports several numbers of function evaluations, such as
  Shortcut at 1, 4, or 128 steps from one checkpoint, and the MeanFlow-family
  variants, the main measurement is the **slope of recall versus NFE at matched
  precision**.
- Drifting is built for 1-NFE. Compare it with a multi-step reference matched
  for precision. Then compare its recall *shortfall* with the averaging
  family's shortfall from 128 steps to 1 step.
- Train a **matched-budget pair from scratch** at B/4 size on ImageNet-64 or
  CIFAR. One member uses an averaging goal and the other uses the Drifting
  goal. This keeps model capacity, data, and compute the same. The released
  checkpoints differ in all three, so they cannot provide this control.

The predictions are clear. If collapse is **intrinsic to 1-NFE**, recall will
fall toward 1-NFE in every family, including Drifting. If collapse is
**specific to averaging**, only that family will show the drop, and Drifting
will keep its recall at 1-NFE. Either answer is useful.

Checked novelty: this is the first cross-family measurement of 1-NFE diversity,
the first diversity result of any kind for Drifting, and the first test of
SubFlow's explanation outside the family it was built on.

## 4. Exact experiment plan

**Models:** the released Drifting latent and pixel B & L checkpoints from
[`Goodeat/drifting`](https://huggingface.co/Goodeat/drifting); Shortcut DiT-B/XL; the MeanFlow, iMF, and pMF HF
weights; a many-step standard flow or diffusion model with matched architecture,
to serve as the precision-matched recall ceiling; and the pair we train from
scratch at B/4.

**Measurements:** precision and recall using [Kynkäänniemi k-NN](https://arxiv.org/abs/1904.06991), with k
fixed before the run. Also coverage and density. Also recall for each
ImageNet-256 class, because class-conditional coverage of sub-modes is where
averaging distortion should appear. Also distance to the nearest training
image, so that a model does not look "diverse" merely by copying. Match
precision through guidance and temperature sweeps. Use the same sample count,
50k, and fixed random seeds.

**Predictions, fixed before the run:**

- **H1:** recall decreases steadily as NFE goes to 1 inside the averaging
  family. That confirms SubFlow's starting claim in the family it studied.
- **H2, the deciding test:** Drifting's 1-NFE recall shortfall from its
  precision-matched multi-step reference is **less than half** the averaging
  family's shortfall from 128 steps to 1 step. We predict TRUE, which would
  support the averaging-specific explanation.
- **H3:** the matched pair trained from scratch shows the same ordering as H2.
  We predict TRUE. If it is FALSE, then differences between the released
  checkpoints caused H2, and we must honestly weaken the claim.
- **H4:** SubFlow-style sub-mode conditioning narrows the averaging family's
  shortfall but does not change Drifting's. We predict TRUE. If SubFlow's code
  is still unavailable, reimplement only its published recipe.

**Decision rules:** use bootstrap confidence intervals over sample splits, and
the Holm correction across H1–H4. Fix the precision-matching tolerance at
±0.02. Drop H4 without penalty if our implementation cannot reproduce
SubFlow's main published result within tolerance. That is our rule for checking
that a test model is usable: we do not publish a negative result that our own
failed reimplementation caused. H4 is only a confirmation test.

**Claims we will not make:** claims about text-to-image or video; claims about
one-step distillation methods, which use a different mechanism; and claims that
any released model is "bad." We study the class of training goals, not one
checkpoint.

## 5. What each possible result means

- **Averaging-specific, H2 TRUE:** the main message is "1-NFE does not force
  diversity collapse; the objective does." That supports training methods which
  do not average. It also shows exactly where SubFlow's fix is needed. The
  method this opens is better guidance for choosing one-step training
  objectives.
- **Intrinsic, H2 FALSE:** the main message is "one call, fewer modes: the
  diversity cost of 1-NFE does not depend on the objective." That opens a way
  to budget diversity by NFE. Measure how many steps buy how much recall, and
  recommend that uses which need broad coverage should not default to 1-NFE.
- In either case, release the cross-family diversity test harness: matched
  precision, precision and recall, coverage, and memorization checks. That is
  the common measuring tool the area currently lacks.

## 6. Resources and schedule

**Cost:** 150–350 GPU-h. Run the inference sweeps on OrangeGrid A100. Run the
from-scratch pair, about 200 GPU-h, on Anvil A100 or Delta H200. **We must
check that JAX works on H200 before we depend on Delta**, because of an earlier
hardware lesson.

Schedule for about 14 weeks before CVPR: week 1 check → weeks 2–4 checkpoint
sweeps → weeks 5–8 from-scratch pair → weeks 9–10 H4 → weeks 11–13 analysis and
writing.

## 7. Risks and competing work to watch

- The main way someone could claim this first is for SubFlow to release code
  and cross-family results. Watch versions of [`2604.12273`](https://arxiv.org/abs/2604.12273) and search again
  every 6 weeks.
- The He group could measure recall for its own Drifting model, because the
  code already sits in its `inference.py`. Move quickly: the first sweep takes
  days.
- Some checkpoint pairs may share no precision range. If that happens, report
  the reachable range honestly and rely on the matched from-scratch pair.

## 8. Week-1 check before locking the plan

1. Run Drifting and Shortcut recall once, from beginning to end. This verifies
   for ourselves that recall can be calculated now.
2. Check the JAX software on H200 and A100.
3. Repeat the literature search, and name the most recent 8 weeks directly.
4. Confirm the exact CVPR 2027 deadline.
5. Get professor approval, then mark the page LOCKED and record the git hash.

## Related

[[Unified-Direction-Ranking-2026-08]] · the removed SigLIP-2 ladder draft (git history) ·
[[GPU-Resources-Across-Clusters]]

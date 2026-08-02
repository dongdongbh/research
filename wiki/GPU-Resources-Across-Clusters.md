# GPU Resources Across Our Clusters

Added 2026-08-02. What GPU hardware we can reach on each system, what it
costs, and what to run where. Deep Anvil/Delta comparison (partitions, quotas,
queueing, accounting): [[Anvil-vs-Delta]]. Moving data between them:
[[Data-Transfer-Between-Clusters]].

## The one-table version

| | **OrangeGrid** (Syracuse) | **Anvil** (Purdue, ACCESS) | **Delta** (NCSA, ACCESS) |
|---|---|---|---|
| GPUs | L40S **or** A100, **2 per node** | 4× A100 40GB per node (16 nodes); **4× H100 80GB per node (21 nodes, "Anvil AI")** | ~100× 4-A100-40GB · ~100× 4-A40 · 6× **8-A100** · 8× **8-H200** · 1× 8-MI100 |
| Cost | **No credits** — apply and use | ACCESS credits charged | ACCESS credits charged |
| Time limits | **None** — hold nodes for as long as needed | Per-job wall-time limits | 48 h standard · 1 h interactive · **30 days on `-long` partitions** |
| Scheduler | HTCondor (jump host to log in) | Slurm | Slurm |
| Access | Bastion → login node | Direct SSH, key auth | SSH with password+Duo (no keys) |

OrangeGrid facts are our own account of the arrangement (2-card nodes, either
L40S or A100, no credit accounting, indefinite holds); the Anvil inventory is
from the official architecture page (fetched 2026-08-02); the Delta inventory
and limits are **verified from `sinfo -a` on dt-login02, 2026-08-02**.

## Delta inventory (verified via sinfo, 2026-08-02)

| Partition family | Nodes | Per node | Wall limits (std / interactive / long) |
|---|---|---|---|
| `gpuA100x4` (gpua001–100) | ~100 | 4× A100 40GB | 48 h / 1 h / **30 d** — note `gpuA100x4-long` is the **default** partition |
| `gpuA40x4` (gpub001–100) | ~100 | 4× A40 48GB | 48 h / 1 h / 30 d (+ `-preempt` 48 h) |
| `gpuA100x8` (gpuc01–06) | 6 | **8× A100 40GB** | 48 h / 1 h / 30 d |
| `gpuH200x8` (gpue01–08) | 8 | **8× H200 141GB** | 48 h / 1 h / 30 d |
| `gpuMI100x8` (gpud01) | 1 | 8× AMD MI100 | 48 h / 1 h / 30 d |
| `cpu` (cn001–137) | ~137 | 128-core, 256 GB | 48 h / 1 h / 30 d |
| `full` | spans all | — | 24 h |

Two policy facts from the login banner (2026-08-02):

- **`gpuH200x8-interactive` per-user cap:** total running jobs limited to the
  equivalent of one node-hour (e.g., one 1-h whole-node job, or eight 1-h
  1-GPU jobs at ≤1/8 of RAM/CPU each). Each job single-node.
- **Proportional charging:** allocating a larger % of RAM or CPUs than of
  GPUs charges at the larger fraction — 1 GPU + all 96 CPUs bills as the
  whole node. Shape requests to match the GPU fraction.

## Anvil inventory (verified from docs, 2026-08-02)

- **Anvil GPU:** 16 nodes — 4× NVIDIA A100 40GB, 2× AMD EPYC 7763 (128
  cores), 512 GB RAM, 100 Gbps IB.
- **Anvil AI:** 21 nodes — 4× NVIDIA **H100 80GB**, Intel Xeon 8468 (96
  cores), 1 TB RAM, dual 400 Gbps IB. This is the best training hardware we
  have credit-metered access to.
- CPU side: 1,000 nodes of 128-core EPYC 7763 / 256 GB.

## Placement guidance

**OrangeGrid = the unbounded-time tier.** No credits and no wall-time limit
change what is feasible:

- **Long-running jobs that would blow a Slurm wall clock** — multi-day RL
  campaigns, long sweeps, monitors, anything that hates checkpoint-restart.
- **Credit conservation** — anything that fits in 2× A100 or 2× L40S should
  default to OG before spending ACCESS credits.
- **Isaac Sim workloads**: OG's **L40S has RT cores**, so it is our *only*
  no-credit Isaac-capable hardware (Isaac does not run on A100/H100/H200 —
  see the note in [[Anvil-Interactive-GPU-Workflow]]). MuJoCo/SAPIEN runs on
  everything.
- Constraint to respect: 2 cards per node means no single-node 4/8-GPU jobs;
  multi-node HTCondor GPU jobs are a different (and worse) story than Slurm —
  treat OG as a ≤2-GPU-per-job system.

**Anvil = the default credit-metered tier.** Direct SSH, key auth, our
project storage lives here, and the H100 (Anvil AI) partition covers serious
training. Wall-time limits apply — design jobs to checkpoint.

**Delta = the big-shape and long-wall tier.** The `sinfo` audit upgraded
Delta's role in two ways:

- **Largest single-node shapes we can reach:** 8× H200 141GB (gpue01–08) —
  more GPU memory per node (1.1 TB) than anything on Anvil — plus 6× 8-A100
  nodes. For jobs needing >4 GPUs or >320 GB GPU memory in one box, Delta is
  the only option.
- **30-day `-long` partitions on every GPU type** (and `gpuA100x4-long` is
  the default partition). So "Delta = short wall clocks" is wrong: month-long
  credit-metered jobs are possible. OrangeGrid remains the *no-credit*
  long-run tier; Delta is the *credit-metered* one for jobs that also need
  4–8 GPUs per node.

Costs credits; password+Duo SSH makes automation clumsy — stage data via
Globus ([[Data-Transfer-Between-Clusters]]). Mind the proportional-charging
rule above. Charge factors and queue behavior:
[[Delta-Setup-and-Parallel-Workflow]] and [[Anvil-vs-Delta]].

**Rule of thumb:** prototype and debug interactively on Anvil
([[Anvil-Interactive-GPU-Workflow]]); park long/unbounded ≤2-GPU work on
OrangeGrid (free); spend credits on Anvil-AI H100s for standard training;
go to Delta for 8-GPU single-node shapes, H200 memory, or multi-day-to-
month walls that exceed Anvil's limits.

## References

- Anvil resource page: allocations.access-ci.org/resources/anvil.purdue.access-ci.org
- Anvil architecture: docs.rcac.purdue.edu/userguides/anvil/architecture/
- Delta resource page: allocations.access-ci.org/resources/delta.ncsa.access-ci.org
- Delta architecture: docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/architecture.html
- OrangeGrid HTCondor: su-jsm.atlassian.net/wiki/spaces/RESCOMP/pages/164169368

## Related

[[Anvil-vs-Delta]] · [[Data-Transfer-Between-Clusters]] ·
[[Anvil-Interactive-GPU-Workflow]] · [[Delta-Setup-and-Parallel-Workflow]] ·
[[CUDA-Compatibility-and-vLLM]]

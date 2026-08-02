# GPU Resources Across Our Clusters

Added 2026-08-02. What GPU hardware we can reach on each system, what it
costs, and what to run where. Deep Anvil/Delta comparison (partitions, quotas,
queueing, accounting): [[Anvil-vs-Delta]]. Moving data between them:
[[Data-Transfer-Between-Clusters]].

## The one-table version

| | **OrangeGrid** (Syracuse) | **Anvil** (Purdue, ACCESS) | **Delta** (NCSA, ACCESS) |
|---|---|---|---|
| GPUs | L40S **or** A100, **2 per node** | 4× A100 40GB per node (16 nodes); **4× H100 80GB per node (21 nodes, "Anvil AI")** | A100 40GB 4-way and 8-way, A40 4-way, MI100 (see refs) |
| Cost | **No credits** — apply and use | ACCESS credits charged | ACCESS credits charged |
| Time limits | **None** — hold nodes for as long as needed | Per-job wall-time limits | Per-job wall-time limits |
| Scheduler | HTCondor (jump host to log in) | Slurm | Slurm |
| Access | Bastion → login node | Direct SSH, key auth | SSH with password+Duo (no keys) |

OrangeGrid facts above are our own account of the arrangement (2-card nodes,
either L40S or A100, no credit accounting, indefinite holds); the Anvil
inventory is from the official architecture page (fetched 2026-08-02); the
Delta row should be verified against the linked docs — the architecture page
is JS-rendered and resisted scraping.

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

**Delta = the overflow / special-shape tier.** 8-way A100 nodes for jobs that
need >4 GPUs in one box; A40s for cheaper inference. Costs credits, and the
password+Duo SSH makes automation clumsy — stage data via Globus
([[Data-Transfer-Between-Clusters]]) and prefer Anvil when shapes are equal.
Partition charge factors and queue behavior: [[Delta-Setup-and-Parallel-Workflow]]
and [[Anvil-vs-Delta]].

**Rule of thumb:** prototype and debug interactively on Anvil
([[Anvil-Interactive-GPU-Workflow]]); park long/unbounded ≤2-GPU work on
OrangeGrid; spend credits on Anvil-AI H100s for training that needs them;
reach for Delta only for 8-GPU single-node shapes or when Anvil queues are
bad.

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

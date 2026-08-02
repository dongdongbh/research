# Anvil vs. Delta, DeltaAI, and Bridges-2 for SVIB2

Purpose: which cluster each SVIB2 workload should run on, with first-party
hardware, queue, accounting, and storage evidence for each candidate system.

Status: 2026-07-17. This comparison uses first-party RCAC, NCSA, PSC, and
ACCESS documentation. Hardware and queue policy can change; check `sinfo` and
the resource page before submitting a paper-scale run.

## Decision

Keep **Anvil as SVIB2's primary system now**. The repository, Python
environment, generated corpus, feature artifacts, and benchmark metadata are
already there; COCO and Visual Genome are available in Anvil's shared AI
dataset collection; and AnvilGPT provides the teacher endpoint directly.
[RCAC lists COCO and Visual Genome](https://docs.rcac.purdue.edu/datasets/ai/),
and [documents the AnvilGPT chat-completions API](https://docs.rcac.purdue.edu/userguides/anvil/anvilgpt/#api).

Use the resources as follows:

| SVIB2 workload | First choice | Why | When to move elsewhere |
|---|---|---|---|
| Qwen teacher API generation | **AnvilGPT from an allocated CPU compute job**, or a non-HPC client | No local GPU is required. Anvil explicitly says Slurm must be used for work and production work on login nodes violates policy. [Anvil job policy](https://docs.rcac.purdue.edu/userguides/anvil/jobs/#overview-slurm-basics) | Move the API client only if another legitimate CPU execution host is easier. Moving to another GPU system does not speed a remote rate-limited API. |
| Local Qwen3.6-27B/vLLM | **One Anvil H100 first** | Lowest migration cost; Anvil has 21 four-H100, 80-GB nodes and permits shared GPU nodes. [Anvil architecture](https://docs.rcac.purdue.edu/userguides/anvil/architecture/#compute-nodes), [accounting](https://docs.rcac.purdue.edu/userguides/anvil/jobs/#gpu-nodes) | Try one Delta H200 if 80 GB is genuinely too tight, or DeltaAI after an ARM container smoke. Do not request multiple nodes for this 27B model. |
| COCO/VG feature extraction | **Anvil H100/A100** | Images already live in the documented shared dataset collection, avoiding a large data transfer. Anvil supplies 100-TB scratch and 5-TB project space. [Datasets](https://docs.rcac.purdue.edu/datasets/ai/), [storage](https://docs.rcac.purdue.edu/userguides/anvil/architecture/#storage) | DeltaAI is the best overflow target only after an `aarch64` environment passes and the data is staged. |
| Compact verifier training | **One Anvil H100** | SVIB2's verifier is small; one GPU avoids distributed-training complexity and the repo already works on Anvil. | Use DeltaAI for a measured multi-GPU scaling need; it has the clearest documented NCCL/Slingshot setup. [DeltaAI NCCL guide](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/prog-env.html#nccl) |
| Large-backbone or memory-bound experiment | **Delta H200** as a secondary allocation | Each H200 has 141 GB, the largest single-GPU memory among these choices. [Delta H200 specifications](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/architecture.html#way-nvidia-h200-gpu-large-memory-compute-node-specifications) | Delta has only eight H200 nodes and its H200 partition has a 3x charge factor, so use it for a demonstrated memory need rather than routine SVIB2 jobs. [Delta queues](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/running_jobs.html#delta-partitions-queues) |

The practical ranking is therefore:

1. **Anvil now**: easiest, data-local, x86-compatible, and already validated.
2. **Delta**: best secondary system for a single very-large-memory H200 job;
   otherwise its A100 generation is not an upgrade over Anvil's H100s.
3. **DeltaAI**: strongest documented multi-node AI fabric and much larger H100
   fleet, but its ARM host architecture creates real packaging risk.
4. **Bridges-2**: credible x86 H100 overflow capacity, but no SVIB2-specific
   advantage large enough to repay a second data/software migration today.

## Hardware and node granularity

| System | Relevant accelerators | Smallest useful request | Multi-GPU topology and implication |
|---|---|---|---|
| **Anvil** | 21 nodes with 4× H100 80 GB, 96 CPU cores and 1 TB RAM; 16 nodes with 4× A100 40 GB. [Official specifications](https://docs.rcac.purdue.edu/userguides/anvil/architecture/#compute-nodes) | One GPU; RCAC says all GPU nodes are shared. [Accounting](https://docs.rcac.purdue.edu/userguides/anvil/jobs/#gpu-nodes) | H100 nodes have dual 400-Gbps InfiniBand in the architecture table. [Architecture](https://docs.rcac.purdue.edu/userguides/anvil/architecture/#compute-nodes) |
| **Delta** | 100 4× A100-40 nodes, 100 4× A40-48 nodes, six 8× A100-40 nodes, and eight 8× H200-141 nodes. [Official specifications](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/architecture.html#computational-compute-nodes) | Shared-node scheduling is supported; GPU accounting can be fractional by GPU, host cores, or host memory. [Architecture](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/architecture.html), [accounting](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/job_accounting.html) | A100 and H200 SXM GPUs are NVLink-connected within their nodes; nodes use 200-Gbps Slingshot 11. [Topology and network](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/architecture.html) |
| **DeltaAI** | 152 nodes, each with 4× GH200: one 72-core Grace ARM CPU, 120 GB CPU memory, and one H100 with 96 GB per superchip. [Official specifications](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/architecture.html#deltaai-gh200-compute-nodes) | One GH200 is the smallest allocatable unit. [Architecture](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/architecture.html) | Four NVLink-connected GH200s per node, four Slingshot NICs, and a documented NCCL OFI plugin for cross-node training. [Topology](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/architecture.html#way-nvidia-gh200-gpu-compute-node-specifications), [NCCL](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/prog-env.html#nccl) |
| **Bridges-2** | 10 nodes with 8× H100-80, three with 8× L40S-48, 24 with 8× V100-32, nine with 8× V100-16, plus one 16× V100-32 DGX-2. [PSC hardware page](https://www.psc.edu/resources/bridges-2/) | `GPU-shared` permits 1–4 GPUs; the full-node `GPU` partition allocates eight GPUs. [PSC user guide](https://www.psc.edu/resources/bridges-2/user-guide/#gpu-partitions) | H100 nodes provide eight GPUs and the system uses HDR-200 InfiniBand. [PSC hardware page](https://www.psc.edu/resources/bridges-2/) |

### How many GPUs for Qwen3.6-27B?

Start with **one H100, one node, tensor parallel size 1**. A dense 27-billion
parameter model has about 54 GB of raw BF16 weights (`27e9 × 2` bytes); an
80-GB H100 therefore has plausible but not guaranteed room for engine state and
KV cache, while DeltaAI's 96-GB H100 and Delta's 141-GB H200 provide more
headroom. This is an engineering inference, not a capacity guarantee: the real
answer depends on the exact checkpoint, context length, concurrency, dtype, and
vLLM version. Measure maximum GPU memory and tokens/s in the one-GPU benchmark
before adding GPUs.

Four GPUs on one node can raise aggregate serving throughput if vLLM has enough
concurrent work, but tensor parallelism also adds communication. Multiple nodes
are inappropriate for this first benchmark: they add scheduler cost and network
coordination without solving a demonstrated capacity problem. DeltaAI is the
best-documented option if multi-node training later becomes necessary because
its guide explicitly configures the NCCL OFI plugin for Slingshot.
[DeltaAI NCCL documentation](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/prog-env.html#nccl)

## Scheduler and accounting

| System | Production/debug access | Charging facts that matter to SVIB2 |
|---|---|---|
| **Anvil** | `ai` and `gpu` allow 48 hours; `gpu-debug` allows two GPUs for 30 minutes; the per-user limit is 12 GPUs. [Queue table](https://docs.rcac.purdue.edu/userguides/anvil/jobs/#anvil-queues-partitions) | One SU is one GPU-hour, and GPU nodes are shared. Charges follow reserved resources, not actual utilization. [Accounting](https://docs.rcac.purdue.edu/userguides/anvil/jobs/#job-accounting) CPU, GPU, and AI credits are separate queue allocations. [Queue access](https://docs.rcac.purdue.edu/userguides/anvil/jobs/#overview-slurm-basics) |
| **Delta** | Production queues allow 48 hours; interactive GPU queues allow one hour. A100/A40 preemptible queues cost half rate but require checkpointing. [Queue table and preemption policy](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/running_jobs.html#partitions-queues) | A100x4 has factor 1, A40x4 factor 0.5, and H200x8 factor 3; H200 jobs are limited to one node. [Queue table](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/running_jobs.html#delta-partitions-queues) CPU and GPU charge accounts are separate. [Accounting](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/job_accounting.html) |
| **DeltaAI** | `ghx4` allows 48 hours; `ghx4-interactive` allows two hours and at most four nodes, with an eight-node-hour per-user running budget. [Queue table](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/running-jobs.html#partitions-queues) | One SU is one reserved GH200-hour; CPU cores or host memory can force allocation of additional GH200 units. [Accounting](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/job-accounting.html) Interactive jobs have factor 2. [Queues](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/running-jobs.html#partitions-queues) |
| **Bridges-2** | `GPU-shared` supports up to four GPUs for 48 hours; the full-node `GPU` partition supports up to 64 GPUs for 48 hours. [PSC partition summary](https://www.psc.edu/resources/bridges-2/user-guide/#gpu-partitions) | V100 and L40S cost 1 SU/GPU-hour; H100 costs 2 SU/GPU-hour. [PSC accounting](https://www.psc.edu/resources/bridges-2/user-guide/#accounting-for-bridges-2-use) |

All four sites treat login nodes as orchestration hosts, not production compute:
[Anvil](https://docs.rcac.purdue.edu/userguides/anvil/jobs/#overview-slurm-basics),
[Delta](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/citizenship.html),
[DeltaAI](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/citizenship.html),
and [Bridges-2](https://www.psc.edu/resources/bridges-2/user-guide/#running-jobs).
For the long AnvilGPT corpus run, the clean solution is a small Anvil CPU
allocation and a resumable CPU Slurm job—not a reserved H100 and not a
multi-day login-node process.

ACCESS credits must be exchanged into resource-specific allocations, and the
provider may approve, modify, or decline an exchange. Use the live calculator
rather than copying a rate into a paper or script.
[ACCESS exchange policy/calculator](https://allocations.access-ci.org/exchange_calculator)
([Anvil](https://allocations.access-ci.org/resources/anvil.purdue.access-ci.org),
[Delta](https://allocations.access-ci.org/resources/delta.ncsa.access-ci.org),
[DeltaAI](https://allocations.access-ci.org/resources/deltaai.ncsa.access-ci.org),
[Bridges-2](https://allocations.access-ci.org/resources/bridges2.psc.access-ci.org)).

## Storage and data locality

| System | Persistent/work storage | SVIB2 consequence |
|---|---|---|
| **Anvil** | Home 25 GB with snapshots, scratch 100 TB with no backup and a 30-day purge, project 5 TB with daily snapshots. [RCAC storage table](https://docs.rcac.purdue.edu/userguides/anvil/architecture/#storage) | Best current locality: the repo and artifacts are already in project space and COCO/VG can remain symlinks into the shared collection. Use scratch for transient extraction, not irreplaceable H5 caches. |
| **Delta/DeltaAI** | Home 100 GB; projects start at 500 GB and can be requested up to 25 TB; work starts at 1 TB and can be requested up to 100 TB. DeltaAI says HDD and NVMe work areas are shared between Delta and DeltaAI. [Delta storage](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/data_mgmt.html#file-systems), [DeltaAI storage](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/data-mgmt.html#file-systems) | Attractive if both NCSA systems are used because one staged corpus can serve both. Projects/work have no snapshots, so keep durable copies elsewhere. |
| **Bridges-2** | Each allocation includes persistent Ocean storage; node-local `$LOCAL` is deleted after the job. [PSC file-space guide](https://www.psc.edu/resources/bridges-2/user-guide/#file-spaces) | Fine operationally, but SVIB2's image data, model cache, repo, and features must all be transferred and rebuilt or relinked. |

Use Globus for a migration. Anvil mounts its file systems on its Globus
endpoints, while Delta and DeltaAI expose data-transfer endpoints.
[Anvil storage](https://docs.rcac.purdue.edu/userguides/anvil/architecture/#storage),
[Delta data management](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/data_mgmt.html#transferring-data),
[DeltaAI architecture](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/architecture.html#specialized-nodes).

## Software and migration friction

- **Anvil and Delta are x86-64**, so the current `uv` lock and CUDA wheels have
  the best chance of transferring between them. Both offer Lmod and NVIDIA
  container support. [Anvil software](https://docs.rcac.purdue.edu/userguides/anvil/anvil-software/),
  [Delta software](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/software.html),
  [Delta Apptainer](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/containers.html).
- **DeltaAI is `aarch64`**. NCSA explicitly warns that many Python packages may
  lack ARM wheels and may need source builds; it nevertheless supplies
  Apptainer/NGC images and a local vLLM installation recipe. This is the main
  reason DeltaAI is not the easiest first move for SVIB2.
  [DeltaAI Python warning](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/software.html#python),
  [containers](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/containers.html),
  [custom recipes](https://docs.ncsa.illinois.edu/systems/deltaai/en/latest/user-guide/software.html#custom-recipes-for-python).
- **Bridges-2 is x86-64** and provides CUDA modules, PSC-built AI environments,
  Singularity/NGC containers, and H100s. It is likely easier to port to than
  DeltaAI but still requires complete data and environment staging.
  [PSC user guide](https://www.psc.edu/resources/bridges-2/user-guide/#using-the-gpu-nodes),
  [containers](https://www.psc.edu/resources/bridges-2/user-guide/#containers).

## External API and model-serving caveat

The official manuals document high-speed WAN/data-transfer connectivity, but
they do **not** make a sufficiently explicit, uniform promise that arbitrary
outbound HTTPS from every compute partition will remain available. Before
moving teacher generation, run a short compute-node smoke against the exact
API endpoint and ask site support if a long-lived client is acceptable. Do not
infer API suitability from successful `curl` on a login node.

AnvilGPT is the exception relevant to this project: RCAC directly documents its
authenticated `https://anvilgpt.rcac.purdue.edu/api/chat/completions` endpoint.
[AnvilGPT API](https://docs.rcac.purdue.edu/userguides/anvil/anvilgpt/#api)

For a local vLLM service, keep the server and its clients inside one Slurm job
or node whenever possible. Long-lived public serving is a different workload
from batch inference and may require a site's persistent-service facility;
none is needed for the current offline SVIB2 corpus generation.

## Next action

Status (2026-07-19): item 1 completed on 2026-07-17 (see
[[Anvil-H100-Qwen36-vLLM-Benchmark]]); the local-vs-hosted question is settled
in [[Current-Standard]] — local generation runs at about 18x hosted speed and
is authorized for attribute/spatial targets only.

1. Finish the approved **single-Anvil-H100** vLLM benchmark and record startup
   memory, peak memory, accepted pairs/minute, and SU per 1,000 accepted pairs.
2. Compare it with the existing AnvilGPT API rate using the same 100 captions.
3. If local vLLM wins, run one H100 with multiple request streams; add GPUs only
   after a 1-vs-2-vs-4 GPU throughput benchmark shows lower SU per accepted row.
4. Request/exchange a small Anvil **CPU** allocation for future remote-API
   generation so no production workload runs on login nodes and no idle H100 is
   reserved.
5. Revisit DeltaAI only if feature extraction or verifier training demonstrates
   useful multi-GPU scaling; first require a complete `uv run pytest` and vLLM
   container smoke on ARM.

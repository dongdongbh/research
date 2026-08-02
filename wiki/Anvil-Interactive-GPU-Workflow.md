# Anvil Interactive GPU Workflow

Updated 2026-08-02 for the general research wiki.

This is the default lab workflow for GPU work that needs iterative debugging,
smoke tests, and then a real run. Request one sufficiently long interactive
allocation, do all related work inside it, and release it or explicitly hand it
to the project owner when the planned work finishes. A 48-hour request is a
ceiling, not a reason to keep an unowned node for 48 hours.

## When to use it

Use an interactive GPU allocation for work such as:

- qualifying a new local model or fixing a vLLM launch;
- extracting frozen image/node features;
- running a training/eval MVP and its controls; or
- any workflow where a short smoke may reveal a problem before the full run.

API-only teacher generation and other CPU/network orchestration do not need an
AI node. They can run on a login node when permitted by RCAC policy. Do not run
GPU workloads directly on a login node.

## Size the request

Request only resources the program can use. A compact training run and a single
model server normally need one H100. Asking for four GPUs or multiple nodes does
not increase throughput unless the program is explicitly configured for tensor,
pipeline, or data parallelism.

There is one owner-approved exception (2026-07-18): when the exact parallel
launch is already specified and per-GPU throughput is already benchmarked from
a prior measured run, request the full multi-GPU shape in a single interactive
allocation — the smoke still runs on one GPU inside it before the parallel
launch, avoiding a requeue between smoke and full run. First use was one node,
4x H100, 32 CPUs, 192 GiB, 48-hour ceiling, for shard-parallel feature
extraction with independent per-GPU workers and resumable outputs. Full detail:
svib2 repo wiki, page Anvil-Interactive-GPU-Workflow.

### GPU-type constraints (verified Aug 2026)

NVIDIA Isaac Sim requires RT cores and therefore does **not** run on A100,
H100, or H200. On this cluster it is L40S-only; request an L40S explicitly for
any Isaac Sim / Omniverse workload. MuJoCo- and SAPIEN-based stacks (LIBERO,
RoboCasa, MetaUrban) have no RT-core dependency and run on all GPU types here.

## Request one 48-hour interactive allocation

Run the allocation command inside `tmux` so a dropped SSH connection does not
kill the `salloc` process:

```bash
tmux new -s gpu-<project>

salloc \
  --account=cis261253-ai \
  --partition=ai \
  --qos=ai \
  --nodes=1 \
  --ntasks=1 \
  --cpus-per-task=16 \
  --gres=gpu:1 \
  --mem=96G \
  --time=2-00:00:00 \
  --job-name=<project>-mvp
```

The job may remain `PENDING` until a node is available. Pending time does not
consume the allocated GPU. Record the job ID printed by `salloc`:

```bash
export JOBID=<job-id>
squeue -j "$JOBID"
scontrol show job "$JOBID"
```

Do not submit duplicate allocations merely because the first is pending. Cancel
and replace the original only when its resource shape or time limit is wrong.
The only sanctioned second request is a hedged scale-out with recorded cancel
criteria (see below).

## Enter and validate the allocation

After `salloc` reports that the allocation was granted, start a shell step on
the allocated node:

```bash
srun --jobid="$JOBID" --overlap --pty bash -l
```

Using `srun` is the reproducible default; it avoids assumptions about whether
direct SSH to the allocated hostname is enabled. In the node shell:

```bash
hostname
nvidia-smi
cd /anvil/projects/x-cis261253/code/<project>
source /home/x-dli26/.config/zsh/api_keys   # Anvil account: x-dli26
uv run python - <<'PY'
import torch
print(torch.__version__)
print(torch.cuda.is_available())
print(torch.cuda.get_device_name(0))
PY
```

Never interpolate an API or Hugging Face token into a command-line argument:
command arguments can appear in `ps`, Slurm diagnostics, and logs. Let the
library read `HF_TOKEN` or another key from the environment, or use its
credential store. If a secret ever appears in a command/process listing,
terminate that launch and rotate the credential.

Keep durable code, manifests, feature caches, and selected outputs under the
project allocation (`/anvil/projects/x-cis261253/...`). Put disposable
materialized images, downloads, and other rebuildable intermediates under
`$SCRATCH/<project>` (`$SCRATCH` is `/anvil/scratch/x-dli26` for this account).

## Smoke, inspect, then run

Use the same allocation for the whole debugging loop:

1. Run the smallest meaningful smoke (one to five images or rows). In a
   multi-GPU allocation, the smoke still runs on one GPU before the parallel
   launch.
2. Inspect output schema, dimensions, provenance, and GPU memory.
3. Run focused tests for any fix made during the smoke.
4. Start the full resumable command only after the smoke passes.
5. Run the matching control with the same split and seed.
6. Verify manifests, metrics, tests, and remaining processes before cleanup.

Prefer resumable outputs and keep logs outside the ephemeral node-local
filesystem. A 48-hour allocation avoids requeueing after each correctable
failure, while early release ensures the project is charged only for time it
actually holds the resource.

For tracked training, authenticate W&B at an interactive prompt with
`uv run wandb login --relogin`; never paste the key into a command argument or
checked-in configuration. Pass the run's W&B project, run name, and online mode
through the training entry point. Use `offline` when the compute node cannot
reach W&B and sync the local run later.

## Hedged scale-out

A second identical request may be submitted while the first allocation is
still running, but only when measured queue latency exceeds the time to the
decision gates that determine the second node's workload. Record the hedge at
submission time: the named uses, the decision gates, and the cancel criteria.
If every gate resolves negative before the request grants, cancel it; if it
grants before the gates resolve, release it immediately unless a gate has
named its work. A pending or promptly released request never idles an unowned
node.

First use (2026-07-18, owner-approved): a second 4x H100 shape was submitted
while the first allocation ran, because measured queue latency (~8 hours)
exceeded the time to its two named decision gates. Both gates resolved
negative and the request was cancelled per its recorded criteria; no unowned
node was ever held. Full detail: svib2 repo wiki, page
Anvil-Interactive-GPU-Workflow.

## Release or hand off explicitly

Exiting an `srun` shell does not necessarily release its parent `salloc`
allocation. When all GPU work and checks are complete, cancel the allocation
explicitly from either login shell:

```bash
scancel "$JOBID"
squeue -j "$JOBID"
sacct -j "$JOBID" --format=JobID,JobName,State,Elapsed,AllocTRES
```

The first verification command should show no active job after Slurm processes
the cancellation. Also check that no forgotten project GPU jobs remain:

```bash
squeue -u "$USER"
```

Then detach or close the `tmux` session. The only exception is an explicit
same-allocation handoff: if the project owner has named additional imminent
tests, report the job ID and node, leave the allocation active, and let the
owner decide when to cancel it. Never leave an idle interactive node running
without an identified owner and next use.

## Resource rule

- Login node: API calls, corpus assembly, validation, manifests, and light tests.
- One H100: model qualification, selected-image feature extraction, and compact
  training/control runs.
- Multiple GPUs: only after a one-GPU smoke passes and the exact parallel launch
  has been specified and benchmarked. When both already hold from a prior
  measured run, the full multi-GPU shape may be requested as one allocation
  (smoke first on one GPU inside it), per "Size the request" above.
- Multiple nodes: only for software explicitly designed for multi-node
  execution, and a second hedged request only under the recorded-cancel-criteria
  discipline above.

The standard approved shape for a single-model interactive session is one H100,
16 CPU cores, 96 GiB RAM, and a 48-hour ceiling. Release it once the full run
and its matching control finish, unless the project owner explicitly retains the
same allocation for a named follow-up test.

# Delta Setup and Parallel Workflow

Updated 2026-08-02 for the general research wiki.

Purpose: the observed NCSA Delta bootstrap state and the rules for running a
project on Delta as a second execution site in parallel with Anvil.

Status: live bootstrap snapshot from `dt-login03.delta.ncsa.illinois.edu` on
2026-07-18 (taken while bootstrapping the svib2 repo), reconciled against the
current NCSA Delta Red Hat 9 documentation. This page records the **actual
Delta installation** plus the workflow for using Delta as a second execution
site for any of the lab's projects. Git/GitHub coordinates source history;
Anvil remains authoritative for existing verified corpora and Anvil-specific
generated artifacts.

Account note: the Delta account is `dli26` (Anvil account: `x-dli26`). Paths
below are Delta-side unless stated otherwise; substitute your own username and
allocation where they appear. `<project>` stands for the repository being
bootstrapped (svib2 in the snapshot).

## Current state at a glance

| Item | Observed Delta state on 2026-07-18 |
|---|---|
| User/login | `dli26` on `dt-login03.delta.ncsa.illinois.edu` |
| Allocation | `bhvn`; group `delta_bhvn` |
| Charge accounts | `bhvn-delta-gpu` is active; no CPU charge account is currently listed |
| Repository | `/projects/bhvn/dli26/code/<project>`, branch `master`, at a recorded base commit |
| Dataset root | `/work/hdd/bhvn/dli26/datasets` |
| Download staging | `/work/nvme/bhvn/dli26/dataset-downloads` |
| Python tooling | `uv 0.11.29`; uv-managed CPython `3.12.13` |
| Project environment | `.venv`, 123 compatible packages, about 7.6 GB |
| ML stack | PyTorch `2.11.0+cu128`, torchvision `0.26.0+cu128`, locked Transformers Git revision `f007617e...` |
| JavaScript tooling | Node `v24.18.0`, npm/npx `11.16.0`, installed under `/u/dli26/.local` |
| Test status | imports and `uv pip check` pass; 406 tests collect; the full suite and GPU smoke remain pending |
| Dataset status | SugarCrepe, SugarCrepe++, VSR, and NegCLIP metadata ready; COCO ready; Visual Genome download still in progress at snapshot time |

## Recommendation

Delta works as a practical second site. It is an x86-64 NVIDIA system whose
current base CUDA is 12.8, matching CUDA 12.8 PyTorch wheels pinned in
`pyproject.toml`. The repository environment was rebuilt locally from
`uv.lock`; no Anvil `.venv`, cache, absolute symlink, or Slurm script was
copied.

Delta does **not** provide AnvilGPT or Anvil's shared COCO/Visual Genome
collection. Rows already generated through AnvilGPT remain reusable after
transfer, but new hosted-LLM work on Delta must use a self-hosted
OpenAI-compatible server or an independently authorized external API.
Image-dependent work must use data materialized, downloaded, or transferred to
Delta.

The bootstrap has completed repository setup, Python/Node tooling, locked
dependency installation, metadata download, and COCO staging. The next gating
sequence is:

1. finish and validate Visual Genome;
2. run the full CPU test suite in an approved short context;
3. run a one-GPU CUDA/OpenCLIP/H5 smoke using `bhvn-delta-gpu`;
4. transfer one known verified-data bundle and compatible feature H5;
5. reproduce one known verifier result before assigning new parallel shards;
6. request a CPU charge account before scheduling API-only CPU jobs.

Delta login nodes are for editing, compilation, and job management, not
production computation. NCSA may automatically stop application-like login
node processes. The owner explicitly chose the login node for the one-time,
resumable public COCO/Visual Genome archive download described below; model
inference, feature extraction, training, full tests, and long API generation
still belong in Slurm jobs.
[Delta good-citizenship policy](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/citizenship.html)

## 1. Allocation, identity, and login

The NCSA identity and Duo login are working for user `dli26`. The live
allocation group is `delta_bhvn`. On 2026-07-18, `accounts` listed only:

```text
bhvn-delta-gpu
```

That account is the only currently authorized charge target. Do not submit the
CPU examples later in this page until a CPU account also appears in `accounts`.
NCSA uses separate CPU and GPU charging accounts, and charges reserved
resources rather than observed utilization.
[Delta job accounting](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/job_accounting.html)

```bash
ssh dli26@login.delta.ncsa.illinois.edu
```

Record the live site state before each substantial run:

```bash
accounts
quota
module reset
module list
sinfo -o '%P %a %l %D %G'
```

The preferred hostname round-robins over the login nodes. For a persistent
`tmux` session, note the concrete `dt-login01` through `dt-login04` hostname and
reconnect to that host rather than the round-robin alias. Login nodes currently
enforce per-user CPU and memory cgroups and have no GPU.
[Delta login limits and tmux](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/login.html)

## 2. Delta storage layout

Delta does not have Anvil's `/anvil/projects`, `/anvil/scratch`, or shared RCAC
COCO/VG mounts. The active layout is deliberately flat under the per-user
allocation directories; there is no per-project work root such as
`/work/hdd/bhvn/dli26/<project>`.

| Role | Active path | Use |
|---|---|---|
| Home | `/u/dli26` | uv/Node binaries, uv-managed Python, current uv cache, shell configuration, secrets |
| Project | `/projects/bhvn/dli26` | repository and `.venv`; future manifests/checkpoints/runs as appropriate |
| Work HDD | `/work/hdd/bhvn/dli26` | durable-for-allocation dataset mirror at `datasets/`; future model/artifact roots |
| Work NVMe | `/work/nvme/bhvn/dli26` | resumable sparse Git staging at `dataset-downloads/`; future high-IOPS scratch/cache |
| Node local | `/tmp`, about 0.74 TB on CPU or 1.5 TB on GPU nodes, removed after the job | per-job extraction, LMDB staging, temporary shards |

These values and policies are from the current
[Delta data-management table](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/data_mgmt.html#file-systems).
Delta storage is for active computation, not archival storage, and access has a
30-day data-management grace period after project expiration.
[Delta allocation/storage policy](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/accounts.html#allocation-policies)

Use explicit variables instead of relying on compatibility meanings of `$WORK`
or `$SCRATCH`:

```bash
export DELTA_ALLOCATION=bhvn
export DELTA_PROJECT=/projects/bhvn/dli26
export DELTA_WORK_HDD=/work/hdd/bhvn/dli26
export DELTA_WORK_NVME=/work/nvme/bhvn/dli26
export PROJECT_REPO=/projects/bhvn/dli26/code/<project>
export DATASETS=/work/hdd/bhvn/dli26/datasets
export DATASET_STAGING=/work/nvme/bhvn/dli26/dataset-downloads
```

Observed quota after environment installation and partial image staging:

| Allocation path | Used | Soft quota | File use |
|---|---:|---:|---:|
| `/u/dli26` | 10.439 GB | 100 GB | 73,451 / 750,000 |
| `/projects/bhvn` | 7.602 GB | 500 GB | 38,042 / 750,000 |
| `/work/nvme/bhvn` | 54.34 MB | 500 GB | 123,642 / 850,000 |
| `/work/hdd/bhvn` | 42.8 GB | 1 TB | 123,642 / 850,000 |

The current `.venv` is about 7.6 GB in project space. The current uv cache is
about 8.0 GB at `/u/dli26/.cache/uv`, and the versioned Node installation is
about 413 MB. This still fits home comfortably, so no cache was moved during
bootstrap. Before large repeated model downloads, move future high-churn caches
to NVMe explicitly, for example:

```bash
mkdir -p \
  /work/nvme/bhvn/dli26/cache/uv \
  /work/nvme/bhvn/dli26/cache/huggingface \
  /work/nvme/bhvn/dli26/cache/torch

export UV_CACHE_DIR=/work/nvme/bhvn/dli26/cache/uv
export HF_HOME=/work/nvme/bhvn/dli26/cache/huggingface
export TORCH_HOME=/work/nvme/bhvn/dli26/cache/torch
```

Changing `UV_CACHE_DIR` does not move the existing cache; it only changes where
future uv commands look. Avoid duplicating the 8-GB cache accidentally. Use
`quota` regularly. Project and work have no snapshots, so Anvil or another
durable copy remains authoritative for anything expensive or impossible to
regenerate.
[Delta quota command and path examples](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/data_mgmt.html#quota-usage)

## 3. Repository, Python, Node, and packages

Clone or check the repository under project space:

```bash
cd /projects/bhvn/dli26/code/<project>
git rev-parse HEAD
git status --short
```

At snapshot time the branch was `master` at a recorded base commit, with
`origin` on GitHub over SSH. Record the base commit in the run manifest, and
commit or otherwise reconcile any bootstrap-time working-tree changes before
running parallel work from both sites.

The live module stack includes GCC 13.2 and `cudatoolkit/25.3_12.8`. The system
`python3` is only 3.9.18 and is too old for this project's intended environment.
No Node.js module is present. User-local installations are therefore used for
both uv/Python and Node.

### uv and Python

`uv 0.11.29` is installed at `/u/dli26/.local/bin/uv`. Its managed Python root
is `/u/dli26/.local/share/uv/python`, and CPython 3.12.13 is installed. Recreate
the tool installation with the pinned official installer if needed:

```bash
UV_INSTALLER=$(mktemp /tmp/uv-installer.XXXXXX.sh)
curl -LsSf https://astral.sh/uv/0.11.29/install.sh -o "$UV_INSTALLER"
UV_NO_MODIFY_PATH=1 sh "$UV_INSTALLER"
rm "$UV_INSTALLER"
rehash
uv python install 3.12
```

The complete locked environment was installed with:

```bash
cd /projects/bhvn/dli26/code/<project>
UV_LINK_MODE=copy \
  uv sync --python 3.12 --locked --all-extras --all-groups
```

`--locked` verifies that `uv.lock` still matches `pyproject.toml`; `--all-groups`
includes the `dev` group, and `--all-extras` future-proofs the command if extras
are added. `UV_LINK_MODE=copy` avoids cross-filesystem hard-link warnings
between home cache and project storage. The sync resolved 130 lock entries and
installed 123 packages. Validate it with:

```bash
uv lock --check
uv pip check
uv run --no-sync python -c \
  'import torch; print(torch.__version__, torch.version.cuda, torch.cuda.is_available())'
uv run --no-sync pytest --collect-only -q
```

Observed results are PyTorch `2.11.0+cu128`, CUDA build `12.8`, all installed
packages compatible, all major runtime imports successful, and 406 tests
collected. `torch.cuda.is_available()` is correctly false on the GPU-less login
node. The full test suite and a GPU-node CUDA assertion are still pending.

[Official uv installation](https://docs.astral.sh/uv/getting-started/installation/),
[official uv locking and syncing](https://docs.astral.sh/uv/concepts/projects/sync/)

### Node, npm, and npx

Delta does not currently expose a Node module. The official Node 24 LTS x64
binary was checksum-verified and installed at:

```text
/u/dli26/.local/node-v24.18.0-linux-x64
```

`node`, `npm`, and `npx` are symlinked into `/u/dli26/.local/bin`, which is
already in `PATH`. Current versions are Node `v24.18.0` and npm/npx `11.16.0`.
After an existing Zsh session has previously cached a command-not-found result,
run `rehash`:

```zsh
rehash
node --version
npm --version
npx --version
cd /projects/bhvn/dli26/code/<project>
npx skills@latest add mattpocock/skills
```

[Official Node release status](https://nodejs.org/en/about/previous-releases)

### GPU validation

GPU validation must run inside a Slurm GPU allocation:

```bash
uv run python - <<'PY'
import torch
print(torch.__version__, torch.version.cuda)
print(torch.cuda.is_available())
if torch.cuda.is_available():
    print(torch.cuda.get_device_name(0))
PY
```

The CUDA 12.8 import match is established, but device execution remains a
compatibility hypothesis until the actual CUDA/OpenCLIP/H5 smoke passes on a
Delta GPU. Delta also supports Apptainer and prebuilt NVIDIA NGC containers
under `/sw/external/NGC/`; use an Apptainer image as the fallback if native
compiled extensions fail.
[Delta container guide](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/containers.html)

### vLLM on Delta's CUDA 12.8 stack

**Yes, vLLM can run on this system, but qualify the exact wheel/runtime before
loading a large model.** Keep vLLM in a separate environment so its strict
PyTorch dependency cannot replace the repository's tested Torch 2.11/CUDA
12.8 environment:

```bash
export VLLM_ENV=/work/hdd/bhvn/dli26/envs/vllm
mkdir -p /work/hdd/bhvn/dli26/envs
uv venv "$VLLM_ENV" --python 3.12
```

The current vLLM installation guide says its default binary is CUDA 12.9 and
that release-specific CUDA 12.8 binaries may also be published. Do not assume
that every release carries every variant: inspect the assets for the exact
version and install a `cu128` wheel when it exists. In particular, the
official `v0.25.1` GitHub release assets observed on 2026-07-18 expose an
explicit `cu129` wheel but no explicit `cu128` asset. A bare `pip install
vllm` therefore must not be treated as a CUDA-12.8 pin.
[Current vLLM GPU installation matrix](https://docs.vllm.ai/en/latest/getting_started/installation/gpu/),
[vLLM v0.25.1 release](https://github.com/vllm-project/vllm/releases/tag/v0.25.1)

Driver `570.148.08` is inside NVIDIA's CUDA-12 minor-version compatibility
range (`>=525` and `<580`), so CUDA-12.9 cubins can generally run on it.
However, NVIDIA explicitly warns that PTX/JIT code can require a newer driver;
vLLM/Triton exercises JIT paths. Therefore a CUDA-12.9 vLLM wheel is a bounded
smoke candidate, not a compatibility guarantee.
[NVIDIA CUDA minor-version compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/minor-version-compatibility.html),
[CUDA 12.9 driver table](https://docs.nvidia.com/cuda/archive/12.9.0/cuda-toolkit-release-notes/index.html)

Use this order:

1. Prefer a version/model-supported `cu128` wheel if the exact release
   publishes one.
2. Otherwise test the pinned CUDA-12.9 wheel on a tiny model inside a one-hour
   Delta GPU allocation, including a real generated token rather than only an
   import.
3. If that fails in PTX/JIT or driver initialization, build the same pinned
   vLLM source revision against Delta's `cudatoolkit/25.3_12.8`; current vLLM
   requires GCC/G++ at least 11.3 and Delta supplies GCC 13.2.
4. Use Delta's Apptainer support plus the official vLLM image/compatibility
   libraries as a fallback, and qualify it with the same smoke.

Never install a CUDA-13 vLLM/PyTorch wheel on driver 570 as the first path:
CUDA 13 requires an R580-or-newer driver. Also never install vLLM into the
project's `.venv`; run the server from `$VLLM_ENV` and the project client from
the repository environment over `http://127.0.0.1:8000/v1`. For the CUDA
forward-compatibility workaround used when a wheel outruns the installed
driver, see [[CUDA-Compatibility-and-vLLM]].

## 4. Secrets and external services

Provision secrets independently on Delta; never copy them into Git, Slurm
scripts, logs, W&B config, or Globus-shared directories. A minimal arrangement
is:

```bash
mkdir -p "$HOME/.config/<project>"
chmod 700 "$HOME/.config/<project>"
$EDITOR "$HOME/.config/<project>/api_keys"
chmod 600 "$HOME/.config/<project>/api_keys"
source "$HOME/.config/<project>/api_keys"
```

The private file can export only the providers needed on Delta, such as
`HF_TOKEN`, `WANDB_API_KEY`, `GEMINI_API_KEY`, `OPENAI_API_KEY`,
`ANTHROPIC_API_KEY`, `DEEPSEEK_API_KEY`, and `ZAI_API_KEY`. Do not design the
Delta workflow around AnvilGPT; it is an Anvil service, not a Delta facility.

The official Delta documentation does **not** make a clear promise of arbitrary
outbound HTTPS from every compute partition. Before assigning API generation
or model downloads to Delta, test DNS/TLS and a tiny authenticated request from
the exact CPU/GPU compute partition, not only the login node. Never print the
key:

```bash
curl -fsS -o /dev/null -w 'github=%{http_code}\n' https://github.com/
curl -fsS -o /dev/null -w 'hf=%{http_code}\n' https://huggingface.co/
curl -fsS -o /dev/null -w 'pytorch=%{http_code}\n' https://download.pytorch.org/
```

If any required endpoint is blocked, pre-stage wheels, model snapshots, and
data through Globus, or ask NCSA support about that exact hostname. Successful
login-node access does not prove compute-node access. This remains an explicit
site-policy uncertainty.

Authenticate W&B and Hugging Face without exposing tokens in shell history:

```bash
source "$HOME/.config/<project>/api_keys"
printf '%s' "$WANDB_API_KEY" | uv run wandb login --relogin
uv run hf auth whoami
```

Use separate W&B run IDs/names and tags such as `system=delta`, `gpu=a100`, and
the Git commit hash. Never have Anvil and Delta resume the same W&B run.

## 5. Dataset bootstrap and transfer state

### Small public metadata: complete

The following public metadata is downloaded under the flat HDD root:

```text
/work/hdd/bhvn/dli26/datasets/
├── negclip
├── sugarcrepe
├── sugarcrepepp
└── vsr
```

The durable metadata occupies about 39 MB. Resumable sparse Git checkouts are
kept at `/work/nvme/bhvn/dli26/dataset-downloads` and occupy about 54 MB. The
repository's git-ignored `data/` symlinks point at the HDD dataset root rather
than at Anvil paths, one link per dataset directory.

Two portability requirements fall out of this and apply to any dataset
bootstrap script:

- it must accept a flag that skips Anvil-only shared-dataset checks and links
  (svib2 uses `--skip-shared-datasets`), because `/anvil/datasets/ai/...` does
  not exist on Delta; and
- every root it writes to must be overridable by environment variable
  (project, datasets, features, scratch, staging), so the same script produces
  the flat Delta layout without creating a nested work root.

Gated datasets, external-compositional corpora, and feature artifacts were not
staged on Delta at snapshot time. Full detail on svib2's downloader flags:
svib2 repo wiki, page Data-and-Caches.

### COCO and Visual Genome: public archive download

The owner chose a one-time login-node download of the official public archives
(an explicit exception to the "no production work on login nodes" rule, granted
because the job is a resumable public download rather than computation). A
resumable downloader script lives in the svib2 repo at
`scripts/download_coco_visual_genome_delta.sh`; the reusable requirements are:

- retain COCO train2017, val2017, and train/val annotations plus both Visual
  Genome image ZIPs;
- test every archive before extraction and overwrite partial extractions safely;
- validate 118,287 COCO train, 5,000 COCO validation, and 108,077 combined
  Visual Genome JPEGs, and create the `data/` links only after validation
  succeeds; and
- log to the dataset root and treat the bootstrap as incomplete until the
  script prints its terminal validation message.

Two site facts worth carrying forward: on Delta the COCO server's HTTPS
certificate did not match its hostname, so COCO's official HTTP archive URLs
were used; Visual Genome downloads over the Stanford HTTPS archive URLs
normally. Run under `tee` into a log on the dataset root and monitor with
`tail -F` from another shell.

The resulting raw layout is:

```text
/work/hdd/bhvn/dli26/datasets/coco/
├── train2017/
├── val2017/
├── annotations/
├── train2017.zip
├── val2017.zip
└── annotations_trainval2017.zip

/work/hdd/bhvn/dli26/datasets/visualgenome/
├── VG_100K/
├── VG_100K_2/
├── images.zip
└── images2.zip
```

Raw COCO directories satisfy code that reads image files by path. Code written
against Anvil's shared `coco_2017.lmdb` / `visualgenome.lmdb` stores does
**not** accept raw JPEG directories: transfer the LMDB stores with Globus,
build a qualified LMDB conversion, or add an explicit raw-directory adapter
before running those commands. Raw-image completion is not LMDB completion.

### Cross-site data movement

For Anvil-to-Delta transfers, use Globus. RCAC states that Anvil Globus can
access Anvil home, scratch, project, and shared dataset storage, while NCSA
exposes the **NCSA Delta** and **ACCESS Delta** collections including project
and work file systems.
[Anvil transfer guide](https://docs.rcac.purdue.edu/userguides/anvil/file_management/#globus),
[Delta Globus collections](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/data_mgmt.html#globus)

Transfer in this order (cheapest and most authoritative first):

1. accepted/human-verified record JSONL, frozen exclusions, and manifests;
2. one known-compatible feature cache (H5) for a reproduction smoke;
3. required LMDB stores or exact selected-image materializations;
4. filtered external subsets only for an assigned ablation;
5. large model weights and additional feature caches only after the smoke.

Keep a transfer manifest with source system/path, Git/data revision, byte and
file counts, and SHA-256 hashes for records/manifests/H5 files. Verify the
destination before deleting any staging copy. Do not expect Anvil absolute
symlinks under `/anvil/...` to work on Delta.

## 6. Slurm patterns for Delta

### Cost-aware interactive work on production GPUs

Prefer a production partition for routine interactive development when queue
wait is acceptable. Delta's named interactive partitions trade faster access
for a one-hour limit and exactly twice the corresponding production charge
factor:

| GPU node type | Production partition/factor | Named interactive partition/factor |
|---|---|---|
| Quad A100 | `gpuA100x4`, 1.0 | `gpuA100x4-interactive`, 2.0 |
| Octa A100 | `gpuA100x8`, 1.5 | `gpuA100x8-interactive`, 3.0 |
| Octa H200 | `gpuH200x8`, 3.0 | `gpuH200x8-interactive`, 6.0 |

Use `salloc` to reserve a production GPU for an interactive session. It is the
supported allocation mechanism; do not submit a dummy `sleep` or busy loop
merely to keep a batch job alive. Start it inside `tmux` on a concrete Delta
login host so a dropped client connection does not discard the controlling
shell:

```bash
tmux new -s delta-gpu
module reset

salloc \
  --account=bhvn-delta-gpu \
  --partition=gpuA100x4 \
  --nodes=1 \
  --ntasks=1 \
  --cpus-per-task=8 \
  --mem=32g \
  --gpus-per-node=1 \
  --time=04:00:00 \
  --constraint="projects&work" \
  --job-name=<project>-dev
```

Once Slurm grants the allocation, the `salloc` shell remains on the login node.
Run an interactive compute-node shell as a Slurm job step:

```bash
srun --pty bash -l

cd /projects/bhvn/dli26/code/<project>
nvidia-smi
uv run --locked python
```

Alternatively, Delta officially permits direct SSH from a login node to a
compute node assigned to a running job. From another login shell, find the node
and connect to it:

```bash
squeue -u "$USER" -o '%.18i %.12P %.8T %.20N'
ssh gpuaXXX
```

Prefer `srun` when practical because it establishes the Slurm job-step
environment explicitly. Direct SSH access and processes end when the allocation
ends. Keep the original `salloc` shell alive, and release resources immediately
when finished by exiting the compute shell and then the allocation shell:

```bash
exit  # leave the srun compute shell
exit  # relinquish the salloc allocation
```

For an H200 session, change the production partition to `gpuH200x8`; for an
octa-A100 session, use `gpuA100x8`. Request only the GPU count, CPU, memory, and
wall time actually needed. Delta charges the reserved resources even while the
session is idle, and a shorter realistic wall time improves the chance of
backfill. Use a named `*-interactive` partition only when fast start is worth
the two-times charge and the work fits within one hour.
[Delta interactive access, direct SSH, queue limits, and charge factors](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/running_jobs.html)
[Delta reserved-resource accounting](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/job_accounting.html)

### Production GPU batch

Production GPU partitions allow up to 48 hours. Prefer `sbatch` for training,
feature extraction, and bulk generation so a disconnected terminal does not
lose the work. Create the output directory before submission because Slurm
opens the log before executing the script:

```bash
mkdir -p /projects/bhvn/dli26/code/<project>/runs/delta-logs
```

```bash
#!/usr/bin/env bash
#SBATCH --account=bhvn-delta-gpu
#SBATCH --partition=gpuA100x4
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=64g
#SBATCH --gpus-per-node=1
#SBATCH --time=48:00:00
#SBATCH --job-name=<project>-delta
#SBATCH --output=/projects/bhvn/dli26/code/<project>/runs/delta-logs/%x-%j.out
#SBATCH --constraint="projects&work"
#SBATCH --requeue

set -euo pipefail
module reset
module list

export DELTA_PROJECT=/projects/bhvn/dli26
export DELTA_WORK_HDD=/work/hdd/bhvn/dli26
export DELTA_WORK_NVME=/work/nvme/bhvn/dli26
export PROJECT_REPO=/projects/bhvn/dli26/code/<project>
export HF_HOME=/work/nvme/bhvn/dli26/cache/huggingface
export TORCH_HOME=/work/nvme/bhvn/dli26/cache/torch
export WANDB_DIR=/work/hdd/bhvn/dli26/runs/wandb

mkdir -p "$HF_HOME" "$TORCH_HOME" "$WANDB_DIR"

cd "$PROJECT_REPO"
uv run --locked python -m <project>.cli.<pipeline_entry_point> ...
```

NCSA asks jobs to declare file-system dependencies. `projects` identifies
`/projects`, and `work` identifies `/work/hdd`.
[Delta file-system dependency labels](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/running_jobs.html#file-system-dependency-specification-for-jobs)
Delta does not automatically requeue/restart jobs unless `--requeue` is added,
and the script must be checkpoint/resume safe because requeue starts it from
the beginning. A resumable pipeline qualifies only when invoked with a fixed
configuration and a `--resume`-style flag after its run directory exists.
[Delta requeue policy](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/running_jobs.html#job-and-node-policies)

Use one GPU for compact model training. For a self-hosted 27B-class BF16 model,
one Delta 40-GB A100 is not expected to fit; first test one 141-GB H200, or a
quantized deployment, before requesting multi-GPU A100 tensor parallelism.
This is an engineering inference, not a Delta capacity guarantee. Delta
documents quad-A100, octa-A100, and octa-H200 node types and shared-node
scheduling.
[Delta architecture](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/architecture.html),
[Delta accounting](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/job_accounting.html)

### API-only CPU batch

Remote LLM APIs do not require a GPU. However, `accounts` did not list a Delta
CPU charge account for `dli26` at snapshot time, so this template is **blocked
until a CPU account is provisioned**. Do not charge API-only work to
`bhvn-delta-gpu` merely to bypass that missing account.

```bash
#!/usr/bin/env bash
#SBATCH --account=YOUR_DELTA_CPU_ACCOUNT
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32g
#SBATCH --time=48:00:00
#SBATCH --job-name=<project>-api
#SBATCH --constraint="projects&work"

set -euo pipefail
module reset
source "$HOME/.config/<project>/api_keys"
cd /projects/bhvn/dli26/code/<project>
uv run --locked python -m <project>.cli.<generation_entry_point> ...
```

Only use this after the exact API endpoint succeeds from a `cpu` compute node.

## 7. Safe parallel work across Anvil and Delta

The two sites should share Git history and immutable manifests, not a live
working directory.

### Code ownership

- Keep `master` clean on both sites.
- Create task/system branches such as `delta/<task>` or `anvil/<task>`; do not
  edit the same files independently on both systems.
- Fetch and rebase/merge deliberately before pushing. Never move uncommitted
  work by copying the repository directory.
- Record `git rev-parse HEAD`, `sha256sum uv.lock`, `module list`, and
  `nvidia-smi` in every run manifest/log.
- Do not copy `.venv`; each system rebuilds from the same lock file.

### Data and run ownership

- Assign non-overlapping shard ranges before launch. A row/shard has exactly
  one owning system.
- Use distinct roots, for example `runs/delta/<run-id>` and
  `runs/anvil/<run-id>`, and include `execution_system` in manifests.
- Never let both sites resume the same run directory, W&B run ID, teacher
  shard, or checkpoint.
- Merge only completed shards whose configuration hash, prompt version,
  validator version, source split, and schema version match (the provenance
  fields listed in [[Data-and-Caches]]).
- Transfer completed immutable outputs plus checksums; do not synchronize
  partially written JSONL/H5/checkpoints.
- Keep frozen benchmark exclusions and no-leakage filters identical on both
  systems. Evaluation test sets remain untouched regardless of system.

### Suggested division of labor

| Anvil | Delta |
|---|---|
| authoritative existing verified corpora, generated artifacts, and RCAC-resident COCO/VG stores | independent ablations on fixed transferred bundles |
| AnvilGPT-hosted generation and validated existing corpora | CPU API experiments only after a compute-node network smoke |
| dataset mining that depends on shared RCAC LMDB/S3 | H200 memory-fit tests or overflow A100 training |
| authoritative long-lived copy of generated artifacts | disposable work-space mirrors and site-specific caches |

GitHub is the integration point for source and wiki history. Before parallel
execution begins, finish the current Delta bootstrap edits, commit them on an
intentional branch, and make both systems start their assigned work from an
explicit common commit.

## 8. Bring-up acceptance checklist

Checked items were observed directly on 2026-07-18 during the svib2 bootstrap.
Reuse the list for any project; do not call Delta production-ready for a
project until the remaining items pass for it:

- [x] NCSA password plus NCSA Duo login works for `dli26`.
- [x] Allocation `bhvn` and GPU charge account `bhvn-delta-gpu` are active.
- [ ] A CPU charge account is provisioned before API-only CPU work.
- [x] Live quotas and explicit project/HDD/NVMe paths are recorded.
- [x] The repository is present at an intentional base commit.
- [ ] Bootstrap edits are committed and the worktree is clean before parallel
  Anvil/Delta execution.
- [x] `uv sync --python 3.12 --locked --all-extras --all-groups` completes.
- [x] `uv pip check`, major imports, and collection of 406 tests pass.
- [ ] Full `uv run --locked pytest` passes in an approved CPU context.
- [x] Small public benchmark metadata is staged and linked.
- [x] COCO train2017, val2017, and annotations are downloaded and extracted.
- [ ] Visual Genome parts 1 and 2 finish, validate, extract, and link.
- [ ] A one-GPU Slurm smoke reports the expected device and executes a CUDA op.
- [ ] One encoder/text-embedding smoke and one feature-cache (H5) read complete
  on a GPU node.
- [ ] One known training/eval run reproduces within an agreed tolerance.
- [ ] A verified bundle/feature H5 or required LMDB is transferred with hashes.
- [ ] API/Hugging Face/W&B access is tested from the exact compute partition,
  with secrets absent from logs.
- [ ] Delta and Anvil have disjoint branches, W&B runs, run directories, and
  assigned shard ranges.

## Remaining uncertainties and blockers

1. **Compute-node outbound HTTPS:** login-node GitHub, Node, uv, PyPI, PyTorch,
   COCO, and Stanford downloads work. This does not establish egress from CPU
   or GPU compute partitions; test required endpoints from the exact partition.
2. **CPU charging:** only `bhvn-delta-gpu` appears in `accounts`. API-only CPU
   jobs remain blocked pending a CPU account.
3. **Full CPU test result:** dependency checks and collection pass, but the full
   406-test run has not yet been executed.
4. **GPU runtime:** the CUDA 12.8 wheels import on the login node, but there is
   no device there. CUDA execution, encoder inference, feature-cache reads, and
   run reproduction remain unqualified.
5. **Visual Genome completion:** the public archive download was still active
   when this snapshot was written. Require the script's terminal validation
   message before marking it complete.
6. **Raw images versus LMDB:** workflows written against the Anvil LMDB schemas
   need an adapter, a qualified conversion, or a Globus transfer of the source
   stores. Raw JPEG directories do not satisfy them.
7. **Provider replacement:** Delta has no AnvilGPT facility. Preserve
   generation provenance when using transferred outputs, self-hosted models, or
   external providers.
8. **Queue state:** documented limits can change. Use `sinfo`, `scontrol show
   partition`, and `accounts` before every large reservation.

## Primary sources

- [NCSA Delta current documentation](https://docs.ncsa.illinois.edu/systems/delta/en/latest/)
- [Delta login methods](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/login.html)
- [Delta accounts and allocation policies](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/accounts.html)
- [Delta storage and data movement](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/data_mgmt.html)
- [Delta jobs, queues, interactive sessions, and batch policy](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/running_jobs.html)
- [Delta accounting](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/job_accounting.html)
- [Delta software and Python](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/software.html)
- [Delta Red Hat 9/CUDA 12.8 environment](https://docs.ncsa.illinois.edu/systems/delta/en/latest/whats_new.html)
- [Delta Apptainer containers](https://docs.ncsa.illinois.edu/systems/delta/en/latest/user_guide/containers.html)
- [ACCESS allocation exchange instructions](https://allocations.access-ci.org/how-to)
- [ACCESS Delta GPU onboarding](https://support.access-ci.org/documentation/resources/delta-gpu)
- [RCAC Anvil file transfer/Globus](https://docs.rcac.purdue.edu/userguides/anvil/file_management/)
- [Official uv installation](https://docs.astral.sh/uv/getting-started/installation/)
- [Official uv locking and syncing](https://docs.astral.sh/uv/concepts/projects/sync/)
- [Official Node release status](https://nodejs.org/en/about/previous-releases)
- [Current vLLM GPU installation](https://docs.vllm.ai/en/latest/getting_started/installation/gpu/)
- [NVIDIA CUDA minor-version compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/minor-version-compatibility.html)

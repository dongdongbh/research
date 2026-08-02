# Data And Caches

Updated 2026-08-02 for the general research wiki.

Purpose: where the lab's datasets, feature caches, generated corpora, and run
artifacts live on Anvil, and the provenance rules that apply to all of them.
Project-specific corpora, symlink sets, and CLI entry points stay in each
project's own repo wiki.

## Storage layout on Anvil

| Role | Path | Rule |
|---|---|---|
| Shared RCAC AI datasets | `/anvil/datasets/ai/` (e.g. `coco`, `visualgenome`) | Read in place. Keep them in their RCAC archive/LMDB/SquashFS forms; never duplicate them into the project allocation. |
| Lab dataset root | `/anvil/projects/x-cis261253/datasets` | Durable downloaded/derived datasets. Currently holds `negclip`, `sugarcrepe`, `sugarcrepepp`, `vsr`, `winoground`, `external_compositional`. |
| Durable artifacts | `/anvil/projects/x-cis261253/artifacts` | Feature caches (`svib_features`) and the shared model cache (`hf-cache`). |
| Code | `/anvil/projects/x-cis261253/code/<project>` | One directory per project repo. |
| Scratch | `$SCRATCH` = `/anvil/scratch/x-dli26` (Anvil account: x-dli26) | Temporary download staging, materialized images, job intermediates only. No backup, 30-day purge — nothing irreplaceable. |

Project space has daily snapshots (5 TB quota); scratch has none (100 TB).
Anything expensive to regenerate belongs in project space with a manifest.

## Repo `data/` symlinks

Each project repo keeps a git-ignored `data/` directory of machine-local
symlinks into the paths above (for example `coco -> /anvil/datasets/ai/coco`,
`<name> -> /anvil/projects/x-cis261253/datasets/<name>`, and a features link
into `artifacts/`). These links are workspace setup, not versioned artifacts,
and Anvil-absolute symlinks do not survive a move to another cluster — see
[[Delta-Setup-and-Parallel-Workflow]] for the second-site equivalents.

svib2's exact symlink set, its `scripts/download_datasets.sh` bootstrap flags
(`--with-winoground`, `--with-external-compositional`, `--with-external-filtered`),
and its filtered VisMin/COCO-Counterfactuals/TripletCLIP corpora are documented
in the svib2 repo wiki, pages Data-and-Caches and
External-Compositional-Datasets.

## Provenance rules

These apply to every project that writes data under the paths above.

- **Manifest every artifact.** Write a manifest next to each output recording
  input paths, output paths, config values, row counts, and SHA-256 hashes.
  An artifact without a manifest cannot be used as evidence.
- **Shard completion is hash-bearing.** A resumable shard counts as complete
  only when its `complete.json` (or equivalent) exists and its hash matches.
- **Encoder provenance is the model pair, not the dimension.** A feature cache
  must record the exact encoder model and pretrained tag; matching vector
  dimension is not sufficient provenance, and text/image encoders that must
  match should be the identical model/pretrained pair.
- **Generated (teacher/judge) records keep the full exchange.** Store model
  name, prompt/schema version, temperature, top-p, raw output, parsed output,
  and the rejection reason when applicable. Keep generated JSONL small enough
  to inspect before it feeds training.
- **Split before caching, and group by image.** Split records into
  train/eval/filtered with a fixed seed and group by `image_id` so multiple
  rows from one image cannot straddle the split. The split manifest records
  total/kept/filtered counts and train/eval group counts.
- **Frozen exclusions travel with the data.** Benchmark-overlap exclusions and
  no-leakage filters must be identical everywhere the corpus is used, including
  on a second cluster. Evaluation test sets stay untouched.
- **Contamination is checked, not assumed.** Known example: 4,256 unique NegCLIP
  images overlap COCO val2017, so NegCLIP metadata contaminates
  SugarCrepe/SCPP++-style evaluation and must not be substituted for a
  leakage-safe source.
- **Adopt external datasets fail-closed.** A new external corpus is adopted only
  after a metadata-only inspection pass and a pre-registered acceptance
  criterion; a pilot shard that fails the criterion blocks the full transfer.
- **Do not commit large binaries.** Multi-GB H5/`.pt`/image archives stay under
  the project allocation or scratch, never in a Git repo.
- **Disposable copies stay disposable.** Materialized image sets rebuildable
  from an LMDB or archive live under `$SCRATCH/<project>/...` and are not
  promoted to durable project storage.

## Cache and run-directory conventions

- Generated data artifacts (requests, outputs, rejections) live outside Git,
  under ignored cache/`runs/` directories, one directory per run id.
- A pipeline run directory holds the split, artifacts, cache, training outputs,
  and a top-level `pipeline_manifest.json`; re-running with `--resume` reuses
  existing stages only when their files and manifests are both present.
- Training caches should be written with a failures file and a manifest so
  skipped rows and cache provenance sit next to the cached tensors. Cache rows
  should carry lightweight source metadata (record id, image id/path, captions,
  source, pair type, label, parse confidence) — tensor-only caches still train
  but cannot be exported back to per-caption scores.
- Run a preflight check against the run-directory contract (manifests, failure
  files, metrics, minimum train/eval sizes) before scaling a recipe or
  committing GPU/API budget to a larger run.
- Feature H5 stores are keyed by image id with per-node features, node masks,
  and boxes; lookups generally assume COCO-style numeric image ids.

Project-specific record schemas, artifact families, and CLI invocations
(`<project>.cli.*`) are documented in each project's own repo wiki. For svib2,
see its Data-and-Caches page.

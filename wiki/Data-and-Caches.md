# Data And Caches

Purpose: where SVIB2's datasets, feature caches, generated corpora, and run
artifacts live on Anvil, and the provenance rules for each.

The current Anvil dataset root is
`/anvil/projects/x-cis261253/datasets`. Durable feature artifacts live under
`/anvil/projects/x-cis261253/artifacts`; scratch is only for temporary download
staging and job intermediates.

Machine-local symlinks in the ignored repo `data/` directory:

```text
coco          -> /anvil/datasets/ai/coco
scpp          -> /anvil/projects/x-cis261253/datasets/sugarcrepepp
sugarcrepe    -> /anvil/projects/x-cis261253/datasets/sugarcrepe
sugarcrepepp  -> /anvil/projects/x-cis261253/datasets/sugarcrepepp
negclip       -> /anvil/projects/x-cis261253/datasets/negclip
vsr           -> /anvil/projects/x-cis261253/datasets/vsr
winoground    -> /anvil/projects/x-cis261253/datasets/winoground
VisualGenome  -> /anvil/datasets/ai/visualgenome
svib_features -> /anvil/projects/x-cis261253/artifacts/svib_features
external_compositional -> /anvil/projects/x-cis261253/datasets/external_compositional
```

These links are ignored by Git. They are machine-local workspace setup, not
versioned project artifacts.

Bootstrap SugarCrepe, SugarCrepe++, VSR, and the official NegCLIP TSV metadata
with:

```bash
bash scripts/download_datasets.sh
```

Add `--with-winoground` after accepting its Hugging Face access terms and
exporting `HF_TOKEN`. The command is resumable, uses sparse Git checkouts under
`$SCRATCH`, validates Anvil's shared COCO and Visual Genome stores, and creates
the ignored repo symlinks. Shared COCO and Visual Genome remain in their RCAC
archive/LMDB/SquashFS forms; do not duplicate them in the project allocation.

Add `--with-external-compositional` to download the metadata-only inspection
snapshot for VisMin, VisMin-bench, COCO-Counterfactuals, and TripletCLIP-HQ.
It shows progress, pins upstream revisions, and does not download image archives
or gated TripletCLIP shards. The current snapshot is 29 MB at:

```text
data/external_compositional
```

Run the frozen benchmark-overlap audit with:

```bash
uv run python -m svib2.cli.audit_external_contamination \
  --vismin-csv \
    data/external_compositional/vismin/historical_metadata/train.csv \
  --coco-counterfactuals-jsonl \
    data/external_compositional/coco_counterfactuals/metadata/data/examples.jsonl \
  --output data/external_compositional/contamination_report.json
```

See [[External-Compositional-Datasets]] for the measured exclusion counts,
source limitations, and fail-closed adoption policy.

Add `--with-external-filtered` to stream the full upstream VisMin/COCO-
Counterfactual releases through scratch while retaining only rows that pass
both frozen image-ID and exact-caption exclusions:

```bash
source /home/x-dli26/.config/zsh/api_keys
bash scripts/download_datasets.sh --with-external-filtered
```

This option implies the metadata download, shows per-shard progress, resumes
completed output shards, and removes the unfiltered transfer files. The current
durable artifacts are:

```text
data/external_compositional/vismin/filtered_current       # 69,286 rows; 22 GB
data/external_compositional/coco_counterfactuals/filtered # 11,194 rows; 3.9 GB
```

VisMin is stored as 75 independently loadable Hugging Face Dataset shards.
COCO-Counterfactuals is stored as 23 ZIP image shards plus `records.jsonl`.
Adjacent manifests pin upstream revisions, filter counts, source/archive
hashes, and retained output hashes.

TripletCLIP-HQ is intentionally limited to the inspected `shard-0.tar` pilot
(1,860 samples, 848 MiB on disk). Its 670 GB full transfer is a no-go because
90.27% of the pilot violates the strict minimal-edit proxy and source-image
provenance is unavailable. Do not expand it unless the pre-registered criteria
in [[External-Compositional-Datasets]] are met.

Useful SVIB feature cache link:

```text
data/svib_features
```

SVIB2 should reuse submitted SVIB feature caches where possible, but new SVIB2
evidence tables should keep a separate schema:

```text
record -> spans/claims -> grounded candidates -> claim-required bindings
```

Do not copy multi-GB H5 files into this repository.

## Verified MVP Cache (2026-07-18)

The first verified-data MVP corpus is kept in ignored run storage:

```text
runs/mvp_verified_20260718/human_gold560_plus_qwen68_records628.jsonl
```

It contains 628 records over 620 distinct images: 283 human-verified VG role
rows, 277 human-verified VG attribute rows, and 68 current-validator AnvilGPT
Qwen3.6 COCO rows. Record SHA-256:
`a6caf8e6aa9d92e6a515f4a2ee6e4b384f4fb9e9f29c3a29bd27aef0673c3fec`.
The adjacent manifest preserves every input path, row count, and hash.

Exact input images are materialized temporarily under
`$SCRATCH/svib2/mvp_verified_20260718/images`; they are rebuildable from
`data/VisualGenome/visualgenome.lmdb` and `data/coco/coco_2017.lmdb` using the
checked-in materializer. Do not promote this 91.7 MB copy to durable project
storage.

The durable selected-image node cache is:

```text
data/svib_features/mvp_verified_20260718_sam3_openclip_vitb32.h5
```

It contains all and only the 620 corpus image IDs, 512-dim normalized OpenCLIP
ViT-B-32 `laion2b_s34b_b79k` features, one global node plus at most 20 SAM3
local nodes, boxes/scores/masks, and a completed processed mask. QC found 2--21
nodes/image (mean 11.329), zero skips/fallback-only images, finite values, and
unit feature norms. H5 SHA-256:
`56a10e0bbd65735f1d25f1578ea9e7ee642686b56a473821449814d3abe5985d`.
The visual encoder and artifact text encoder are deliberately the exact same
OpenCLIP model/pretrained pair; matching dimension alone is not sufficient
provenance.

## Benchmark Sampling

SugarCrepe and SCPP++ JSON files under `data/sugarcrepe` and
`data/sugarcrepepp` can be sampled into SVIB2 records with:

```bash
uv run python -m svib2.cli.sample_benchmark_pairs sugarcrepe \
  --input data/sugarcrepepp/swap_obj.json \
  --output scpp_swap_obj_records.jsonl \
  --source sugarcrepepp \
  --pair-type swap_obj \
  --limit 100
```

The sampler preserves benchmark captions and uses the current deterministic
parser. Some rows will intentionally have empty claims and `parse_confidence=0`
until the parser/evidence path becomes broader or teacher parses are merged.

For mixed SCPP++ coverage, prefer the directory sampler:

```bash
uv run python -m svib2.cli.sample_benchmark_pairs sugarcrepe-dir \
  --input-dir data/sugarcrepepp \
  --output runs/scpp-mixed/records.jsonl \
  --source sugarcrepepp \
  --limit-per-file 100 \
  --manifest-output runs/scpp-mixed/sample_manifest.json
```

The manifest records the source, inferred pair types, per-pair-type counts, and
limit used for each file. This makes later training and evaluation runs
auditable when several benchmark pair types are mixed into one record file.

## Offline Teacher Outputs

Superseded note (2026-07-19): Gemini is not a production teacher lane. Per
[[Current-Standard]], production generation uses the hosted AnvilGPT
Gemma 4 -> Qwen 3.6 cascade plus local generation restricted to
attribute/spatial targets; Gemini remains a bounded calibration/audit
reference. The storage and provenance conventions below apply to every teacher
lane.

Teacher outputs (Gemini/Qwen alike) should be stored as generated data
artifacts outside Git, for example under ignored cache or results directories.
Keep the generated JSONL small enough to inspect before using it for training.

Recommended teacher artifact families:

```text
teacher_parse_requests.jsonl      # Gemini Batch API input, keyed requests
teacher_parse_outputs.jsonl       # parsed Gemini Batch output
teacher_pair_requests.jsonl       # Gemini Batch API input, keyed requests
teacher_pair_outputs.jsonl        # generated positive/negative pairs
teacher_rejections.jsonl          # validation failures and rejection reasons
```

The leakage-safe production source is
`runs/anvilgpt_qwen36_coco_train2017_full_20260717/captions.jsonl`: 118,287
rows, exactly one per distinct COCO train2017 image. Generated artifacts live
under `runs/anvilgpt_qwen36_coco_train2017_v2_20260717/shards/`; each shard is
independently resumable and complete only when its hash-bearing
`complete.json` exists. Do not substitute NegCLIP metadata: 4,256 of its unique
images overlap COCO val2017 and therefore contaminate SCPP++/SugarCrepe-style
evaluation.

Each validated teacher record should preserve model name, prompt/schema
version, temperature, top-p, raw output, parsed output, and rejection reason when
applicable.

## Verifier Training Cache Inputs

Records should be split before cache building:

```bash
uv run python -m svib2.cli.split_records \
  --input counterfactual_records.jsonl \
  --train-output train_records.jsonl \
  --eval-output eval_records.jsonl \
  --filtered-output filtered_records.jsonl \
  --manifest split_manifest.json \
  --eval-fraction 0.2 \
  --seed 13 \
  --group-by image_id \
  --require-claims
```

The manifest records total, kept, filtered, train/eval record counts, and
train/eval group counts. Use image-level grouping for benchmark work to avoid
leaking multiple rows from the same image across train and eval.

For full runs, prefer `svib2.cli.run_verifier_pipeline`. It writes the split,
artifact, cache, and training outputs under one run directory and records a
top-level `pipeline_manifest.json`. Re-run with `--resume` to reuse existing
split, artifact, cache, and training outputs when their files and manifests are
present.

`svib2.cli.build_verifier_cache` joins accepted `CaptionPairRecord` JSONL with
four artifact families and writes `cached_train_pairs.pt` or
`cached_eval_pairs.pt`. Use `--failures-output` and `--manifest-output` for
training caches so skipped rows and cache provenance are stored next to the
`.pt` file. The manifest records input paths, output paths, config values, and
written/skipped/total counts. New cache rows also store lightweight source
metadata: record id, image id/path, positive and negative captions, source,
pair type, label, and parse confidence. Older tensor-only caches still load for
training, but caption-score export requires metadata-bearing caches.

`svib2.cli.generate_verifier_artifacts` can produce these artifact files from
records plus SVIB-style node feature H5 files. Its hash backend is only for
smoke tests. Its OpenCLIP backend is the first real text-embedding path, using
node-feature means for global image vectors and side-specific span-to-node
candidate tables for evidence. Text embeddings are batched with
`--text-batch-size`; use a larger value for full-scale GPU generation. Passing
`--open-clip-pretrained none` disables weight loading for smoke tests only.
Use `--manifest-output artifact_manifest.json` on artifact-generation runs so
backend, model, pretrained tag, input/output paths, and row counts are captured
next to the artifacts.

Global image/caption vectors:

```json
{"image_id":"10","kind":"image","vector":[0.0,1.0]}
{"image_id":"10","caption":"a dog chasing a cat","vector":[1.0,0.0,0.0]}
```

Claim embeddings are keyed by the stable record artifact key emitted by
`svib2.build_verifier_cache.record_artifact_key(record)`:

```json
{"record_id":"...","side":"pos","embeddings":[[0.1,0.2,0.3]]}
{"record_id":"...","side":"neg","embeddings":[[0.3,0.2,0.1]]}
```

Evidence tables are keyed by image id and span id:

```json
{"image_id":"10","image_width":100,"image_height":100,"candidates_by_span":{"s1":[{"span_text":"dog","box":[0,0,10,10],"score":0.9,"candidate_id":"0"}]}}
```

Side-specific evidence rows avoid mixing repeated span ids between positive and
negative captions:

```json
{"image_id":"10","side":"pos","image_width":100,"image_height":100,"candidates_by_span":{"s1":[{"span_text":"dog","box":[0,0,10,10],"score":0.9,"candidate_id":"0"}]}}
{"image_id":"10","side":"neg","image_width":100,"image_height":100,"candidates_by_span":{"s1":[{"span_text":"cat","box":[20,0,30,10],"score":0.8,"candidate_id":"1"}]}}
```

Node features come from an SVIB-compatible H5 store with image ids, per-node
features, node masks, and boxes. Current lookup assumes COCO-style numeric
image ids for these H5 features.

## Verifier Score Outputs

`svib2.cli.score_verifier_cache` reads a trained checkpoint plus a
metadata-bearing verifier cache and writes two useful artifacts:

```text
eval_pair_scores.jsonl       # one row per pair, with score_pos, score_neg, delta
eval_caption_scores.jsonl    # two image_id/caption/score rows per pair
```

`eval_caption_scores.jsonl` is intentionally compatible with cached global
score rows, so it can be passed to `svib2.cli.evaluate_pairs` as
`--verifier-scores`. Use `--constant-global-score 0.0` with
`svib2.cli.evaluate_pairs` for verifier-only evaluation before external
VLM2Vec/SigLIP2 score files are available.

## Run Preflight

`svib2.cli.preflight_run` validates the expected run directory contract before
a recipe is scaled. It reads `pipeline_manifest.json`, cache manifests, failure
JSONL files, training metrics, verifier score exports, and evaluation metrics.
Use it to enforce skip gates and minimum train/eval sizes before committing GPU
or teacher-generation budget to a larger run.

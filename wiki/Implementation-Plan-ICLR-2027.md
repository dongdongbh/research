# Work Plan for Two ICLR 2027 Projects

This plan was written on 2026-08-02. It covers
[[Prereg-RoboJudge-Audit]] and [[Prereg-Crop-Consistency-Distillation]]. It
explains which cluster will run each task, which data and models we need, and
what must happen before the September 18 abstract deadline. A check mark with
the date means we verified the fact directly on 2026-08-02.

## Problems already checked (✓ 2026-08-02, Hugging Face API)

- **The RoboArena download is only 18.5 GB.**
  [`RoboArena/DataDump_02-03-2026`](https://huggingface.co/datasets/RoboArena/DataDump_02-03-2026)
  has `usedStorage` set to `18,473,184,727` bytes, has an MIT license, and
  contains 23,600 metadata rows in its Parquet export. The older
  [`DataDump_08-05-2025`](https://huggingface.co/datasets/RoboArena/DataDump_08-05-2025)
  is 8.1 GB. Storage is no longer a go/no-go risk. The whole set easily fits
  on `$SCRATCH`, and labels could fit in project space. Week one still needs
  to download the files and parse one complete session.
- **RoboReward-4B and RoboReward-8B are available.** The model names are
  `teetone/RoboReward-*`, and the license is CC-BY-4.0. Both are Qwen3-VL
  fine-tunes, so they need a recent Transformers or vLLM version that supports
  `qwen3_vl`. Our Anvil H100 vLLM setup does. See
  [[Anvil-H100-Qwen36-vLLM-Benchmark]] and
  [[CUDA-Compatibility-and-vLLM]].
- **Cosmos-Reason is available.**
  [`nvidia/Cosmos-Reason1-7B`](https://huggingface.co/nvidia/Cosmos-Reason1-7B)
  uses Qwen2.5-VL and plain Transformers. The newer
  [`Cosmos-Reason2-2B/8B`](https://huggingface.co/nvidia/Cosmos-Reason2-8B)
  models, released in December 2025, use Qwen3-VL. Their license is listed as
  “other” by NVIDIA. We can evaluate them in a paper, but must save the exact
  license in the run record. Use Reason2-8B first and Reason1-7B when we need
  a result comparable with older work.
- **CLIPSelf checkpoints exist.** They are available at
  [`wusize/clipself`](https://huggingface.co/wusize/clipself) and in the GitHub
  releases. They have no stated license. We may evaluate them but must not
  copy and republish them.

## Important corrections made later on 2026-08-02

These corrections replace the older Anvil A100 choices.

- **Our Anvil account, `cis261253-ai`, can use only the `ai` partition.** An
  A100 `gpu` job submitted with `sbatch` is rejected with “Please use
  --partition=ai.” All work on
  Anvil must therefore use H100s in `ai`, with 24 CPUs and 250 GB of memory
  for each GPU. A free `sbatch --test-only` check found 97 waiting jobs and an
  estimated delay of about seven days. Book the 72B-judge and DreamGen windows
  now. If those bookings do not work, use Delta's 8×H200 node as the main
  alternative, not as a last-minute rescue.
- **Use the lab's exact SigLIP2 model:**
  [`ViT-B-16-SigLIP2-256`](https://huggingface.co/timm/ViT-B-16-SigLIP2-256)
  trained on WebLI and loaded through OpenCLIP. All SVIB settings, cache names,
  and rebuttal records use this model. The `google/siglip2-base-*` Hugging Face
  tags, including the old `google/siglip2-base-patch16-*` plan, do not load
  with our current feature extractor. Do not use the staged 224-pixel Hugging
  Face checkpoint.
- **Build the SigLIP2 teacher cache on OrangeGrid.** The extraction code is in
  `svib`, and crop-distillation reuses it. OrangeGrid already has the required
  template H5 files: Visual Genome with 108,073 rows, COCO with 5,000, and
  Winoground with 800. It also has 193 GB of existing caches and the image
  files. Anvil has none of these. The grid has 20 local views plus 1 global
  view, and uses aligned 1×1 `roi_align`.
  The older plan expected to read shared images from `/anvil/datasets/ai`, but
  that does not provide the complete local extraction setup listed here.
- Small RoboJudge sweeps remain on OrangeGrid. The old “Anvil A100 because the
  data is nearby” option is invalid.

## Where each task should run

| Task | Cluster | Reason |
|---|---|---|
| Download RoboArena, extract frames, fit Bradley–Terry rankings, and make the H3 figure | **Anvil CPU and `$SCRATCH`** | The download is 18.5 GB. Ranking needs only CPUs, and Anvil is the data home. |
| Run 2–8B judges: RoboReward-4B/8B, Qwen2.5-VL-7B, InternVL3-8B, Cosmos-Reason, and the SigLIP2 scorer. Also run hidden-factor battery A3 and degenerate-set A4. | **OrangeGrid** with 2×A100 or L40S | It is free and has no short wall-time limit. After frame extraction, only a few GB must move. The old Anvil A100 overflow plan is invalid. |
| Run the Qwen2.5-VL-**72B** judge | **Anvil AI 4×H100 with vLLM**; use **Delta 8×H200** if the queue blocks the booked window | Four H100s provide 320 GB of GPU memory, and this software stack is tested. Score all items in one or two reserved windows. |
| Regenerate DreamGen videos for A5, about 60–120 GPU-h | **Anvil AI H100**; move to Delta if booking fails | Video diffusion benefits from H100s. Save checkpoints between jobs because jobs have time limits. |
| Run crop-distillation A4/A5 checks, train adapters for 6 settings × 3 seeds, evaluate the full test set, and measure H6 speed | **OrangeGrid** | The SVIB evaluation code and L40S timing method already live there. Training from saved features needs at most two GPUs and is free. |
| Build the SigLIP2-B/16 teacher cache from COCO and Visual Genome crop grids | **OrangeGrid** | The code, images, exact template H5 files, and existing caches already live there. This replaces the old Anvil A100 plan. |

Use OrangeGrid for everything that fits on two GPUs. Spend ACCESS credits only
on jobs that truly need the large Anvil or Delta GPUs: the 72B judge and
DreamGen. The older estimate was 100–200 H100-hours plus 20–40 A100-hours for
all three large tasks, including the teacher-cache build. After moving that
cache build to free OrangeGrid, the paid need should be no larger than that
estimate.

## Data and model files to prepare

Stage Hugging Face model files under `artifacts/hf-cache` unless a step below
names a different location.

### RoboJudge

1. Download
   [`RoboArena/DataDump_02-03-2026`](https://huggingface.co/datasets/RoboArena/DataDump_02-03-2026)
   to `/anvil/projects/x-cis261253/datasets/roboarena/dump/`. Save a manifest
   and a SHA-256 checksum as required by [[Data-and-Caches]]. On 2026-08-02,
   the owner chose project space for the whole 18.5 GB download because 4.6 TB
   is free there. This avoids `$SCRATCH`'s 30-day deletion rule and removes the
   need for a second label copy. Set `HF_HUB_DISABLE_XET=1`; the many small
   files otherwise cause HTTP 429 rate-limit errors at the Xet token service.
2. Download these models:
   [`teetone/RoboReward-4B`](https://huggingface.co/teetone/RoboReward-4B),
   [`teetone/RoboReward-8B`](https://huggingface.co/teetone/RoboReward-8B),
   [`Qwen/Qwen2.5-VL-7B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct),
   [`Qwen/Qwen2.5-VL-72B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-VL-72B-Instruct),
   its AWQ version for a 2×A100 fallback,
   [`OpenGVLab/InternVL3-8B`](https://huggingface.co/OpenGVLab/InternVL3-8B),
   [`nvidia/Cosmos-Reason2-8B`](https://huggingface.co/nvidia/Cosmos-Reason2-8B)
   with its license note, and the shared SigLIP2 model below.
3. For [DreamGen](https://arxiv.org/abs/2505.12705), get the four published
   video models, benchmark prompts, and the paper's published vector of policy
   success results. Check the exact model files when A5 starts, not now.

### Crop consistency

1. Use the OpenCLIP ViT-B/32 model with the correct QuickGELU version. A GELU
   mismatch silently makes source-matched comparisons wrong. The second
   backbone is the OpenCLIP
   `ViT-B-16-SigLIP2-256`/WebLI model named in the correction above. Record the
   exact resolution and model pair in the cache manifest.
2. Download the [`wusize/clipself`](https://huggingface.co/wusize/clipself)
   checkpoint from Hugging Face or the
   [GitHub release](https://github.com/wusize/CLIPSelf). This is the deciding
   A4 check in week one.
3. Prepare the A5 comparison methods from the wave-2 check:
   [LABCLIP](https://arxiv.org/abs/2502.03566), a text-side matrix with about
   590,000 parameters that we can rebuild in one day if code is unavailable;
   [DCSM, ICCV 2025](https://arxiv.org/abs/2503.08723), whose code release must
   be checked; and [DeGLA](https://arxiv.org/abs/2504.16801), whose checkpoint
   must be checked. If DeGLA is unavailable, use its published numbers as the
   pre-registration says.
4. Reuse `artifacts/svib_features`, including CLIP crop-grid teacher features
   and the trained grid-plus-self-attention head. This is our multi-region
   teacher. Reuse the datasets already in the lab root: `sugarcrepe`,
   `sugarcrepepp`, `winoground`, `vsr`, `external_compositional`, and
   `negclip`. Keep the NegCLIP/COCO validation-contamination rule. Read COCO
   and Visual Genome in place.
5. Build one new H5 cache for the SigLIP2-B/16 model. Save patch tokens and the
   20 local plus 1 global view for the same images. The estimate is about 2.5
   million crop encodes, or single-digit A100-hours. Key every row by image ID
   and save a manifest.

## Work order for both projects

### Week 1: lock the plans, August 4–10

- **RoboJudge:** download and open the data; parse one full session; freeze the
  Bradley–Terry human ranking and its bootstrap confidence intervals; and make
  the H3 sample-size sensitivity figure. All of this is CPU-only. Search the
  latest eight weeks again, now including OpenReview, which was installed on
  August 2. Then lock the pre-registration.
- **Crop consistency:** download CLIPSelf. On OrangeGrid, run A4 by ROI-pooling
  through the complete test set, and run A5 with the LABCLIP-style 1× fix.
  Compare them with the two stop rules. Search recent work again. Then lock
  the pre-registration.
- Send both pre-registrations to the professor for approval in the same
  meeting. The shared wiki already lists both projects.

### Later weeks

- **Weeks 2–3:** run RoboJudge A1/A2. Small judges run on OrangeGrid; the 72B
  batch runs in the booked Anvil-AI window or on Delta. For crop consistency,
  build the SigLIP2 cache on OrangeGrid and train adapters within the six-setting
  attempt limit.
- **Week 4:** run RoboJudge A3/A4, which are mainly CPU plus small judges. For
  crop consistency, test the second backbone and check that retrieval quality
  is kept.
- **Week 5:** regenerate DreamGen videos and score them with five judges on
  Anvil-AI or Delta. Finish crop-consistency seeds and confidence intervals.
- **Week 6:** analyze both projects and apply Holm correction when testing
  many claims.
- **Week 7:** write both papers.

By the September 18 abstract deadline, RoboJudge needs the H3 figure from week
one. Crop consistency needs the A3-versus-A1 result on its main backbone,
expected around week three.

## Risks to keep watching

- The Anvil-AI H100 queue is the main scheduling risk. Book the 72B and
  DreamGen windows early. Use Delta 8×H200 if those bookings fail.
- RoboReward and Cosmos-Reason2 need an inference stack that supports
  `qwen3_vl`. Record exact vLLM and Transformers versions before scoring.
- Six weeks after each pre-registration is locked, repeat the recent-work
  check. Watch the evaluator authors' groups and the groups moving aggregation
  ideas into training.

## Related

[[Prereg-RoboJudge-Audit]] · [[Prereg-Crop-Consistency-Distillation]] ·
[[GPU-Resources-Across-Clusters]] · [[Data-and-Caches]] ·
[[Data-Transfer-Between-Clusters]] · [[Anvil-H100-Qwen36-vLLM-Benchmark]] ·
[[Method-Gates-Wave-2-2026-08]]

# Implementation Plan — ICLR 2027 Pair (RoboJudge Audit + Crop-Consistency Distillation)

Written 2026-08-02. Operationalizes [[Prereg-RoboJudge-Audit]] and
[[Prereg-Crop-Consistency-Distillation]] — cluster placement, data/weights
staging, and the order of work to the Sep 18 abstract deadline. Facts
verified live today are marked (✓ 2026-08-02).

## Live de-risking results (✓ 2026-08-02, HF API)

- **RoboArena dump is small: 18.5 GB** ([`RoboArena/DataDump_02-03-2026`](https://huggingface.co/datasets/RoboArena/DataDump_02-03-2026),
  `usedStorage` = 18,473,184,727 B; MIT; 23.6K metadata rows in the parquet
  export). Fallback [`DataDump_08-05-2025`](https://huggingface.co/datasets/RoboArena/DataDump_08-05-2025) = 8.1 GB. The prereg's "dump size
  unverified" go/no-go risk is resolved — fits on `$SCRATCH` trivially,
  labels can live in project space. Remaining week-1 check is only
  hydration + one end-to-end session parse.
- **RoboReward-4B/8B live** (`teetone/RoboReward-*`, CC-BY-4.0). Both are
  **Qwen3-VL** finetunes → needs a recent transformers/vLLM with `qwen3_vl`
  (our Anvil H100 vLLM stack qualifies; see
  [[Anvil-H100-Qwen36-vLLM-Benchmark]], [[CUDA-Compatibility-and-vLLM]]).
- **Cosmos-Reason available**: [`nvidia/Cosmos-Reason1-7B`](https://huggingface.co/nvidia/Cosmos-Reason1-7B) (Qwen2.5-VL base,
  plain transformers) and newer [`Cosmos-Reason2-2B/8B`](https://huggingface.co/nvidia/Cosmos-Reason2-8B) (Dec 2025, Qwen3-VL
  base). License "other" (NVIDIA) — fine to evaluate in a paper; record the
  license in the manifest. Use Reason2-8B as primary, Reason1-7B as the
  comparability fallback.
- **CLIPSelf checkpoints**: [`wusize/clipself`](https://huggingface.co/wusize/clipself) exists on HF (plus the GitHub
  releases). No license (known) — evaluate, never fork.

## Corrections (2026-08-02, evening — from the blocked SigLIP2-cache probe)

- **Our Anvil account `cis261253-ai` is AI-partition-only.** `sbatch` to the
  A100 `gpu` partition is REJECTED ("Please use --partition=ai"). Every
  "Anvil A100" placement below is stale — Anvil work runs on `ai` (H100)
  only, proportionally shaped (1 GPU → 24 CPU / 250 GB). Observed `ai`
  queue: 97 pending, ~7-day estimated start (`sbatch --test-only`, free
  probe) — **book 72B-judge and DreamGen windows NOW or use Delta 8×H200 as
  primary**, not fallback.
- **SigLIP2 secondary backbone is pinned to [`ViT-B-16-SigLIP2-256`](https://huggingface.co/timm/ViT-B-16-SigLIP2-256)/webli
  via open_clip** (the lab's backbone of record — all svib configs, cache
  names, and the rebuttal provenance). The HF `google/siglip2-base-*` tags
  are NOT loadable by the existing extractor backend. The staged 224 HF
  checkpoint stays unused.
- **SigLIP2 teacher-cache build moves to OrangeGrid**: the extraction
  pipeline lives in `svib` (cropdistill reuses it), its required template
  H5s (the frozen image-set/exclusion specs: VG 108,073 rows, COCO 5,000,
  WG 800) and the 193 GB of existing caches are OG-resident, and OG has
  the images as files. Anvil has none of these. Grid definition confirmed:
  GRID_LEVELS 20 local + 1 global, roi_align 1×1 aligned.
- RoboJudge small-judge sweeps: OG primary (unchanged); the "Anvil A100
  data-local" alternative is void per the partition fact.

## Cluster assignment

| Work item | Cluster | Why |
|---|---|---|
| RoboArena staging, frame extraction, Bradley–Terry, H3 figure | **Anvil CPU + `$SCRATCH`** | 18.5 GB download; ranking is CPU-only; data home is Anvil |
| 2–8B judges (RoboReward-4B/8B, Qwen2.5-VL-7B, InternVL3-8B, Cosmos-Reason, SigLIP2 scorer) + confound battery A3 + degenerate set A4 | **OrangeGrid** (2×A100/L40S), Anvil A100 overflow | Free, unbounded walls; after frame extraction the working set is a few GB — trivially transferable |
| Qwen2.5-VL-**72B** judge | **Anvil AI 4×H100**, vLLM | 320 GB GPU mem, benchmarked stack; batch-score in one or two wall-clock windows |
| A5 DreamGen video regeneration (60–120 GPU-h) | **Anvil AI H100** | Video diffusion wants H100; checkpoint between jobs for wall limits |
| Crop-distill: A4/A5 week-1 checks, adapter training (6 configs × 3 seeds), full battery evals, H6 timing | **OrangeGrid** | SVIB eval stack + L40S timing protocol live there; cached-feature training is ≤2-GPU work; free |
| SigLIP2-B/16 teacher-cache build (crop-grid re-encode of COCO/VG) | **Anvil A100** (single node) | Data read in place from `/anvil/datasets/ai`; a few GPU-h; then ship the H5 cache to OG via Globus |
| **Delta** | **Not used** | No job here needs >4 GPU/node or H200 memory. Reserve as fallback if Anvil-AI queues block the 72B judge (8×H200 node) |

Credit posture: everything that fits 2 GPUs defaults to OrangeGrid;
ACCESS credits are spent only on (i) the 72B judge, (ii) DreamGen video
generation, (iii) the SigLIP2 cache build — roughly 100–200 H100-h +
20–40 A100-h total, well inside both preregs' envelopes.

## Data and weights to stage (into `artifacts/hf-cache` unless noted)

**RoboJudge:**
1. [`RoboArena/DataDump_02-03-2026`](https://huggingface.co/datasets/RoboArena/DataDump_02-03-2026) →
   `/anvil/projects/x-cis261253/datasets/roboarena/dump/` (manifest +
   SHA-256 per [[Data-and-Caches]]). Owner decision 2026-08-02: the whole
   dump lives in **project space** (18.5 GB against a 4.6 TB-free quota),
   not `$SCRATCH` — avoids the 30-day purge entirely; no separate labels
   copy needed. Download quirk: use `HF_HUB_DISABLE_XET=1` — the dump's
   thousands of small files rate-limit (429) the xet token endpoint.
2. Models: [`teetone/RoboReward-4B`](https://huggingface.co/teetone/RoboReward-4B), [`teetone/RoboReward-8B`](https://huggingface.co/teetone/RoboReward-8B),
   [`Qwen/Qwen2.5-VL-7B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct), [`Qwen/Qwen2.5-VL-72B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-VL-72B-Instruct) (+ its AWQ
   as the 2×A100 fallback), [`OpenGVLab/InternVL3-8B`](https://huggingface.co/OpenGVLab/InternVL3-8B),
   [`nvidia/Cosmos-Reason2-8B`](https://huggingface.co/nvidia/Cosmos-Reason2-8B) (license note), SigLIP2 (shared below).
3. [DreamGen](https://arxiv.org/abs/2505.12705) Bench assets: the four published video models, bench prompts,
   and the published downstream policy-success vector (from paper/repo) —
   verify exact checkpoints at A5 kickoff, not now.

**Crop-consistency:**
1. Backbones: OpenCLIP ViT-B/32 **QuickGELU-correct pair** (standing
   lesson: GELU mismatch silently corrupts source-matched comparisons) and
   `google/siglip2-base-patch16-*` (pin the exact resolution tag in the
   cache manifest — encoder provenance is the model pair).
2. [`wusize/clipself`](https://huggingface.co/wusize/clipself) checkpoint (HF or [GitHub release](https://github.com/wusize/CLIPSelf)) — the A4 week-1
   decisive check.
3. A5 concretizations from the wave-2 gate: LABCLIP ([arXiv 2502.03566](https://arxiv.org/abs/2502.03566)) —
   ~590K-param text-side matrix, reimplement in a day if unreleased; [DCSM
   (ICCV 2025)](https://arxiv.org/abs/2503.08723) — check code release; [DeGLA](https://arxiv.org/abs/2504.16801) checkpoint — check release,
   else published numbers (per prereg).
4. Reuse: `artifacts/svib_features` (CLIP crop-grid teacher features + the
   trained grid+self-attention head = our multi-region teacher);
   battery datasets already in the lab root (`sugarcrepe`, `sugarcrepepp`,
   `winoground`, `vsr`, `external_compositional`, `negclip` — mind the
   NegCLIP/COCO-val contamination rule); COCO + VG read in place.
5. New cache to build: SigLIP2-B/16 features for the 20-view grid + patch
   tokens over the same image set (est. ~2.5M crop encodes ≈ single-digit
   A100-h; H5 keyed by image id, manifested).

## Order of work (both projects run in parallel)

**Week 1 (lock week, target Aug 4–10):**
- RoboJudge: download dump → hydrate → parse one session end-to-end →
  freeze the Bradley–Terry human ranking + bootstrap CIs → produce the H3
  n-sensitivity figure (all CPU). Confirmatory lit pass (8 weeks, now with
  OpenReview coverage — installed Aug 2). → prereg LOCK.
- Crop: pull CLIPSelf checkpoint → run A4 (ROI-pooled through the battery)
  and A5 (LABCLIP-style fix at 1×) on OG → decisive check against the two
  kill criteria. Confirmatory lit pass. → prereg LOCK.
- Both preregs to professor for sign-off in the same meeting; the shared
  wiki already carries the slate.

**Weeks 2–3:** RoboJudge A1/A2 (small judges on OG; 72B batch on Anvil-AI).
Crop: SigLIP2 cache build (Anvil) + adapter training sweep within the
6-config attempt budget (OG).
**Week 4:** RoboJudge A3/A4 (confounds + degenerate injection — mostly CPU
+ small judges). Crop: secondary backbone + retrieval preservation.
**Week 5:** RoboJudge A5 (DreamGen regeneration + 5-judge scoring,
Anvil-AI). Crop: seeds/CIs.
**Week 6:** analysis + Holm-corrected stats, both. **Week 7:** write-ups.
**Sep 18 abstract needs:** H3 figure (RoboJudge — available week 1) and
A3-vs-A1 on the primary backbone (crop — available ~week 3).

## Standing risks to watch

- Anvil-AI H100 queue depth is the only scheduling risk → book the 72B and
  DreamGen windows early; Delta 8×H200 is the fallback shape.
- RoboReward/Cosmos-Reason2 need `qwen3_vl`-capable inference — pin the
  vLLM/transformers versions in the env manifest before scoring anything.
- 6-week re-gate clocks from both preregs (evaluator authors' orbit;
  aggregation-line groups moving training-side) run from lock date.

## Related

[[Prereg-RoboJudge-Audit]] · [[Prereg-Crop-Consistency-Distillation]] ·
[[GPU-Resources-Across-Clusters]] · [[Data-and-Caches]] ·
[[Data-Transfer-Between-Clusters]] · [[Anvil-H100-Qwen36-vLLM-Benchmark]] ·
[[Method-Gates-Wave-2-2026-08]]

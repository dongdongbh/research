# Unified Research-Direction Ranking — version 3, 2026-08-05

**Status: THIS IS THE CURRENT RANKING — everything in one place.** It
replaces version 2 (Aug 3, kept in git history) and folds in: the wave-2
and wave-3 method gates, the role-decodability probe results, the TMLR
decision for SVIB, the crop-project hold, and the Aug-5 scouts of three
new areas (generative video, LLM/VLM robotics, AI-for-science) with their
five formal checks.

**The one lesson to carry:** across 18 formally checked ideas this month,
scout/first-look ratings dropped by 1–2 stars on average once a deep check
ran, and the two worst kills were papers less than eight weeks old with
zero citations. Nothing enters a pre-registration without the two-pass
check, an OpenReview sweep, and a verified first-cheap-step.

## Part 1 — Committed work (not ranked; already chosen)

| Deadline | Project | State |
|---|---|---|
| ICLR (Sep 18) | **RoboJudge audit** ★★★★½ — do robot-policy evaluators recover the human ranking | Week 1 done; ranking frozen; **ready to lock** at sign-off (4 pinned choices) |
| ICLR (Sep 18) | **Crop-consistency distillation** | **ON HOLD** — insight published April ([2604.11496](https://arxiv.org/abs/2604.11496)); decide at sign-off: fold into the frontier paper (recommended) / standalone ★★ / bench |
| CVPR (~Nov 13) | **1-NFE diversity** (one-step image generators) | Prereg drafted; unchanged |
| CVPR (~Nov 13) | **Readout-budget vs binding frontier** ★★★½ (Cell A) | Method-slot candidate; 1-day pilot required before prereg; absorbs the crop machinery as its method arm |
| ICML (~Jan 28) | **Epistemic contextualization** | Prereg drafted; citation corrected to [2606.29657](https://arxiv.org/abs/2606.29657); pipeline work can start |
| TMLR (no deadline) | **SVIB audit paper** | Decided Aug 4: one paper — audit + locked protocol + grid+attention positive + released suite; writing task ~2–3 weeks |
| Running | **Role-decodability program** | Powered result in hand (99.5% in patches → 50.1% at the score; readout inverted); owner labeling verb data; feeds Cell A and the algebraic-binding idea |

## Part 2 — The all-in-one ranking of available directions

Stars are post-check where a check ran (marked ✓); pre-check otherwise.
"First step" is the cheapest action that decides whether to continue.

### ★★★★½–★★★★ — strongest available

| Direction | ★ | Cost (GPU-h) | First step | Note |
|---|---|---|---|---|
| Abbeel parallel-RL factorial (which fast-RL ingredients matter) | ★★★★½ | 400–650, OrangeGrid | re-check the source group's latest releases | **Expiring** — the authors are working through the axes; swap-in for a slate slot or it dies |
| Replay-mechanism arbitration ([Liang 2603.04964](https://arxiv.org/abs/2603.04964)) | ★★★★ | 250–400 | loss-spike detector pilot | Five candidate causes, each with a cheap falsifier |
| RLVR vs self-consistency calibration | ★★★★ | 100–250 | verify OLMo-3 checkpoint availability | Could ship calibrated agreement aggregation |
| KV-cache footprint under agentic workloads | ★★★★ | 200–400, eval only | confirm PruLong replacement plan | Turn-aware eviction is the unlocked method |
| Bengio sparsity-premise test (Req. 5.23) | ★★★★ | ~250 | design the 300-model sweep | Decides whether contextualization gets its theory arm |
| [MOCHI](https://arxiv.org/abs/2409.05862) VLM arm | ★★★★ | 20–60 | run it — it is nearly free | Cheapest real result on the board |
| GMP three-mechanism arbitration (Song) | ★★★★ | 100–250 | **read the PDF first** — check self-ablation | Unchanged from v2 |

### ★★★½–★★★ — real but narrowed (all ✓ checked)

| Direction | ★ | Cost | First step (all cheap and decisive) | What the check found |
|---|---|---|---|---|
| **Robot-evaluator uncertainty audit** ✓ (Aug 5) | ★★★ | 200–400 | **0 GPU, one afternoon**: Fisher-z intervals on the published n=4/5/8 correlation tables — if the n=4 interval spans ~[0,1], the headline exists | Survives narrowed: two papers DO report intervals, but never on the ranking statistic itself; sim-vs-sim instability floor is unclaimed; **must write the boundary vs RoboJudge first** (same data, same construct family); new best target: RoboDojo (30 policies) |
| **Video role-direction test** ✓ (Aug 5) | ★★★ | 300–600 | **0 GPU**: score 3–4 generative judges on VELOCITI's human-labeled agent-swap pairs; no judge ≥85% → kill | Gap verified by cloning repos (video benchmarks dropped the swap idiom the image benchmark ships); the risk moved to the judge: video-LLMs score 44–49% vs 93% human on exactly this judgment |
| Algebraic role-binding embeddings (probe-promoted) | ★★★ (cond.) | 130–250 | OpenReview re-check, then design | Crux cleared for spatial roles by the probe; verb case pending owner labels; [OC-CLIP](https://arxiv.org/abs/2502.14113) is the cross-encoder-shaped neighbor |
| Anthropic VLA-supervision replication | ★★★½ (pre-check) | 100–200 + API | gate it if pursued; their repo is still unreleased | Their n=36 headline; discount expected on check |
| Choi hivemind decomposition | ★★★½ | low | resolve the 404'd dataset | Blocked on data |
| T1 narrowed interaction study | ★★★ | 150–250 | use PIVOT's harness | Mid-venue target |
| KV three-way interaction | ★★★ | see v2 | pilot only | Port-and-scale, underpowered risk |

### ★★½ and below — bench (run only as cheap side arms)

| Direction | ★ | First step |
|---|---|---|
| Video coverage re-measurement ✓ (Aug 5) | ★★½ | ~15 GPU-h: score public few-step checkpoints with off-the-shelf STREAM-D/TopP&R; if coverage tracks dispersion (ρ>0.8), kill. Metric novelty is dead ([STREAM, ICLR 2024](https://arxiv.org/abs/2403.09669)); only the audit remnant lives; must solve "coverage of what?" (no real-video reference for T2V prompts) |
| Fork-preserving context construction (wave-2 survivor) | ★★½ | 48-h re-check, then the operator pilot | Lane moves ~1 paper/month |
| Cell B: per-item position split | ★★½ | 5 GPU-h pilot rides Cell A encodes | |
| Spectral-band × knowledge injection (MiCA) | ★★ | 10 GPU-h repro gate — fails → dead | |
| Accept-rule run-level residual | ★★ | ~20 GPU-h seed-variance check | PACE is the mandatory baseline |
| V2A secondary analysis (salvage) | note-scale | days: bootstrap CIs + power curves on [SynthSync's released 306K ratings](https://arxiv.org/abs/2607.09091) | Not a paper; a methods note |
| Wei verifier-rule Q1 · polychromic audit · Arora drag-fork addon · SigLIP-2 ladder (next-cycle, Delta) · environment-provenance study | ★★–★★★ | as in v2 | Environment-provenance decision window has lapsed — revisit or drop at sign-off |

## Part 3 — Killed, vetoed, or expired (consolidated — do not re-propose)

**Aug-5 checks:** language-necessity index (Level-1 kill — [2606.04233](https://arxiv.org/abs/2606.04233) published the cross-benchmark study in June; LIBERO-Plus ran real policies blind) · V2A metric validation (killed — [SynthSync 2607.09091](https://arxiv.org/abs/2607.09091), 306K annotations, plus Omni-Judge/PEAVS/AVBench) · **seed-noise/variance of agent leaderboards (OWNER VETO: not significant, no barrier, not novel — includes the design-layer arm)**.

**Waves 1–3 and earlier (see [[Method-Gates-Wave-2-2026-08]], [[Method-Gates-Wave-3-2026-08]], v2 in git history):** autoresearch accept rule (PACE) · cross-scaffold critic (source group published) · active-view spatial (World2VLM/SIMS-V) · compositional merging (AlignMerge) · frozen-VLM binding heads (DCSM/Q-Former) · KV from-scratch factorial (CLA/MixAttention) · late-interaction distillation (ComAlign) · spectral-PEFT LR-artifact question (answered 4×) · verifier hardening · safety-aware KV · Hyperball · CAID · multi-agent budget-matching · LPT control · PULSE · BPP (LIBERO-Gen does not exist) · DriveJudge (labels unreleased; watchlist) · weather · novelty forensics (biology-side owned) · T4 anneal-window · B1 diversity attribution.

## Part 4 — The cheap decisive steps, gathered in one list

1. Fisher-z afternoon (robot-evaluator audit) — 0 GPU.
2. VELOCITI judge audit (video role direction) — 0 GPU, API only.
3. Cell A dynamic-range pilot — 1 day.
4. MOCHI — 20–60 GPU-h, run whenever.
5. STREAM-D vs dispersion correlation (video coverage) — ~15 GPU-h.
6. MiCA repro gate — 10 GPU-h.
7. Owner: verb labeling sessions (powers the below-chance real-photo lead).

## Part 5 — Standing lessons (updated)

All of v2's lessons stand. Added from Aug 5: (6) **scout ratings are
provisional by ~1.5–2 stars** — never cite a scout star in a decision
document; (7) **zero-citation papers under eight weeks old are the main
scoop source now** — recency-weighted search matters more than breadth;
(8) two ideas this month died to papers that our own earlier sweeps had
already surfaced for a different purpose — check our own wiki first.

## Related

[[Direction-Scouts-2026-08-05]] · [[Method-Gates-Wave-3-2026-08]] ·
[[Compositional-VLM-Survey]] · [[Binding-Root-Cause-Analysis]] ·
[[Prereg-RoboJudge-Audit]] · [[Prereg-Crop-Consistency-Distillation]] ·
[[Prereg-1NFE-Diversity]] · [[Prereg-Epistemic-Contextualization]] ·
[[Top-Researcher-Scan-2026-08]] · [[Home]]

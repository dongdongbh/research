# Unified Direction Ranking — 2026-08-03
## (Direction Re-evaluation, v2)

Status: **CURRENT AUTHORITY.** Supersedes the ranking in
[[Direction-Reevaluation-2026-08]] (kept as record) and the opportunity lists
in [[Top-Researcher-Scan-2026-08]]. Produced by a full gating sweep: 8
parallel lane agents + 1 teammate substrate audit, each running lane-wide
literature search (not person-scoped), claimed-or-not with quotes, substrate
liveness verification, resource fit against the verified fleet, a concrete
pre-registered design, the method paper each diagnosis unlocks, and ICLR-2027
(abstracts Sep 18) feasibility.

Coverage caveat shared by all lanes: WebSearch/OpenAlex budgets exhausted and
arXiv/S2 APIs rate-limited during the sweep; evidence is arXiv/HF-heavy and
venue-native proceedings (CoRL/RSS/GECCO/ICSE) were under-swept. Each lane
recorded its specific gaps; one clean confirmatory pass is advised before any
abstract is committed.

## The ranking

### Flagship tier

| # | Direction | ★ | Cost / cluster | ICLR? | Method-unlock |
|---|---|---|---|---|---|
| 1 | **Autoresearch FDR + sound accept rule** — **DECLINED as a paper by owner 2026-08-03** (statistics-as-contribution reads as engineering; see the removed autoresearch accept-rule draft (git history)). Retained as internal tooling only | ~~★★★★★~~ | — | — | — |
| 2 | **Judge/evaluator validity-audit program** (anchor: [RoboReward](https://arxiv.org/abs/2601.00675); + [DreamGen](https://arxiv.org/abs/2505.12705) judge-swap + off-the-shelf VLM judges; ground truth = [RoboArena](https://arxiv.org/abs/2506.18123) human dumps) | **★★★★½** | 250–500 GPU-h · inference-only, OrangeGrid | **Yes** | Rank-calibrated evaluator + "evaluator report card" |
| 3 | **Abbeel parallel-RL factorial** (BRC vs TD-overfitting contradiction; factorial around authors' own published config) | **★★★★½** | 400–650 GPU-h · OrangeGrid, MuJoCo-only | **Yes** | Minimal [FastTD3](https://arxiv.org/abs/2505.22642) recipe |

### Strong tier (★★★★)

| Direction | Cost | ICLR? | Note |
|---|---|---|---|
| Replay-mechanism arbitration (Liang [2603.04964](https://arxiv.org/abs/2603.04964); 5 mechanisms incl. loss-spike discriminator) | 250–400 · OG | Yes | Unlocks replay-free stage-transition schedule; Marin dual-track (contribute infra, keep hypotheses private) |
| RLVR × self-consistency calibration tension (D. Zhou; DCPO misses on all 3 axes) | 100–250 · OG+Anvil | Yes | Unlocks calibrated-agreement aggregation; Olmo-3 lineages free |
| Cross-scaffold critic transfer (+ [PACE](https://arxiv.org/abs/2606.08106) forward test as validity section) | 100–500 + API | Yes | Ships 8B critic weights; outcome-labeled traces are our edge ([TraceLab](https://arxiv.org/abs/2606.30560) killed exclusivity) |
| 1-NFE diversity: averaging vs intrinsic (Drifting = falsifier; recall computable today, unreported) | 150–350 | Likely | Different venue/community — keep separate from LLM collapse |
| Agentic KV-footprint audit (footprint metric never tested on agent workloads; gap publicly named Jul 9) | 200–400 · OG eval-only | Yes | Unlocks turn-aware eviction; PruLong checkpoints missing — drop or retrain |
| Bengio sparsity-premise falsification (Req 5.23; ~300 small predictors) | ~250 · OG | Yes | Gates the full contextualization build (M4) |
| [MOCHI](https://arxiv.org/abs/2409.05862) VLM arm (human RTs shipped; unclaimed across all citations) | 20–60 · OG L40S | Yes | Perception-aligned training signal |
| Bundle A "the readout, not the representation" (Isola shared-Q on compositional probes + map-class ladder + COCO-nuisance join; binding-locus as a cited rung) | <100 · cached features | Yes, gated | Compositionality-preserving alignment map. **Gate: zero-GPU DataComp 7×7 Jaccard first** |
| Sutskever RL-environment provenance | 350–600 + heavy authoring | No — ICML | **2-week decision clock** (Microsoft Echoverse owns adjacent apparatus) |
| GMP 3-mechanism arbitration (Song) | 100–250 | Yes | **Gate: read the PDF — does it self-ablate?** |
| Levine offline-RL arbitration, reframed as theory-test of [2601.00831](https://arxiv.org/abs/2601.00831)'s 3 failure modes | ~840 (was 200–500) | Tight | **Expires ~Sep 13** — independent group ran our methodology on OGBench Jul 29 |
| SigLIP-2 ingredient ranking across scale | 1,000–1,400 H100-h · **Delta 8×H200-long** | No — ICML/CVPR | 3 of 5 ingredients must be built in OpenCLIP (big_vision frozen) |
| Arora drag=fork arbitration (bolt-on) | 50–200 | Yes | Fork-preserving context intervention |

### Demoted this sweep

| Direction | Was | Now | Why |
|---|---|---|---|
| **T1 freeze × objective × stage (as designed)** | ★★★★½ #1 | **★★★** | Three of four cells published at CVPR/ICLR/ICML 2026 (CoVFT; **PIVOT — title: "RL makes MLLMs see better than SFT"**; From Seeing to Thinking). The flagged scoop risk (Darrell) was the wrong group. Budget was also ~5× low vs PIVOT's own 144 H100-h/recipe. **Narrowed survivor (★★★★): interaction-term + {tuned}×{auxiliary} + GRPO arm at 1.5–3B on PIVOT's harness, 150–250 GPU-h, mid-tier venue** |
| KV from-scratch factorial (M1) | ★★★★ (scan) | **★★★** | Dropped twice: [Cost-Optimal GQA (EMNLP 2025)](https://arxiv.org/abs/2503.09579) ran the from-scratch KV-head sweep incl. n_kv=1; [CLA](https://arxiv.org/abs/2405.12981) crossed heads×tying at matched bytes; [MixAttention](https://arxiv.org/abs/2409.15012) crossed tying×window. Survivor: the three-way interaction only, pilot-gated. LCKV repo implements all 3 axes (port-and-scale, not invent) |
| Choi hivemind decomposition | ★★★★ (scan) | ★★★½ | "First decomposition" dead — 3 rival causal accounts published Apr–May 2026, mutually unreconciled. Survivor: inter-model axis + 3-account arbitration. **Blocker: Infinity-Chat HF dataset 404s** |
| Wei verifier-rule Q1 | ★★★★ (scan) | ★★½ | ICML 2026 37-author saturation study owns corpus + attention control; verifiability axes absent but delta is absorbable |
| CAID budget-matching (as paper) | Tier-1 (scan) | internal only | Headline answered 5× in 8 months, ending in **Nature MI 2026-07-24**; honest cost $2–5k. Run narrowed edit-isolation internally |
| Polychromic set-RL audit | ★★★★ (scan) | ★★½ | Confound real but **zero public code for all three targets** — a null on a reimplementation is not defensible |
| M5 diversity-preserving method | method dir. | benchmark only | 15+ occupants; the opening is the matched-pass@1 measuring stick |

### Killed this sweep (evidence in lane reports)

Hyperball factorial (scooped [2607.22444](https://arxiv.org/abs/2607.22444) + design mis-specified + refuted in
advance by source group) · M7 verifier hardening (claimed 2×, 3 days apart,
Jun 2026) · **safety-aware KV allocation** ([2606.09864](https://arxiv.org/abs/2606.09864) ran the exact study
Jun 1 — "no universal safe bit-width", diverge published; AnchorKV Jun 16 is
the allocator; **both predate our July gate that called it unclaimed**) ·
binding-locus arbitration as framed (LABCLIP Feb 2025 + [DCSM ICCV 2025](https://arxiv.org/abs/2503.08723)) ·
VPBench grid as proposed (category error; salvage = COCO annotation join) ·
LPT token-matched control (conclusion published ACL 2026; target superseded) ·
PULSE replication (no deployment path) · DriveJudge audit **for now**
(annotations unreleased — watchlist, re-gate in 6 wks) · BPP dose-response
(**LIBERO-Gen does not exist as a public artifact**) · Cosmos Policy audit as
scoped (target superseded by Cosmos 3; salvage = build the procedural-shift
generator as the artifact).

## Recommended portfolio (professor's decision; our suggestion)

**Start now, aimed at ICLR Sep 18** (pre-registrations drafted:
[[Prereg-Crop-Consistency-Distillation]], [[Prereg-RoboJudge-Audit]]):
1. **Crop-consistency distillation** (method flagship; selected 2026-08-04
   after the owner's real-method criterion re-gated the slate — see
   [[Method-Gates-2026-08]]. Week-1 decisive checks: CLIPSelf checkpoint on
   our battery; aggregation-fix arm).
2. **Judge-audit program first paper** — uses the VLM eval infra,
   inference-only, disjoint resources from #1. Week-1 go/no-go: RoboArena
   dump storage footprint.
3. **Cheap bolt-ons in parallel:** [MOCHI](https://arxiv.org/abs/2409.05862) (20–60), calibration tension test
   (100–250), Bundle A if the Jaccard gate passes (<100).

**Next cycle — SELECTED (pre-registrations drafted):** 1-NFE diversity →
CVPR 2027 ([[Prereg-1NFE-Diversity]]); epistemic contextualization → ICML
2027 ([[Prereg-Epistemic-Contextualization]], pipeline engineering starts
now; **revised to a mid-training design on OLMo-2-1B, ~300–450 GPU-h**, with
a gated from-scratch escalation). Bench (designs in git history / lane reports): parallel-RL factorial
(expires with time — the source group is sweeping its axes serially),
SigLIP-2 ingredient ladder, replay arbitration, calibration tension test.

**Also holdable for next cycle:** Abbeel factorial (or swap it in
for #2 if the professor prefers a method-flavored flagship), SigLIP-2
ingredient ranking (start OpenCLIP engineering now), replay arbitration
(or run now — it also fits ICLR), provenance study (decide within 2 weeks).

**This week's zero/low-cost gates:** DataComp 7×7 Jaccard (0 GPU) · RoboArena
dump size check · GMP PDF read · Infinity-Chat dataset resolution ·
pre-register autoresearch noise floor and launch the 30 repeats (~10 GPU-h).

## Cross-lane conflicts resolved

- The collapse direction's "statistical stopping rule" secondary track is
  **struck** — it belongs to flagship #1 (same claim, same reviewers).
- The 60k-trajectory agentic-variance study ([2602.07150](https://arxiv.org/abs/2602.07150)) is fit **once** as a
  shared variance/power module serving flagships #1–#2 and the critic paper.
- The predictive-validity framing ([2606.19704](https://arxiv.org/abs/2606.19704)) is assigned to the critic/PACE
  paper, not the FDR paper.
- LLM collapse and 1-NFE diversity stay **separate papers** sharing one
  instrumentation layer (matched-quality coverage protocol).

## Process lessons added by this sweep

1. **Re-gates must re-identify the competitor, not just re-check the claim.**
   T1's cells went to three venues while we watched the wrong group.
2. **Lane-wide sweeps beat person-scoped checks** — four scan premises died
   on lane search (CAID, hivemind-decomposition, M7, safety-KV).
3. **Every search must explicitly cover the most recent 8 weeks** — the July
   KV gate's "6 searches, 0 hits" missed two June papers that had already
   killed the direction.
4. **Substrate liveness gates outcomes, not just cost** — a null on your own
   reimplementation of unreleased code is not defensible (polychromic); a
   benchmark cited by a profile may not exist (LIBERO-Gen).
5. **Verify quoted numbers against primary artifacts** — the "7-run 0.016
   CORE spread" figure circulating in our own notes could not be verified and
   was replaced with verbatim-sourced evidence.

## Related

[[Direction-Reevaluation-2026-08]] (superseded ranking, kept as record) ·
[[Top-Researcher-Scan-2026-08]] · [[Self-Improving-AI-Survey]] ·
[[Method-Opportunities]] · [[Live-Research-Opportunities]] · [[Home]]

# Method Gates — Wave 2, 2026-08-02

Second brainstorm-and-gate round. Source material: our own evidence base
(SVIB post-mortem, sweep numbers, collapse/diversity lane) plus
[[Top-Researcher-Scan-2026-08]] method directions (per the PI directive).
Twelve brainstormed ideas + three scan-derived; seven killed by knowledge we
already held; **seven candidates through live gates, run by seven parallel
Opus agents** (multi-source search incl. mandatory ≥2026-05-01 recency pass
+ last-8-weeks web sweep; 2–6 full-paper deep reads each with verbatim
evidence; 4-axis overlap scoring, worst-case level).

**Method definition (owner, updated 2026-08-02).** Three admission routes:
(1) a new mechanism — architecture, training procedure, representation,
broken assumption, changed data/workflow (diffusion, VGGT, D4RT, 3DGS,
Dreamer); (2) an improvement on a current method; (3) **an old method
applied to a new problem**. Diagnosis/ablation/statistics alone do not
qualify; statistics-flavored candidates pass only if the deliverable is a
shipped working mechanism.

## Instant self-gates (no search needed)

| Idea | Kill reason |
|---|---|
| M6 generative-teacher distillation | Absorbed — one teacher arm of the crop-distillation flagship |
| M7 uncertainty-routed two-tier ITM | Route-3 method but transfer too familiar; absorbed as a flagship inference-policy arm |
| M8 stage-transition smoothing | Scooped — 2607.00634 (found during replay gate) |
| M9 granular-nonlinearity block | Wrong regime — pretraining-scale architecture race |
| M10 turn-aware KV eviction | Engineering per owner definition; MemDecay owns the observation; gated on D. Chen audit |
| M11 diversity-preserving post-training | Saturated lane (15+ occupants); gated on collapse-mechanism diagnosis |
| M12 format-anchored RL | Premature — method before the collapse arbitration has run |

Scan exclusions: S-M4 is already the ICML flagship; S-M5/S-M6 stay
gated-on-diagnosis per the scan's standing rule; S-M7 folds into the E lane.

## Live-gate scoreboard

| # | Candidate | Verdict | Lvl | Closest prior | ★ |
|---|---|---|---|---|---|
| M1 | Energy/predictive scoring head (frozen VLM) | **SCOOPED** | 2 | DCSM (ICCV 2025); 2604.11496 | 0 |
| M2 | Binding-register adapter tokens | **SCOOPED** | 1 | SGVL (EMNLP 2023); BLIP-2 Q-Former mechanism match | 0 |
| M3 | Fork-preserving context construction | **SURVIVES-NARROWED** | 3 | 2607.26117 (5 days old); DeepMind 2604.05868 | ★★½ |
| M4 | Compositionality-preserving merging maps | **SCOOPED** | 2 | AlignMerge 2512.16245 | ★ |
| M5 | Active-view spatial pretraining | **SCOOPED** | 1 | World2VLM 2604.26934; SIMS-V 2511.04668 | ½ |
| S-M1 | From-scratch KV recipe @ matched bytes | **SCOOPED** | 2 | CLA 2405.12981 (NeurIPS 2024); MixAttention | ★ |
| S-M2 | FDR-controlled accept rule (autoresearch) | **SCOOPED** | 2 | PACE 2606.08106 (Jun 2026) | ★½ |
| S-M3 | Cross-scaffold agent critic | **SCOOPED** | 2 | Neubig's own 2603.03800 (Mar 2026) | ½ |

## Gate records (condensed)

**M1/M2 — frozen-encoder binding heads.** DCSM (ICCV 2025) occupies the
learned-1×-readout-on-frozen-CLIP slot; DiffusionITM's 17× cost (68 min vs
4 min per their own numbers) was M1's one real delta and DCSM already spent
it. M2 = BLIP-2 Q-Former's ITC path verbatim (32 query tokens on frozen
ViT, max-over-query scoring, hard-negative ITM — verified; zero
compositional eval in BLIP-2), stacked on SGVL → Level 1. **Byproducts for
the flagship:** "LABCLIP" is really **2502.03566 (ICLR 2026)** — D×D
text-side matrix, frozen encoders, ARO+SugarCrepe → mandatory baseline; the
whole neighborhood (DCSM, LABCLIP, ABE-CLIP, TF-Local) reads out *existing*
frozen features — none injects a stronger teacher into the 1× patch path,
which is exactly the flagship's mechanism. FILIP/SPARC/DINOSAUR downgrade
to citations (no compositional eval, not heads-on-frozen).

**M3 — fork-preserving context (the survivor).** Framing and insight are
published: 2607.26117 (blind resample beats self-repair below 7B at 2.5–5.5×
fewer tokens; anchoring reproduces the failed program in 33–68% of retries),
DeepMind 2604.05868 (sequential deficit = exploration loss, and anchoring is
**correctness-agnostic**), 2605.08563 (contamination model + clean-restart
dominance theorem), Meta 2604.16529 (summarize operator shipped on
SWE-bench/Terminal-Bench). **Unoccupied cell:** operators that keep a
failure's *information* while removing its *anchoring* (redact /
counterfactual-rewrite, beyond keep-vs-wipe), selected by an
entropy-at-forking-tokens diagnostic. Win regime: long-horizon agentic tasks
where a wipe discards expensive environment knowledge — NOT short math/code
retry, where blind resample provably wins. Kill-arm: blind resample at
matched token budget. Required control: same operators on *successful*
attempts (correctness-agnostic check). Lane pace ~1 adjacent paper/month
since May; **48h re-gate before any commitment.** ★★½.

**M4 — compositional merging maps.** AlignMerge (2512.16245) is the same
template with the property swapped: "merging can preserve loss while quietly
destroying alignment" + explicit latent-preservation regularizer in the
merge objective. Representation Surgery (ICML 2024/25) is the ICML
precedent; Tam et al. 2409.18314 owns the title collision. Premise is also
contested: 2509.19476 and Mai (CVPR 2026) find merged/stitched models can
*exceed* parents on structural competence. Residual: nobody runs
compositional probes on merged CLIPs — a one-week **measurement** note at
most, not a method paper. Gate-process flag: the first pass missed
AlignMerge entirely; the verification sweep caught it.

**M5 — active-view spatial.** World2VLM (2604.26934) owns action-conditioned
view-transition mid-training with zero-shot SAT/VSI-Bench/MindCube transfer;
SIMS-V (2511.04668, by the VSI-Bench authors) owns sim-data-to-real-transfer
(25K examples beat a 72B baseline); Spatial-SSRL owns non-QA predictive
objectives. No axis left. Bonus hazard: 2511.16655 shows the
spatial-supersensing eval family is gameable. Abandon.

**S-M1 — KV recipe.** Matched-KV-bytes-per-token is CLA's own Figure-3 axis
(NeurIPS 2024); its from-scratch 1B/3B grid already contains the
MQA-vs-GQA overturn (MQA ties GQA-2 at half the bytes) and reaches the
proposed conclusion ("CLA2 + MQA"). MixAttention (2409.15012) published the
Character.AI local:global × cross-layer recipe with RULER-32K. Residual:
the three-way interaction (heads × tying × local:global) at long context —
affordable only at 300M, underpowered for the interesting 0.03–0.06-ppl
effects; Gemma 4 ships axes 1+2 as product default. ★.

**S-M2 — accept rule.** PACE (2606.08106) is the same paper: acceptor as
the silent weak point, greedy accept = "the agent p-hacks itself,"
anytime-valid e-process commit gate, online-FDR baseline already run (too
conservative). autoresearch PR #560 (unmerged 2σ noise floor) is the
repo-level incumbent; Rehearse (2607.27687) owns the nanochat testbed with
4,000 runs. Residual: run-level-scalar k-seed racing under GPU-hour cost
with a **run-level** FDR guarantee (PACE's is per-decision on paired
per-instance outcomes). ★½→★★ only if hard-narrowed; gate-before-gate: ~20
GPU-h σ_seed measurement on nanochat speedruns.

**S-M3 — agent critic.** Neubig's group already shipped it: 2603.03800 —
rubric-supervised 4B critic on 154K private OpenHands production segments,
Best@8 +15.9, early stopping at 83% compute reduction, **checkpoint
released**. SWE-RM (2512.21919) did 400K multi-scaffold trajectories;
2607.05391's "Agent Harness Generalization" appendix answers the
cross-harness question (transfers fine, at least training-free). Residual:
the trained-critic scaffold-transfer off-diagonal — a 5 GPU-h kill-arm
(score our Claude Code/Codex traces with their released critic; <2–3 pt gap
on Best@8 = nothing there). ½★.

## Convergence verdict

**No new flagship.** The wave's best survivor (M3-narrowed, ★★½) sits well
below both preregistered flagships (★★★★): [[Prereg-Crop-Consistency-Distillation]]
(ICLR) and [[Prereg-Epistemic-Contextualization]] (ICML) **stand as the
method slate**, and Gate A's neighborhood read *strengthens* the crop
flagship's positioning.

Dispositions:
1. **M3-narrowed → bench, first alternate.** If a third method lane is
   wanted: pilot = operator battery {verbatim, wipe, summarize (their PDR),
   redact, counterfactual-rewrite} × entropy-vs-similarity diagnostics on
   one long-horizon agentic benchmark, vs the blind-resample kill-arm at
   matched tokens. Cheap (API + small open models), training-free, fits our
   controls-first strengths. Mandatory 48h re-gate first.
2. **Bench notes (recorded, not scheduled):** S-M2 hard-narrowed (σ_seed
   gate first); S-M1 three-way interaction (underpowered at our budget);
   S-M3 5-GPU-h kill-arm; M4 merged-CLIP compositional measurement note.
3. **Flagship additions from Gate A:** LABCLIP (2502.03566) and DCSM become
   named baselines / A5 instantiations in the crop prereg (edit logged
   there).

## Process lessons

1. **Visibility predicts scoop speed.** All five candidates rooted in famous
   artifacts or named researcher agendas died at Level 1–2 to papers from
   the last 3–15 months; the sole survivor came from an obscure seam of our
   own measurements. Person-level "unclaimed" checks are insufficient in
   fast lanes — S-M3 was claimed by the very group the opening was
   attributed to, five months before the scan.
2. **Single-pass novelty gates are unreliable** — Gate C's first pass
   missed AlignMerge and would have green-lit a scooped method; the
   verification sweep caught it. Keep the two-pass structure mandatory.
3. **Template exhaustion is a kill class:** "diagnosis: optimization
   preserves loss but destroys latent property X + preservation regularizer"
   is published for safety, calibration, invariance, unlearning — any new X
   is a property swap, not a method.
4. **Infrastructure gap, fix before the next gate:** every agent lost
   OpenReview (`openreview-py` not installed) and DBLP (connection resets),
   so under-review ICLR/NeurIPS work is systematically invisible — exactly
   where concurrent scoops sit. `pip install --user openreview-py` on Anvil.

## Related

[[Method-Gates-2026-08]] · [[Top-Researcher-Scan-2026-08]] ·
[[Prereg-Crop-Consistency-Distillation]] · [[Prereg-Epistemic-Contextualization]] ·
[[Direction-Reevaluation-2026-08]] · [[Status-And-Survivors]]

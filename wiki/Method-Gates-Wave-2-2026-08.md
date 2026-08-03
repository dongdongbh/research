# Method Gates — Wave 2, 2026-08-02

This was the second round of brainstorming and checking whether recent work
already did each idea. The source material was our own evidence: the SVIB
post-mortem, sweep numbers, and collapse/diversity work. We also used the method
directions in [[Top-Researcher-Scan-2026-08]], as the PI requested.

We brainstormed twelve ideas and took three more from the scan. Knowledge we
already had ruled out seven. **Seven candidates went through live checks run by
seven parallel Opus agents.** Each agent searched several sources, including a
required search from ≥2026-05-01 and a web search of the last 8 weeks. Each
agent closely read 2–6 full papers and saved exact quoted evidence. Overlap was
scored on four parts, using the worst-case level.

**What counts as a method (owner definition, updated 2026-08-02).** An idea can
qualify in three ways: (1) a new mechanism, such as an architecture, training
procedure, representation, changed assumption, data source, or workflow
([diffusion](https://en.wikipedia.org/wiki/Diffusion_model), [VGGT](https://arxiv.org/abs/2503.11651), [D4RT](https://arxiv.org/abs/2512.08924), [3DGS](https://arxiv.org/abs/2308.04079), Dreamer); (2) an
improvement to a current method; or (3) **an old method used on a new problem**.
A diagnosis, a test that removes or changes one part, or statistics alone does
not qualify. An idea based on statistics qualifies only if the final result is
a working mechanism that we release.

## Ideas ruled out without a new search

| Idea | Why it was ruled out |
|---|---|
| M6 generative-teacher distillation | Already included as one teacher arm in the crop-distillation main project |
| M7 uncertainty-routed two-tier ITM | It qualifies through route 3, but the transfer is too familiar; include it as an inference-policy arm in the main project |
| M8 stage-transition smoothing | Already done by [2607.00634](https://arxiv.org/abs/2607.00634), found during the replay check |
| M9 granular-nonlinearity block | Wrong scale for us: this is a pretraining-scale architecture race |
| M10 turn-aware KV eviction | Engineering under the owner's definition; [MemDecay](https://arxiv.org/abs/2607.10582) already owns the observation; wait for D. Chen's audit |
| M11 diversity-preserving post-training | Already studied heavily, with 15+ existing projects; wait for the collapse-mechanism diagnosis |
| M12 format-anchored RL | Too early; first run the experiment that decides between the collapse explanations |

From the researcher scan, S-M4 is already the main ICML project. S-M5 and
S-M6 still wait on a diagnosis, as the scan requires. S-M7 becomes part of the
E line of work.

## Results of the live checks

| # | Candidate | Verdict | Lvl | Closest prior | ★ |
|---|---|---|---|---|---|
| M1 | Energy/predictive scoring head (frozen VLM) | **SCOOPED** | 2 | [DCSM (ICCV 2025)](https://arxiv.org/abs/2503.08723); [2604.11496](https://arxiv.org/abs/2604.11496) | 0 |
| M2 | Binding-register adapter tokens | **SCOOPED** | 1 | [SGVL (EMNLP 2023)](https://arxiv.org/abs/2305.06343); [BLIP-2](https://arxiv.org/abs/2301.12597) Q-Former mechanism match | 0 |
| M3 | Fork-preserving context construction | **SURVIVES-NARROWED** | 3 | [2607.26117](https://arxiv.org/abs/2607.26117) (5 days old); DeepMind [2604.05868](https://arxiv.org/abs/2604.05868) | ★★½ |
| M4 | Compositionality-preserving merging maps | **SCOOPED** | 2 | AlignMerge [2512.16245](https://arxiv.org/abs/2512.16245) | ★ |
| M5 | Active-view spatial pretraining | **SCOOPED** | 1 | World2VLM [2604.26934](https://arxiv.org/abs/2604.26934); SIMS-V [2511.04668](https://arxiv.org/abs/2511.04668) | ½ |
| S-M1 | From-scratch KV recipe @ matched bytes | **SCOOPED** | 2 | CLA [2405.12981](https://arxiv.org/abs/2405.12981) (NeurIPS 2024); MixAttention | ★ |
| S-M2 | FDR-controlled accept rule (autoresearch) | **SCOOPED** | 2 | PACE [2606.08106](https://arxiv.org/abs/2606.08106) (Jun 2026) | ★½ |
| S-M3 | Cross-scaffold agent critic | **SCOOPED** | 2 | Neubig's own [2603.03800](https://arxiv.org/abs/2603.03800) (Mar 2026) | ½ |

## Short record of each check

**M1/M2 — binding heads on a frozen encoder.** [DCSM (ICCV 2025)](https://arxiv.org/abs/2503.08723) already
uses a learned 1× readout on frozen CLIP. M1's only real difference was avoiding
[DiffusionITM](https://arxiv.org/abs/2305.16397)'s 17× cost: 68 minutes instead of 4 minutes by its own numbers.
DCSM already did that. M2 exactly matches the [BLIP-2](https://arxiv.org/abs/2301.12597) Q-Former ITC path: 32
query tokens on a frozen ViT, maximum-over-query scoring, and hard-negative ITM.
We verified this, and BLIP-2 has no compositional evaluation. Combined with
[SGVL](https://arxiv.org/abs/2305.06343), this makes M2 a Level-1 overlap.

The search did find useful comparisons for the main crop project. "LABCLIP" is
really **[2502.03566](https://arxiv.org/abs/2502.03566) (ICLR 2026)**. It uses a D×D text-side matrix, frozen
encoders, [ARO](https://arxiv.org/abs/2210.01936), and [SugarCrepe](https://arxiv.org/abs/2306.14610), so it is a required baseline. The whole
group of nearby methods—DCSM, LABCLIP, [ABE-CLIP](https://arxiv.org/abs/2512.17178), and TF-Local—only reads
features that already exist in the frozen model. None teaches the 1× patch path
with a stronger teacher. That is exactly our main project's mechanism.
[FILIP](https://arxiv.org/abs/2111.07783), [SPARC](https://arxiv.org/abs/2401.09865), and [DINOSAUR](https://arxiv.org/abs/2209.14860) become citations only because they have no
compositional evaluation and are not heads on frozen models.

**M3 — keep useful context after a failed attempt (the survivor).** Other work
already published the main framing and finding. [2607.26117](https://arxiv.org/abs/2607.26117) shows that, below 7B,
starting a blind new sample beats self-repair while using 2.5–5.5× fewer tokens.
It also finds that 33–68% of retries repeat the failed program because of
anchoring. DeepMind [2604.05868](https://arxiv.org/abs/2604.05868) says the weakness of a sequence of attempts is
lost exploration, and that anchoring is **correctness-agnostic**, meaning it can
happen after right or wrong attempts. [2605.08563](https://arxiv.org/abs/2605.08563) gives a contamination model and a theorem
that favors a clean restart. Meta [2604.16529](https://arxiv.org/abs/2604.16529) released a summarize operation on
[SWE-bench](https://arxiv.org/abs/2310.06770) and Terminal-Bench.

**The exact unanswered question:** can an operation keep useful information
from a failure while removing *anchoring*, the tendency to repeat that failure?
Possible operations include redacting or rewriting the failure as a
counterfactual. These go beyond simply keeping or deleting everything. Choose
the operation using an entropy measurement at the tokens where paths split.
The likely useful setting is a long agent task where deleting everything also
deletes costly knowledge about the environment. It is **not** short math or code
retry, where blind resampling is proven to win. The comparison that can end the
idea is a blind resample with the same token budget. The required control
applies the same operations to *successful* attempts, because anchoring does not
depend on correctness. Nearby papers have appeared at about one per month since
May, so run a **48h re-check before any commitment.** ★★½.

**M4 — merging maps that preserve compositional skill.** AlignMerge
([2512.16245](https://arxiv.org/abs/2512.16245)) uses the same pattern with a different property: "merging can preserve
loss while quietly destroying alignment," followed by a regularizer that keeps
that hidden alignment during merging. [Representation Surgery](https://arxiv.org/abs/2402.02705) (ICML 2024/25)
is the ICML example. Tam et al. [2409.18314](https://arxiv.org/abs/2409.18314) already uses the title. Even the
basic claim is disputed: [2509.19476](https://arxiv.org/abs/2509.19476) and [Mai (CVPR 2026)](https://arxiv.org/abs/2603.12433) find that
merged or stitched models can *beat* their parents on structural skill. What
remains is that nobody runs compositional tests on merged CLIPs. That supports
at most a one-week **measurement** note, not a method paper. The first search
missed AlignMerge; the required second search found it.

**M5 — spatial training with active views.** World2VLM ([2604.26934](https://arxiv.org/abs/2604.26934)) already does
action-conditioned view-transition mid-training and tests zero-shot transfer on
SAT, VSI-Bench, and [MindCube](https://arxiv.org/abs/2506.21458). SIMS-V ([2511.04668](https://arxiv.org/abs/2511.04668)), from the [VSI-Bench](https://arxiv.org/abs/2412.14171)
authors, already studies simulated-data-to-real transfer; 25K examples beat a
72B baseline. [Spatial-SSRL](https://arxiv.org/abs/2510.27606) already uses non-QA predictive objectives. No
unanswered part remains. Another warning is that [2511.16655](https://arxiv.org/abs/2511.16655) shows the spatial
"supersensing" tests can be gamed. Abandon this idea.

**S-M1 — KV-cache recipe.** Matching KV bytes per token is already CLA's Figure
3 axis (NeurIPS 2024). Its from-scratch 1B/3B grid already contains the result
that overturns the expected MQA-vs-GQA ranking: [MQA](https://arxiv.org/abs/1911.02150) ties [GQA-2](https://arxiv.org/abs/2305.13245) while using half
the bytes. It reaches the proposed answer, "CLA2 + MQA." MixAttention
([2409.15012](https://arxiv.org/abs/2409.15012)) already published Character.AI's local:global × cross-layer recipe
with [RULER-32K](https://arxiv.org/abs/2404.06654). What remains is the three-part interaction among heads,
tying, and local:global attention at long context. We could afford it only at
300M, which is too weak for the interesting 0.03–0.06-perplexity effects.
[Gemma 4](https://arxiv.org/abs/2607.02770) already ships axes 1+2 as its default. ★.

**S-M2 — rule for accepting a result.** PACE ([2606.08106](https://arxiv.org/abs/2606.08106)) is the same paper. It
calls the acceptor the hidden weak point, says greedy acceptance means "the
agent p-hacks itself," uses an anytime-valid e-process commit rule, and already
tests an online-FDR baseline, which was too cautious. autoresearch PR #560 is
the current repository-level answer, though it is unmerged and uses a 2σ noise
floor. Rehearse ([2607.27687](https://arxiv.org/abs/2607.27687)) owns the [nanochat](https://github.com/karpathy/nanochat) testbed with 4,000 runs.

What remains is to compare k random-seed runs using one score per run while
accounting for GPU-hour cost, with a **run-level** false-discovery-rate (FDR)
promise. PACE's promise is per decision and uses paired outcomes for individual
items. ★½ can become ★★ only after this strong narrowing. Before considering the
full idea, spend ~20 GPU-h measuring variation across seeds (σ_seed) on
nanochat speedruns.

**S-M3 — a model that grades an agent.** Neubig's group already released this:
[2603.03800](https://arxiv.org/abs/2603.03800), a rubric-supervised 4B critic trained on 154K private [OpenHands](https://arxiv.org/abs/2407.16741)
production segments. It improves Best@8 by +15.9, cuts compute by 83% through
early stopping, and has a **released checkpoint**. SWE-RM ([2512.21919](https://arxiv.org/abs/2512.21919)) used 400K
trajectories from several scaffolds. The "Agent Harness Generalization"
appendix in [2607.05391](https://arxiv.org/abs/2607.05391) answers the cross-harness question: transfer works, at
least without training.

What remains is the off-diagonal transfer of a trained critic between
scaffolds. A 5 GPU-h comparison can end the idea: score our Claude Code/Codex
traces with their released critic. If Best@8 differs by <2–3 points, there is
nothing left. ½★.

## Final decision for this round

**No new main project.** The best survivor, narrowed M3 at ★★½, is much weaker
than the two pre-registered main projects at ★★★★:
[[Prereg-Crop-Consistency-Distillation]] for ICLR and
[[Prereg-Epistemic-Contextualization]] for ICML. Those remain the method plan.
The M1/M2 search also makes the crop project's position stronger.

Next steps:

1. **M3-narrowed → idea bench, first backup.** If we want a third method line,
   first test {verbatim, wipe, summarize (their PDR), redact,
   counterfactual-rewrite} × {entropy, similarity} measurements on one long
   agent benchmark. Compare against blind resampling with the same tokens. The
   pilot is cheap, needs no training, and fits our habit of testing controls
   first. A 48h re-check is required before starting.
2. **Idea-bench notes, recorded but not scheduled:** strongly narrowed S-M2,
   with the σ_seed check first; S-M1's three-part interaction, which is too weak
   at our budget; S-M3's 5-GPU-h deciding comparison; and M4's compositional
   measurement note for merged CLIP.
3. **Additions to the main crop project from Gate A:** LABCLIP
   ([2502.03566](https://arxiv.org/abs/2502.03566)) and [DCSM](https://arxiv.org/abs/2503.08723) become named baselines / A5 versions in the crop
   pre-registration. That page records the edit.

## Process lessons

1. **Highly visible ideas are claimed faster.** All five ideas taken from
   famous tools or named researcher plans were already done at Level 1–2 by
   papers from the last 3–15 months. The only survivor came from an obscure
   gap in our own measurements. Checking whether one famous person has claimed
   an idea is not enough in a fast area. S-M3 had been claimed five months
   earlier by the same group whose work suggested the opening.
2. **A single check for earlier work is not reliable.** The first search for
   Gate C missed AlignMerge and would have approved an already-done method. The
   second search found it. Keep the two-search rule.
3. **A reused paper pattern can rule out an idea.** The pattern "optimization
   keeps loss low but destroys hidden property X, so add a regularizer to keep
   X" already exists for safety, calibration, invariance, and unlearning.
   Swapping in a new X does not create a new method.
4. **Fix this tool problem before the next search.** Every agent lost access to
   OpenReview because `openreview-py` was not installed, and DBLP connections
   reset. This hides work under review at ICLR and NeurIPS, where competing work
   is most likely. Run `pip install --user openreview-py` on Anvil.

## Related

[[Method-Gates-2026-08]] · [[Top-Researcher-Scan-2026-08]] ·
[[Prereg-Crop-Consistency-Distillation]] · [[Prereg-Epistemic-Contextualization]] ·
[[Direction-Reevaluation-2026-08]] · [[Status-And-Survivors]]

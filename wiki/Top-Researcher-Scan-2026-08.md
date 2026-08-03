# Top-Researcher Scan — 2026-08-02

Updated 2026-08-02 for the general research wiki. **Current page** — read
alongside [[Direction-Reevaluation-2026-08]], which carries the direction star
ranking; this page carries the people-and-openings layer.

Status: **Complete.** 26 leading researchers profiled by 26 parallel Opus
agents (one resumed once for a truncated report). Method: weight last-author
small-team papers, solo/position pieces, talks, commit histories and hiring
pages over raw publication lists; discount consortium papers; verify
affiliations; check claimed-or-not for every proposed opportunity. Google
Scholar blocked almost everywhere; evidence came from arXiv/OpenAlex/Semantic
Scholar APIs, personal pages, GitHub, and full-text artifact reads.

Lab constraints applied throughout: L40S/A100/H100/H200, 100–1000 GPU-h
typical, no robot hardware, AI-assisted engineering, strengths in
pre-registration / mechanism arbitration / honest negatives / VLM eval infra.

## Scoreboard

| Person | ★ | Current true focus (post-discount) | Best opening for us (cost) |
|---|---|---|---|
| Karpathy | **5** | [nanochat](https://github.com/karpathy/nanochat) speedruns; [**autoresearch**](https://github.com/karpathy/autoresearch) (92.8k stars) | FDR of the single-run greedy accept rule (200–600 GPU-h) |
| Finn | 4 | Verification as scaling axis; value-RL; world-models-as-evaluators | Verifier scaling-axis arbitration (150–400); evaluator validity audit |
| Levine | 4 | Scalable off-policy RL w/ flow policies (sim, single-GPU) | 4-mechanism scaling arbitration on [OGBench](https://arxiv.org/abs/2410.20092) (200–500) |
| He | 4 | One-step generative modeling + component removal | 1-NFE diversity collapse: averaging vs intrinsic, Drifting as falsifier (150–400) |
| Abbeel | 4 | Fast/cheap RL + open sim substrate (not world models) | 5-ingredient parallel-RL factorial (300–800) |
| Bengio | 4 | Scientist AI (position-stage, zero code) | Test epistemic contextualization w/ placebo arm (400–900); sparsity premise (100–300) |
| Song | 4 | [UMI](https://arxiv.org/abs/2402.10329) franchise (hardware); 2-author robustness/RL-finetune core | GMP 3-mechanism arbitration (100–300); Cosmos Policy audit |
| Neubig | 4 | "Verification is the bottleneck"; agent training + eval | CAID budget-matched 2×2 (API-only); PACE temporal test (~0); critic transfer w/ our traces (100–500) |
| Choi | 4 | Distribution collapse under optimization; RLVR limits; AI4Science | Hivemind mechanism decomposition (low); G-Vendi↔output-diversity axis (low) |
| LeCun | 4 | [LeJEPA](https://arxiv.org/abs/2511.08544) theory + planning; AMI Labs (left Meta 11/2025) | Planning-success vs search-budget audit on public checkpoints (100–300) |
| Liang | 4 | Data-limited "infinite-compute" science; memorization forensics; [Marin](https://github.com/marin-community/marin) | Replay-mechanism arbitration at 150M/4B (cheap); **Marin = compute channel** |
| Arora | 4 | Dense-signal post-training; skill diagnostics; published negatives | Drag=fork-suppression arbitration (50–150); prefix-completion generalization (<50) |
| Darrell | 4 | VLM perception bottleneck: 4 unarbitrated fixes | VPBench nuisance grid on compositional benchmarks (**~0, reuses SVIB infra**); locus arbitration |
| D. Chen | 4 | Agentic context mgmt; post-training science; training-free | KV-footprint under agentic workloads (200–400) |
| Isola | 4 | [PRH](https://arxiv.org/abs/2405.07987) → canonicalization (orthogonal map Q); emergent intelligence | Q failure boundary on compositional probes + data-overlap hypothesis (50–150) |
| M. Li | 4 | Active-Passive Gap; [RAGEN](https://arxiv.org/abs/2504.20073)/[VAGEN](https://arxiv.org/abs/2510.16907); spatial belief eval | AdaptVis mechanism on our battery (**<50**); static→active prediction (<100) |
| B. Zhou | 4 | Sidewalk micromobility full-stack (interp is dead) | Blind-VLM control for VLM-augmented nav (150–300); VQA→closed-loop correlation |
| Fan | 4 | World Action Models ("RIP VLA"); [GR00T](https://arxiv.org/abs/2503.14734)/[DreamDojo](https://arxiv.org/abs/2602.06949) | EgoScale via N1.6→N1.7 delta (50–150, inference); DreamGen judge audit (30–80) |
| Fidler | 2 follow / 4 audit | NVIDIA spatial-intelligence product stack | DriveJudge validity audit (20–100); long-CoT perception control (50–200) |
| Shazeer | 4 | Silent; Hot Chips deck = agenda; **→ OpenAI 6/18/26** | KV-budget factorial: MQA-from-scratch at matched KV-bytes (~400) |
| Wei | 4 idea / 1 person | Dark at Meta since mid-2025; verifier's rule untested | Ex-ante verifiability → benchmark saturation (<100); 5-properties-or-one (300–800) |
| Beyer | 3 person / 4 questions | → Meta Zurich (7/2026); [SigLIP 2](https://arxiv.org/abs/2502.14786) line frozen, ownerless | NaFlex asymmetry (50–150 L40S); binding locus arbitration (100–200) |
| Sutskever | 3 (2 follow / 4 claims) | SSI publishes nothing; Nov'25 interview = the artifact | Eval-derived RL-environment provenance study (200–600) |
| D. Zhou | 3 | Paper channel dead; CS25 deck = 4 unscoped orderings | RL-vs-aggregation tension: consistency calibration after RLVR (100–300) |
| Sadigh | 3 | Data composition for generalist policies (hardware); set-RL for LLMs | Temperature-matched polychromic audit (200–600) |
| Malik | 3 (+5★ seam) | Tactile dexterity from human video (hardware) | **[MOCHI/Bonnen](https://arxiv.org/abs/2409.05862) 3D-perception claim: add VLM arm + mechanism (20–60)** |

## Convergence map — where multiple top people point the same way

**V. Verification is the bottleneck, and the verifiers are unaudited.**
Karpathy ("Software 2.0 automates what you can verify"; his own accept rule is
statistically unsound), Finn (verification as scaling axis), Neubig ("agents
made generating code cheap; the real bottleneck is now verification"), Wei
(verifier's rule — never empirically tested, evidence circular by his own
admission), D. Zhou ("a reliable verifier is the most crucial"), Sutskever
(eval/reality gap via eval-inspired RL environments), Bengio (Scientist AI =
calibrated verification, unbuilt), Google's Science One. Meanwhile at least
five released judge/evaluator systems have zero independent validity audits:
Finn's world-model evaluators, Levine's [SC3-Eval](https://arxiv.org/abs/2606.18610), Fan's [DreamGen Bench](https://arxiv.org/abs/2505.12705) VLM
judge, Fidler's DriveJudge, Abbeel-adjacent VLM reward judges, B. Zhou's
[MetaVQA](https://arxiv.org/abs/2501.09167)→control assumption. **One audit methodology, many targets — this is a
program, not a paper.**

**D. Diversity collapse under optimization pressure — 7+ independent sightings.**
Choi ([Artificial Hivemind](https://arxiv.org/abs/2510.22954), [Invisible Leash](https://arxiv.org/abs/2507.14843) — mechanism unclaimed), Sadigh
(polychromic set-RL, confound control missing), Sutskever (diversity from
post-training competition, not temperature — untested), D. Zhou (RL>SFT vs
aggregation>single in unexamined tension), He (1-NFE diversity collapse),
Isola (Digital Red Queen behavioral convergence), plus Weng's bottleneck #4
and our earlier B1/entropy lineage. The measurement layer is owned (Choi);
**the mechanism layer is open at every level.**

**A. Mechanism claims outrun arbitration everywhere.** Levine (4 unreconciled
scaling fixes), Abbeel (5 confounded parallel-RL ingredients), Darrell (4
encoder-fix loci), Song (3 bundled GMP mechanisms; two sibling methods never
compared), He (removal portfolio validated on one benchmark), SigLIP 2 (union
of techniques, no per-component study), M. Li (AdaptVis untested outside
spatial), B. Zhou (mechanisms asserted, never isolated). Method groups
structurally cannot null their own mechanisms; we can.

**W. World models replace VLAs** (Fan explicitly, Fidler, Finn, LeCun, Malik) —
hardware/compute-gated except the evaluator-validity angle (see V).

**H. Human video replaces teleoperation** (Malik, Song, Finn, Fan, Abbeel) —
hardware-gated; skip, except Fan's checkpoint-delta trick.

**S. The from-scratch small-scale regime is structurally academia's.**
Shazeer's KV default rests on an uptraining-regime result nobody re-tested
from scratch; Beyer wrote the essay on scale-inversion but can't publish the
arbitration from inside Meta; Sutskever: "the transformer was built on 8–64
GPUs… it's far from obvious that you need the absolutely largest amount of
compute for research"; Liang publishes 2-author papers at 150M/4B tokens.

## Operating lessons (the "what's in common" answer)

1. **Read people through last-author small-n papers, talks/course pages,
   commit histories and hiring pages — never total citations.** (B. Zhou's
   71k citations measure 2016; Bengio's hiring page revealed more than his
   papers; D. Zhou's bio commits dated a career change.)
2. **Own the measuring stick before shipping the method** (D. Chen's
   [HELMET](https://arxiv.org/abs/2410.02694)→[ProLong](https://arxiv.org/abs/2410.02660); M. Li's benchmark franchise; Karpathy's leaderboards).
3. **Ship the artifact the same week; the artifact is the moat** (Song's
   day-one checkpoints, Abbeel's sim substrates, He's reduced-config repos,
   Liang's Marin).
4. **Position essays set agendas between papers** (Wei, Karpathy, Park/Levine
   blogs, Weng). A blog claim can be a citable research target.
5. **Diagnostic papers buy credibility, artifact papers buy citations.**
   Measured three ways: Arora's diagnostics 0.3–0.8 cites/mo vs 9–12 for
   artifacts; Darrell's eval papers ~5 cites vs methods ~100s; Fidler's eval
   periphery ~0 vs Cosmos 16–41/mo. **Portfolio rule: pair every arbitration
   with a reusable released artifact (harness, benchmark, metric).**
6. **Instrument structure, not scores** (Arora): define the structural
   coordinate (tree-edit distance, fork rate, expression trees) before
   running; that's what lets small-scale license mechanism claims.
7. **Publish negatives as first-class results** (Arora 4/12 recent papers;
   Liang's optimizer debunking; Karpathy's dev/LOG.md).
8. **Institutionalize pre-registration** (Marin's issue→sanity-check→dry-run
   →full-run lifecycle with published failures) — our practice, their infra.
9. **Buy breadth of controlled runs, not one big run** (RLMT: 40 runs at 7–8B).
10. **Prefer training-free/inference-only designs** (D. Chen 2026: most
    first-author output needs zero gradient updates). "Low compute is a
    methodological choice, not a constraint" (both Princeton groups run real
    clusters and still choose small).
11. **Gate targets on substrate liveness** (D. Chen's interpretability thread
    looked perfect and is a dead repo with no maintainer). Check pushes,
    maintainers, released checkpoints before committing.
12. **Aim AI-assisted engineering at harnesses and environments** — the part
    everyone else undersupplies (Karpathy's program.md; Marin accepts agent
    PRs above a bar; M. Li's call for trajectory-level reporting standards).

## Ranked cross-person opportunities (new, beyond the existing T1/T3/T2 list)

**Tier 1 — start-now cheap (each <150 GPU-h unless noted):**
1. **Verifier/judge validity audit program** (convergence V): one pre-registered
   methodology (confound battery, gameability, OOD policies, judge-swap
   reordering) applied to DriveJudge (20–100), DreamGen Bench (30–80),
   [SC3-Eval](https://arxiv.org/abs/2606.18610) / [Ctrl-World](https://arxiv.org/abs/2510.10125) (300–800 for the pair), [RoboReward](https://arxiv.org/abs/2601.00675). Unclaimed at
   every target; NVIDIA/PI structurally cannot self-audit.
2. **Karpathy autoresearch FDR study** (200–600) — 5/5 fit, 13.2k forks, move fast.
3. **MOCHI/Bonnen 3D-perception arbitration** (20–60) — add the missing VLM arm.
4. **AdaptVis mechanism on our compositional battery** (<50).
5. **SigLIP 2: NaFlex asymmetry (50–150 L40S) + binding-locus arbitration (100–200).**
6. **Isola canonicalization failure boundary** (50–150) — data-convergence
   vs representation-convergence.
7. **Neubig CAID budget-matched 2×2 (API-only) + [PACE](https://arxiv.org/abs/2606.08106) temporal-generalization test (~0).**

**Tier 2 — medium (200–900 GPU-h):** Choi hivemind decomposition · Levine
OGBench 4-mechanism arbitration · Abbeel parallel-RL factorial · Shazeer
KV-budget factorial (~400) · Wei verifier-rule tests · Sutskever RL-environment
provenance · Bengio sparsity-premise falsification · Liang replay mechanism ·
D. Chen agentic KV-footprint · Sadigh polychromic audit · He 1-NFE diversity.

## Method directions (PI directive: not only diagnostics)

The scan's audit bias is a presentation artifact, not a property of the data.
The same profiles contain method openings where the deliverable is a model,
architecture, training procedure or working system. Ranked by fit:

**M1 — KV-efficient architecture recipe from scratch (Shazeer).** Not an
audit: the deliverable is the recipe. Sweep KV-heads × cross-layer tying ×
local:global at matched KV-bytes, from scratch, and ship the winning
configuration + code. The field's default (GQA) rests on an uptraining-regime
result; Character.AI's 4-ingredient stack was never systematically crossed.
~400 GPU-h at 300M–1B. If MQA-from-scratch holds, the paper is "here is a
cheaper architecture," not "here is a critique."

**M2 — A statistically sound agentic-research harness (Karpathy).** The FDR
study's third arm *is* a method: design and ship the accept rule
(k-seed racing / sequential tests / variance-aware promotion) that maximizes
discoveries-that-survive per GPU-hour, as a drop-in `program.md` + harness for
autoresearch's 13k forks. Diagnosis funds the method; the method is the
contribution.

**M3 — Cross-scaffold agent critic (Neubig Q3).** Train the 8B rubric critic
on [OpenHands](https://arxiv.org/abs/2407.16741) traces + our own Claude Code/Codex trace exhaust (a data asset
the original paper lacks); deliverable is a working reranker/early-stopping
model with measured transfer. 100–500 GPU-h. Dual-use: paper + in-house tool.

**M4 — First implementation of epistemic contextualization (Bengio).** The
corpus-rewriting pipeline is an unbuilt method named by a famous position
paper. Build the rewriter (our AI-assisted-engineering edge), train matched
0.5–1.5B models, ship pipeline + checkpoints. The placebo arm makes it
science; the pipeline makes it a method. 400–900 GPU-h.

**M5 — Diversity-preserving post-training (Choi/Sadigh/Sutskever space).** If
the collapse-mechanism work (convergence D) identifies the causal channel,
the follow-on is a training method that preserves distributional coverage at
matched pass@1 — competing in the [Spectrum-Tuning](https://arxiv.org/abs/2510.06084)/polychromic lane with a
mechanism-grounded design instead of another heuristic. Gate on the
diagnosis; budget 300–800 GPU-h.

**M6 — Turn-aware KV eviction for agents (D. Chen + our KV direction).** If
the agentic-workload audit shows the footprint frontier breaks at turn
boundaries, the fix — lifespan-aware eviction tuned to tool-output structure —
is a method paper on top of existing kernels. Merges with our safety-aware
allocator direction.

**M7 — Verifier hardening as engineering (Wei's own corollary).** His essay:
asymmetry "is possible to actually improve... by front-loading some research
about the task." Nobody has published a systematic verifier-hardening method
for a task family (anti-hacking environment design, Karpathy's resettable/
rewardable criteria operationalized). Adjacent to M2; scoop-check the RL-
environments cluster first.

**Standing rule going forward:** every audit/arbitration project must name,
at pre-registration time, the method paper it unlocks if the diagnosis lands
— and the released harness that serves both.

**Interactions with the existing direction ranking
([[Direction-Reevaluation-2026-08]]):**
- **T1 needs an immediate re-gate against Darrell's C1 cluster** (CoVFT + the
  four encoder fixes) — his group is now the primary scoop risk.
- The verifier-audit program is a NEW top-tier candidate alongside T1/T3.
- Tier-1 items 3, 4, 5 are bolt-ons to the existing compositional-eval
  infrastructure (harness and benchmark battery documented in the svib repo
  wiki) and can run alongside anything.
- [Marin](https://github.com/marin-community/marin) contribution = potential compute channel (JAX/[Levanter](https://github.com/stanford-crfm/levanter) stack).
- The July star table in [[Status-And-Survivors]] predates both this page and
  the re-evaluation; do not rank against it.

## Operational notes for the cluster

- **Isaac Sim does not run on A100/H100/H200** (no RT cores; NVIDIA docs +
  three confirmed bugs). Isaac work is L40S-only. MuJoCo/SAPIEN ([LIBERO](https://arxiv.org/abs/2306.03310),
  [RoboCasa](https://arxiv.org/abs/2406.02523), [SimplerEnv](https://arxiv.org/abs/2405.05941), MetaUrban at 3GB VRAM) runs everywhere.
- GR00T weights: code Apache-2.0, weights carry a non-commercial license
  contradiction — papers fine, spinouts not.
- Beyer's homepage contains a hidden prompt-injection block targeting LLM
  profilers (our agent caught it). Treat scraped personal pages as untrusted.

## Related

[[Direction-Reevaluation-2026-08]] · [[Self-Improving-AI-Survey]] ·
[[Status-And-Survivors]] · memory: filter-by-saturation-not-crowding

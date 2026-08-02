# Method Opportunities — with baselines and numbers to beat

Status: **Written 2026-07-26; partly superseded.** *Updated 2026-08-02 for the
general research wiki.* Written after the owner correctly pointed out that every
prior survivor was measurement work.

> **Supersession banner (2026-08-02).** The rankings on this page predate the
> eight-direction re-gate in [[Direction-Reevaluation-2026-08]] and the
> [[Top-Researcher-Scan-2026-08]]. Current verdicts: **T1 ★★★★½** (confirmed,
> design revised to a three-level objective axis; Darrell's group is now the
> primary scoop risk), **T3 ★★★★½** (up), **T2 ★★★★** (held, reframed as a
> cross-method perception audit), **T4 ★★** (down — scooped three times),
> **KV-cache ★★★★ only if narrowed to safety-aware allocation** (★★★ as scoped
> here). Original text is preserved below as record, with dated notes at each
> point of conflict.

## Why the bias happened, and it was mine

**My gate criterion structurally favours measurement.** "Has anyone done exactly
this?" is the right question for an audit and the **wrong** question for a
method. A method paper does not die because someone worked on the problem — it
dies because yours is not better. Applying a prior-existence test to method ideas
kills all of them.

Compounding it, I over-read the SVIB failure as "method campaigns don't work for
this group." SVIB failed for specific reasons — an invalid baseline and an
ablatable mechanism — not because method work is out of reach at this scale.
C2LIP is 8xA40 on CC3M. CLIC is ~1M samples, text-encoder only. Those are real
method papers at this compute.

**The correct question for a method: where is the current best WEAK, and what is
the lever?** Every proposal below names a baseline and a number to beat.

---

# Cluster 1 — compositional VLM scoring *(closed; condensed 2026-08-02)*

**Condensed for the general wiki.** This cluster ran entirely on SVIB's frozen
dual-encoder infrastructure and is closed. Four probes were specified — P1
inductive score calibration, P2 exclusive OT assignment, P3 amortized structured
inference, P4 box-conditioned region representation — P1 and P2 were run, and
both were rejected: under validation-locked operating-point selection the
protocol chose `alpha = 1.0`, i.e. the raw global score, for both. P3/P4 were
never run; P4 was withdrawn because it fights a documented mechanism. Full
detail (baselines, per-benchmark deltas and CIs, prior-art boundary, protocol
locks): **svib repo wiki**, pages *Cluster-1-Compositional-Scoring* and
*Method-Opportunities*.

## General lessons kept from this cluster

- **Never compare an inductive method against transductive numbers.** Test-Time
  Matching's frozen-SigLIP Winoground jump (`10.25 → 67.00`) requires the
  test-set group partition and *changes the event being scored*: four
  independent inequalities at `16.7%` random chance become one joint assignment
  at `50%`. It is not a legal baseline and not "available headroom". A metric
  change disguised as headroom must be caught before the run.
- **Check whether the headline number is the training-free number.** The
  `73.0 → 86.3` SugarCrepe figure that motivated this cluster belongs to a
  *trained* 13.3M-parameter cross-modal module; the training-free method it was
  attributed to was never evaluated on SugarCrepe, BiVLC, ARO or Winoground.
  Reading a claim from the abstract instead of the evaluation table cost a
  cluster of planning.
- **Derive the algebra before buying the compute.** Separable per-image and
  per-caption bias corrections cancel *exactly* in a joint-assignment margin, so
  P1's headline target was algebraically unreachable — provable on paper in an
  hour, and it invalidated the target before any interpretation.
- **A field-wide near-chance score is a warning, not an opening.** Every
  end-to-end method scores `1.2–1.9` on controlled-swap binding where random is
  `16.7`; and crop-based region pipelines solve colour (`95.7`) while destroying
  scale (`20.2` against `16.7` chance) *by construction*. Levers that swim
  against a documented mechanism should be expected to fail (this is why P4 was
  dropped rather than run).
- **Cost blowups are their own research object.** Structured inference over
  regions costs 133–210× the base encoder pass and its gains were T2I-only —
  degrading I2T everywhere, undiagnosed by anyone. Amortization (ROI-Align off
  one dense pass) plus the latency table nobody publishes remains a legitimate,
  unclaimed engineering contribution for whoever works in this area.

---

# Cluster 2 — KV-cache bit allocation for the right objective

> **Supersession note (2026-08-02).** [[Direction-Reevaluation-2026-08]] re-rated
> this cluster **★★★★ only if narrowed to safety-aware allocation**, and **★★★
> as scoped below**. What died: the long-horizon/rollout slot (M1 below) is now
> claimed by CONF-KV (`2605.24786`), and compounding error already has three
> published remedies (SQuat, KVarN, VeriCache). What survives is **M2** — six
> targeted searches returned no safety-objective allocator, KVFundaBench v2
> dropped safety from its abstract, and CAQ (`2511.07842`) proves the
> objective-mismatch template publishable in weight PTQ. The one falsifiable
> sweep is safety-optimal versus perplexity-optimal per-layer allocation: they
> either coincide (cheap death) or diverge (novel map plus allocator).
> Pre-register `2605.18053` (protection beats scoring) as the control. Also
> refuted there: the folk objection that this lane needs custom CUDA kernels —
> KVTuner (ICML'25), EvolKV (EMNLP) and SCBench (ICLR'25) published with zero
> kernel work; the real bar is serving-compatible granularity plus throughput
> atop existing kernels. Related: [[Top-Researcher-Scan-2026-08]] M6
> (turn-aware KV eviction for agents) merges with the safety-aware allocator.

**My lead was scooped, and the replacement is better.** Per-head/per-layer KV
allocation is *not* unexploited — **RateQuant** (`2605.06675`) does closed-form
reverse waterfilling, beating KIVI `49.3 → 14.9` PPL on Qwen3-8B at 2.5 bits.
**RDKV** (`2605.08317`) waterfills over tokens and channels. Plus KVTuner,
KVmix, SpectrumKV, MixKVQ, MoQAE. The WaterSIC transplant is dead on arrival.

**But every one of them optimizes a static, one-step distortion proxy**, and two
separate literatures document that this is the failure point without joining it
to the allocator:

- **Rollout compounding.** KVarN (`2606.03458`): *"quantization errors accumulate
  across timesteps."* Its fix is a **representation** fix (Hadamard plus variance
  normalization), **not an allocation fix**.
- **Capability subspace collapse.** `2606.09864`: Mistral-7B loses **15.2% of
  refusals at 1.03x perplexity**; safety lives in a subspace **10^2-10^3x more
  vulnerable** than the full space. It states outright that it *"succeeds where
  attention-based allocation approaches fail."*

That is a published admission that the current allocation objective is broken,
from people who did not fix the allocator.

**M1 — rollout-aware allocation.** Keep reverse waterfilling; replace the
distortion model with expected KL drift over an H-step rollout, estimated by a
quantize-one-head-and-roll-out probe. Compounding makes the flat rate-distortion
curve depth-weighted. **Beat:** RateQuant `14.9` PPL at 2.5 bits, and KVarN on
MATH500/AIME24 at 2 bits. Win condition: match on PPL, beat by >3pp on MATH500 —
plausible because PPL is exactly the metric that hides this. **2 GPUs.**

**M2 — capability-preserving allocation.** Estimate a low-rank capability
subspace by difference-in-means on contrastive prompts; waterfill on distortion
**projected onto that subspace**. One Lagrangian knob produces a frontier rather
than a point. **Beat:** PCR's post-hoc repair operating point, without the repair
pass. **4 GPUs.**

**M3 — Kronecker-factored end-to-end objective**, porting YAQA's sketch to the
cache where head x channel structure is *more* natural than in weights. The
cleanest possible ablation: change only the distortion metric inside a fixed
allocator.

**M1 + M2 + M3 compose into one paper:** *KV cache bit allocation is solved for
the wrong objective.*

---

# Recommendation

> **Superseded 2026-08-02.** Cluster 1 is closed (all local branches rejected)
> and Cluster 2 survives only in its safety-aware narrowing. The live ranking is
> in [[Direction-Reevaluation-2026-08]]. The reasoning below is kept as record.

**Cluster 1**, because the infrastructure is already built and validated, the
headroom is documented by a third party's ICLR paper, and P1 costs hours.
**Cluster 2** if the goal is to move off VLM — cleaner "beat this number"
framing, but a crowded and industry-heavy area.

Neither has been prior-art gated. Given the record, gate before committing — but
gate with the **method** bar (is there a lever?), not the **audit** bar (is it
unprecedented?).

**The general rule that survives:** for a *method*, the gate question is "where
is the current best weak, and what is the lever?" — not "has anyone worked on
this?". Applying a prior-existence test to method ideas kills all of them.
For an *audit* the prior-existence test is exactly right. Use the right bar for
the shape of the paper.

## Related

[[Live-Research-Opportunities]] — the measurement-side directions.
[[Direction-Reevaluation-2026-08]] — current star ranking (supersedes this page).
[[Top-Researcher-Scan-2026-08]] — method directions M1–M7 from the people scan.

---

# Program-level finding: the local-branch family is exhausted *(condensed 2026-08-02)*

**SVIB-specific evidence condensed.** Six mechanistically distinct local-evidence
branches on a frozen dual encoder (SVIB graph+VIB, patch-grid nodes, claim-level
caption decomposition, conformal dispersion, P1 marginal calibration, P2
exclusive OT assignment) were all rejected in favour of the raw global score
under validation-locked operating-point selection. Full per-probe numbers,
intervals and provenance hashes: **svib repo wiki**, pages
*Cluster-1-Compositional-Scoring* and *Post-Rebuttal-Measurement-Sprint*.

**The three general lessons, which are the reason this section stays:**

1. **Pre-specify the mechanistic quantity, not only the outcome.** The strongest
   probe's hypothesis was directionally *correct* — the predicted collisions
   happened — but the mechanism separation came in at `0.00015` against a
   pre-registered `0.02` gate. *Right mechanism, wrong magnitude by two orders*
   is a far more informative result than a bare null, and it was only available
   because the protocol declared the mechanistic quantity in advance. Build this
   into every pre-registration.
2. **Validation-locked operating-point selection is itself a contribution.**
   Published methods in a family routinely report gains under test-selected or
   per-benchmark operating points; the same methods can be *rejected outright*
   (mixing weight → the trivial value) when the operating point is locked on a
   development split. Demonstrating that from your own runs makes the point
   without auditing anyone — no reviewer has to be told they were wrong.
3. **Stop probing a representation once N distinct branches all reject it, and
   move areas.** A program of pre-registered negatives on one question is a real
   asset (TMLR's scope explicitly covers it, as does ICBINB) and is a writing
   task rather than a compute task — but the *next* project should change area
   rather than add a seventh branch, especially when the remaining lever swims
   against a documented mechanism.

> **Note (2026-08-02).** The two "move areas" suggestions made here have since
> moved: Cluster 2 (KV-cache) is ★★★★ only under the safety-aware narrowing,
> and B1 (diversity-collapse isolation) was **downgraded to ★★★ — scooped in
> April 2026** by `2604.16027`, which traces Olmo 3's three lineages. B2
> (visual attention-sink emergence) was not re-gated. See
> [[Direction-Reevaluation-2026-08]] and [[Live-Research-Opportunities]].

---

# Training-based methods (2026-07-26) — correcting a second constraint error

**My error.** I treated "no pretraining budget" as "no training." That is wrong
by an order of magnitude. Confirmed reference costs on this exact hardware class:

| System | Cost | Outcome |
|---|---|---|
| **Prismatic VLM, full 7B run** | **8x A100, under 9 hours** | ICML 2024 |
| C2LIP | 8x A40, CC3M, 5 epochs, batch 768 | CVPR 2026 |
| Open-Qwen2VL | 220 A100-40G GPU-h pretrain + 48 SFT | beats Qwen2-VL-2B |
| DIVA | 66.4 GPU-hours total | — |
| Perception-R1 | 16x A100, ~16h (~256 GPU-h) | — |
| DINORankCLIP | 8x H100, 72h for a *full ablation* | May 2026 |

A 1B model on 6.25B tokens is ~80 H100-hours — ten hours on eight GPUs. Full
fine-tuning at 7-8B, contrastive training at CC3M scale, VLM instruction tuning,
and small-scale pretraining are all **in scope**.

## T1 — the freeze x objective x stage factorial *(top pick of the session)*

> **Update 2026-08-02 — confirmed ★★★★½, design revised.**
> [[Direction-Reevaluation-2026-08]] re-gated T1 and it held at HIGH density:
> CoVFT (`2603.21077`) states freeze-vs-finetune "remains unresolved" and its own
> benchmark is SFT-only (VFT wins on 6/12); no consolidating survey on VLM
> training recipes exists; the 7B band is uncontested. **Design change:** the
> objective axis becomes **three-level — SFT / RL / SFT + perceptual auxiliary
> (VIRAL-style)** — because PIVOT already occupies {unfrozen} × {SFT, DPO}. The
> sharpened question: *does the freeze effect flip sign under a
> non-language-generation objective?* **Scoop risk is now concrete:** the CoVFT
> group adding an RL arm (use their public harness), and per
> [[Top-Researcher-Scan-2026-08]] **Darrell's group is the primary risk** — T1
> needs an immediate re-gate against his C1 cluster (CoVFT plus the four
> unarbitrated encoder-fix loci).

**A live three-way contradiction on the most basic VLM training question.**

**Prismatic** (`2402.07865`, ICML 2024) claims verbatim that *"including the
explicit projector pretraining stage is unnecessary, with single-stage training
improving aggregate performance"* (saving 20-25% of cost), and that
*"finetuning the visual backbone significantly degrades performance, especially
on tasks requiring fine-grained spatial reasoning"* — VQAv2 `77.09 -> 73.53`,
TextVQA `44.45 -> 38.33`, GQA `62.57 -> 59.65`.

**The field disagrees, in both directions:**

| Claim | Supports | Contradicts |
|---|---|---|
| Alignment stage unnecessary | Prismatic, Molmo | Eagle (pre-align helps *atop* unfreezing, `662.5 -> 672.3`), MM1.5, LLaVA-OV-1.5 |
| Unfreeze the ViT | Cambrian-1 (*"benefits performance across all benchmarks"*), Eagle (`616.5 -> 674.2`), InternVL3 (*"trains every layer jointly"*) | Prismatic, NVLM-1.0 (InternViT-6B frozen through *every* stage at 72B) |

**And Prismatic names its own suspect mechanism, untested by anyone:**

> *"The degraded performance from full finetuning could be for a number of
> reasons ranging from the scale and diversity of the vision-language data we
> train on to **language generation as a learning objective (vs. objectives that
> encourage learning fine-grained perceptual features)**."*

**PIVOT** (`2510.16333`) supplies exactly that missing arm: RL *"produces
stronger and precisely localized visual representations"* at under 1% of
vision-pretraining cost.

**The experiment:** `{frozen, unfrozen} x {SFT, RL/preference} x {one-stage,
two-stage}`, matched data and token budget, 7B. **~300-600 GPU-hours.**

Why it is the best item surveyed: it resolves a **cited, live, three-way
disagreement**; it tests the mechanism the original authors themselves named;
it **subsumes** the earlier freeze/unfreeze proposal (B2); the reference run is
**nine hours**; and every outcome is publishable.

## T2 — RL improves the answer without improving the seeing

> **Update 2026-08-02 — held ★★★★, reframed as a cross-method perception audit.**
> The original framing is partly scooped (`2602.12395` corroborated
> Perception-R1's null mechanistically; `2603.01301` ran the sharpening
> decomposition in the medical domain), and the PSR estimator was flagged broken
> on 2026-07-30 (`2607.28336`: it "conflates perceptual insufficiency with
> reasoning difficulty"). **What is open and unclaimed:** roughly 50
> perception-targeted RL methods exist against three 2026 diagnostics showing
> gains survive image masking/corruption (`2605.09266`, `2604.03179`) — and
> *nobody has run those controls on the methods claiming the fix*. The
> diagnostic sub-lane is unowned (those papers have 3 / 0 / ~0 citations).
> Inference-only on open weights, control arms are small 3B–7B GRPO runs,
> ~256 GPU-h, ICLR-2027-feasible. See [[Direction-Reevaluation-2026-08]].

**Perception-R1** (`2506.07218`): McNemar's test shows standard RLVR yields **no
statistically significant improvement in visual perception** (`p = 0.22-0.69`)
despite rising headline accuracy — RL amplifies latent correctness rather than
fixing perception. **PAPO** (`2507.06448`) confirms from the other side: **67% of
errors under standard GRPO are perception failures**; its fix cuts them 30.5%.

Other numbered RL failure modes on record: VLM-R1's mAP-reward gaming (models
spam redundant boxes); MM-Eureka's *"sudden training collapse"* at 32B with
reward reaching zero, and its finding that RL makes it *"difficult for the model
to acquire new knowledge — improvements come from increasing the probability
that the model generates correct answers."*

The incumbents published the null, the tests are statistical, and the discipline
required is exactly what this program has spent months building. **~256 GPU-h.**

## T3 — constant-compute data mixture (largest measured effect sizes)

> **Update 2026-08-02 — upgraded ★★★ → ★★★★½.** The apparent saturation was
> text-only: of the 8 mixture methods DataComp-VLM cites, 7 are text-only and
> the 8th is SFT-stage, and DataComp-VLM says verbatim that "there exists no
> systematic study on filtering and mixing strategies in the VLM setting" (zero
> surveys in its 347 references; 13 unresolved LaTeX labels including the
> promised multi-axis mixture appendix — that analysis is unwritten). **Live
> contradiction to adjudicate:** Shukor `2507.09404` (mixture scaling laws
> extrapolate) versus DataComp-VLM's measured rank inversion (caption-heavy wins
> at 1B×6.25B, instruction-heavy at 2B/4B×25B+) — neither cites the other on it.
> Public checkpoints at four scales turn a ~25,000 H100-h study into ~500.
> **Sharpened question:** can the small→large mixture-ranking crossover be
> *predicted* without paying for the large runs? A negative invalidates
> small-proxy mixture search. Watch: the consortium runs this as a competition.
> See [[Direction-Reevaluation-2026-08]].

**Filtering is dead; mixing is alive.** DataComp-VLM (`2606.28551`) verbatim:
*"no quality filter we tested produces a robust and significant improvement"* —
the best filter gives `+0.8pp`. But **mixture** at 70% instruction-tuning : 10%
caption gives **`+5.4pp`**, and *"a 4B model trained for 100B tokens beats an 8B
model trained on FineVision for 200B tokens."*

**20/20 VLM** (`2605.11405`): curation alone **at constant compute** (25B tokens,
2B params, single stage) gives **`+11.7pp`** on a 20-benchmark suite and
**`+57.1pp` on grounding** — parity with InternVL3.5-2B at ~17x less compute.

Corroborating nulls: MM1.5 *"did not find conclusive evidence that high-quality
synthetic captions improved performance over the arguably simpler OCR data"*;
and `2405.11850` reports SEED-Bench **dropping 3.3 points** as pretraining data
scales 20M to 100M. Constant-compute mixture design at 2-4B: **~400 GPU-h.**

## Also surfaced

**Connector design is a near non-factor** — MM1: *"the vision-language connector
design is of comparatively negligible importance"*; Eagle finds plain channel
concatenation (`690.4`) beats deformable attention (`674.3`). One sharp
exception: Cambrian-1's Perceiver-resampler collapse on OCR&Chart, `27.1` vs
`55.5`. **But none of those ablations measured compositional transfer**, and
C2LIP's `+6.8` encoder gain shrinking to `+0.4` through the connector, plus
CLIC's verbatim *"a detailed study of this is left for future work"*, remain
open.

**Verification item:** Prismatic's p-values sit in figure captions that do not
render in the arXiv HTML; two independent reads returned `0.00381` and `0.00407`.
**Confirm from the PDF before quoting.**

**Competitors on the clock:** DINORankCLIP (May 2026, same hardware class,
objective lane); Bottleneck Tokens and MoCa (embedding lane resets roughly every
two months); CABS (CVPR 2026, holds the concept-annotation data).

## T4 — anneal-window data allocation *(strongest paper; three sweeps converged)*

> **DOWNGRADED 2026-08-02: ★★★★★ → ★★. Do not start this.** Everything below is
> preserved as record, but the central premise — "there is no method paper for
> anneal-window data selection" — was **false when written**.
> [[Direction-Reevaluation-2026-08]] found the lane scooped three times:
> **DiReCT** (`2605.31175`, 29 May 2026) contains our exact motivation paragraph
> ("effectively selecting training data during this phase remains a key
> challenge... lack a principled grounding") at Llama-3-8B/300B with theory and
> code; **QAFSL** (`2605.25698`) owns "decay reduces update intensity exactly
> when high-quality data becomes available" with +1.70 over WSD at 15B-MoE; and
> **MIRA** (`2605.30288`) owns mid-training-selection-is-distinct. DiReCT had
> been public for ~8 weeks when the July sweep declared the lane empty.
> Compounding it: the small-scale moat is now citable *against* us
> (`2606.07597`: forked-decay extrapolation "frequently fails" when high-quality
> data repeats — our exact protocol); the object may dissolve entirely (WSM,
> WSO and `2604.13627` independently converge on less or no decay); the
> schedule-coupled subgenre has **zero top-venue acceptances**; and
> Compute-Constrained Data Selection (ICLR'25) shows gradient-class selectors —
> T4's lever — are almost never compute-optimal, so any survivor must carry its
> cost-aware baseline curve.
>
> **What remains is cheap, unclaimed and mechanism-shaped:** does the
> per-document value ranking *reorder* between stable-phase and decay-phase
> learning rates? Rank correlation on a shared trunk discriminates the "wasted
> data" story from the "sharpness" story, and a null undercuts the three papers
> that scooped the lane.
>
> **The process lesson (recorded in [[Direction-Reevaluation-2026-08]]):** an
> empty lane must explain *why* it is empty before being credited for emptiness,
> and any gate older than ~6 weeks needs re-running before compute is committed.

**The hole.** arXiv sweeps for `"annealing data"`, `"decay phase"`,
`"cooldown phase"`, `"annealing phase"` in cs.CL 2025-2026 return **essentially
nothing** beyond OLMo 2's system report. **The optimizer side of the cooldown is
studied** (`2508.01483`, WSM `2507.17634`, `2603.16127`); **the data side is
not.** There is no method paper for anneal-window data selection.

So the heuristic every frontier lab relies on — MiniCPM, OLMo 2 and Llama 3 all
dumping high-quality data into the decay phase — **has no method behind it.**

**The effect size is the largest in either survey.** PRISM: mid-training data
choice is worth **`+17` to `+28` GPQA-Diamond realized during RL**, while
changing the RL mixture is worth **under 2 points**. Mid-training restructures
over 90% of weights; RL touches about 5%. The field pours effort into RL recipes
while the data entering the anneal window matters an order of magnitude more.

**The lever is specific, not vague.** Estimate per-document marginal
decay-phase value **at the decay-phase learning rate** rather than the peak LR.
The LR enters the influence estimate linearly and everyone currently ignores it.

**Baselines to beat** (Qwen2.5-1.5B arch, 30B DCLM-Baseline, core avg over
MMLU/ARC/CSQA, from `2511.18903`): WSD+uniform `46.21`, WSD+ascending `45.45`,
EMA+ascending `46.95`, ConstLR+ascending `47.02`. Note what they did — they
**fixed the schedule to the data** (optimal curriculum end-LR `1e-3` versus
`1e-5` for uniform). The opportunity is to **fix the data to the schedule**, and
their own future work asks for exactly that: *"a more systematic recipe for the
strategy combinations."*

**A live contradiction resolved as a bonus axis.** `2603.16127`
(Warmup-Stable-Only, 1B and 8B) finds **no-decay consistently beats decay after
SFT**, even when decay wins on pretraining loss — decay drives sharper minima.
That is in direct tension with `2511.18903`. Controlling *what data is in the
window* adjudicates it.

**Timing is non-negotiable, and that is measured.** `2510.14865` (Pythia
70M-1B, 128B C4 tokens): code at **80% weight injected at 12B tokens is fine;
the same 80% at 105B tokens degrades worse than 10%**. Late introduction outside
the plasticity window cannot be compensated by more weight later.

**The design that makes it affordable and statistically honest.** WSD makes the
trunk shareable: **train the stable phase once** (20B tokens at 1B is about 256
H100-hours), **then fork K decay phases** (5B each, about 64 hours). Twelve arms
is roughly **1,020 H100-hours**, and **every comparison is paired.** That is
worth more than the method contribution itself — RegMix-D publicly concedes
**single-seed target runs** as a limitation, and PolyPythias (45 runs, 9 seeds x
5 sizes) shows seed effects are real.

## Crowding verdicts from the same sweep

> **Read these as saturation evidence, not as crowd counts (2026-08-02).** The
> selection criterion behind this list was corrected in
> [[Direction-Reevaluation-2026-08]]: *a topic is not filtered out by being
> hot — only by having fewer remaining opportunities.* The specific verdicts
> below still carry their evidence (Aioli's null, the 22 mixers, the
> repeat-placement gap), but "SATURATED" and "cleanest hole" labels derived from
> paper counts should not be trusted on their own. The anneal/decay entry in
> particular was wrong; see the T4 downgrade above.

- **Data mixture optimization: SATURATED, do not enter head-on.** At least 22
  methods between Jan 2025 and Jul 2026, **plus two surveys**. Aioli showed
  **no existing method consistently beats stratified sampling** (some up to
  `6.9` test-perplexity *worse*) back in Nov 2024, and the field responded with
  more mixers.
- **Curriculum/ordering: thin and contested.** Three serious pretraining papers,
  and they disagree.
- **Anneal/decay window: the cleanest hole.** One eight-month-old single-lab
  neighbour.
- **Repetition/replay: under-crowded**, and **nobody studies repeat *placement***.
  `2605.12715` (over 2,000 runs) finds mixture training tolerates **15-20
  repetitions** of a scarce corpus, far above the 4-epoch single-source rule.
- **Trap:** do not build on multi-token prediction at 1B — Meta's gains are
  `+12%` HumanEval at **13B**, explicitly *"increasingly useful for larger model
  sizes."*

## Two standing caveats for any training study here

1. **Budget two scales in everything.** Ranking inversion is now documented in
   post-training algorithms, optimizers **and** mixtures. DataDecide (1,050
   models, 25 corpora x 14 sizes x 3 seeds): ranking at 150M predicts the
   1B-best corpus in only **~80%** of pairwise comparisons, and **none of eight
   scaling-law baselines beats naive single-small-scale prediction.** Apple's
   joint mixture law proves the mechanism — the cross-derivative is nonzero, so
   the optimal weight vector provably moves with scale.
2. **Never run single-seed 1B comparisons.**

## Choosing between T1 and T4

> **Moot as of 2026-08-02 — T4 is dead (★★, scooped 3×).** The comparison is
> kept because its *reasoning* is still the right way to choose between two
> training studies, and because it is the clearest example of how the analysis
> can be sound while resting on a stale emptiness claim. The live comparison is
> now T1 versus **T3** (both ★★★★½); see [[Direction-Reevaluation-2026-08]].

**T1 (VLM freeze x objective x stage)** is cheaper (~300-600 GPU-h), sits in the
domain where infrastructure already exists, and resolves a three-way
disagreement. **T4 (anneal window)** has the larger effect size, a genuinely
empty lane confirmed by three independent sweeps, and a more elegant design —
but costs ~1,000 H100-h and is further from prior experience.

**T1 is the better fit; T4 is the stronger paper.**

> **What the re-gate proved about this judgement.** "Confirmed by three
> independent sweeps" was worth nothing: all three sweeps shared the same
> keyword vocabulary and all three missed a paper that had been public for two
> months. Independence of *searches* is not independence of *evidence*. Crowd
> count predicted nothing either — every crowding-based downgrade in the July
> ranking (T3, AHD, self-improvement, KV) was reversed, and both
> emptiness-credited directions (T4, B1) fell.

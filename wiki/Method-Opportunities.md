# Method Opportunities — with baselines and numbers to beat

Status: **Active 2026-07-26.** Written after the owner correctly pointed out
that every prior survivor was measurement work.

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

# Cluster 1 — compositional VLM scoring (uses infrastructure already built)

**Execution:** [[Cluster-1-Compositional-Scoring]] locks the P1 protocol,
records the prior-art boundary, and carries P1/P2/P3 results.

**Outcome update (2026-07-26):** P1 and the internal P2 mechanism ablation are
closed. Fixed-bank normalization improves COCO retrieval but changes SigLIP2
Winoground Group only `11.25 -> 11.50` (`95% CI [-1.00,+1.50]`; test-selected
maximum `12.00`). Hard exclusive assignment over the existing SigLIP2 crop
pipeline gives active SCPP++ Swap `+0.44` (`CI [-0.78,+1.66]`), significantly
harms Replace by `-0.77` (`CI [-1.25,-0.32]`), and changes active Winoground
Group by `+0.90` (`CI [0.00,+2.11]`). Wrong Swap captions collide slightly more
often, but the negative-minus-positive exclusivity penalty is only `0.00015`
against the predeclared `0.02` gate. Both local branches select `alpha=1.0` on
development, returning raw SigLIP2 when fused.

P2's algebraic gate passed: exclusive assignment can change the actual pairwise
metrics. Its empirical mechanism gate did not. The official SGI P2/P3
reproduction remains blocked because both released repositories still contain
only "Coming soon" placeholders; the internal P2 result is not represented as
an SGI comparison. The earlier one-pass patch-grid control remains adjacent
P3 evidence.

## The Test-Time Matching metric distinction

Test-Time Matching shows frozen SigLIP-B16 Winoground group goes **10.25 →
67.00** under joint assignment, and MMVP-VLM **22.96 → 81.48**. That is
transductive — it requires the test-set group partition — so it is not a
deployable method and not a legal comparison. It also changes the event being
scored: four independent inequalities with `16.7%` random Group chance become
one joint assignment comparison with `50%` chance. The `67.00` is therefore not
calibration headroom and does not prove that frozen similarity ordering is
nearly correct.

Our P1 derivation further proves that separable image and caption bias
corrections cancel exactly in the joint assignment margin. The earlier `30-45`
target below was algebraically invalid and should have been removed before the
run.

## Correction to my earlier brief

I reported `73.0 → 86.3` on SugarCrepe as a frozen-encoder result. **It is not.**
That is **TF_Local**, a *trained* 13.3M-parameter cross-modal transformer. **SGI
(the training-free method) is never evaluated on SugarCrepe, BiVLC, ARO or
Winoground** — only on BiSCoR-Ctrl. And "beats full fine-tuning at 85.5" is
trained-versus-trained. The frozen-inference story is narrower and weaker than I
said, which is precisely why it is an opening.

## Documented failures of the current best — the targets

**The field is anti-correlated on controlled swap binding.** BiSCoR-Ctrl group
score, **random = 16.7**: NegCLIP `1.8`, TripletCLIP `1.2`, FSC-CLIP `1.2`,
X-VLM `1.7`, FineCLIP `1.4`, full fine-tuning `1.9`. Every end-to-end method sits
at roughly one tenth of chance.

**Size and Quantity are at chance for everyone.** Best in world
(SigLIP2-G + structured inference): Size `20.2`, Quantity `19.4` against random
`16.7` — while Color reaches `95.7` and Material `91.7`. The authors' own
diagnosis: *"for SIZE, the image crops that capture objects in isolation tend to
remove the size information."* **Cropping solves colour and destroys scale.**

**Structured inference degrades I2T everywhere, and nobody has diagnosed it.**
CLIP Winoground I2T `31.25 → 23.75`; group `8.75 → 6.75` (it *hurts*). SigLIP2-G
BiVLC I2T `57.1 → 56.3` while T2I goes `9.2 → 36.5`. All gains are T2I-only.

**Cost is 133-210x.** Measured: CLIP BiSCoR-Ctrl 1k instances 36s → 1.33h;
SigLIP2-Giant 6min → 21h. The authors name ROI-Align amortization as unexplored
future work.

**Sobering calibration.** SVIB's own result is SigLIP2 SCPP `75.91 → 76.71`,
Winoground `11.25 → 13.00`. The **training-free** 2025 crop baseline gets SigLIP2
Winoground `10.75 → 13.75` and BiVLC `13.7 → 24.5`. The trained graph module is at
parity with a training-free baseline that predates it. Any new proposal must
leapfrog that, not match it.

## P1 — inductive score calibration *(closed)*

**Lever.** The residual error after joint assignment is a per-image and
per-caption **additive bias** (hubness, modality gap, caption prior). Estimate
marginals against a **fixed background corpus** rather than the test group:
`s'(I,T) = s(I,T) - λ_i·E_{T'~B}[s(I,T')] - λ_t·E_{I'~B}[s(I',T)]`, or full
Sinkhorn scaling against background marginals. Fully inductive — one pair, one
forward pass, background embeddings cached once.

**Historical target, invalidated before interpretation:** SigLIP2 Winoground
group `11.25`; best *inductive* method `13.75`; target `30-45`. The target
mistook a metric change for accessible separable-calibration headroom.

**Cost: hours on one GPU.** The caches and both evaluators already exist.

**Risk:** QB-Norm and DualSoftmax exist in cross-modal retrieval. The novelty must
be the *claim* — "compositional failure is dominantly a calibration artifact,
and here is the inductive fix" — not the normalizer.

## P2 — Sinkhorn/OT assignment instead of greedy argmax

**Lever tested.** Structured inference uses **greedy independent argmax** with a
uniform mean. There is no exclusivity constraint, so "black cat" and "black dog"
can both claim the same black region. We replaced it with exact maximum-weight
partial one-to-one assignment and private dustbins.

**Outcome: closed.** The hypothesized collision direction exists
(`36.21%` negative versus `33.50%` positive on active Swap), but the penalty
separation is only `0.00015`; near-equivalent alternative crops make the hard
constraint almost inert. Active Swap improves only `+0.44` point with an
interval spanning zero, while Replace falls a resolved `-0.77`. A Sinkhorn
entropy sweep is not justified for this segment/crop representation.

**Historical external targets:** CLIP+SGI BiSCoR-Ctrl `24.9`, PE+SGI `44.7`,
SigLIP2-G `56.8`; and the per-category collapses CLIP Size `4.7`, Quantity
`2.5`. These were not evaluated because the official assets are unreleased.

**Cost:** trivial — Sinkhorn on an N x M matrix.

## P3 — amortized structured inference (kill the 210x)

ROI-Align pooled region features off one dense forward pass, replacing 86-270
independent encoder passes. **Beat:** SigLIP2 BiVLC `24.5` at 160x cost — match
at under 2x, or beat by spending the freed budget on more regions. Ship the
latency table nobody has.

## P4 — size- and count-preserving region representation

Crops destroy scale by construction. Condition each region embedding on its box
(scale, position, aspect ratio); add multi-resolution context regions.
**Beat:** Size `20.2`, Quantity `19.4`. Target above 40. Highest raw headroom of
anything here, and the cleanest narrative.

**Cluster decision:** P1 and internal P2 both close. P3's one-pass patch-grid
evidence remains useful engineering evidence, but the combined claim that
compositional failure is dominantly a scoring problem is not supported.

**Protocol warning.** Never compare inductively against Test-Time Matching
numbers: they require the test-set partition and score a different event with a
different chance level. The `67.00` must not be described as available
calibration headroom.

---

# Cluster 2 — KV-cache bit allocation for the right objective

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

**Cluster 1**, because the infrastructure is already built and validated, the
headroom is documented by a third party's ICLR paper, and P1 costs hours.
**Cluster 2** if the goal is to move off VLM — cleaner "beat this number"
framing, but a crowded and industry-heavy area.

Neither has been prior-art gated. Given the record, gate before committing — but
gate with the **method** bar (is there a lever?), not the **audit** bar (is it
unprecedented?).

## Related

[[Live-Research-Opportunities]] — the measurement-side directions.

---

# Program-level finding: the local-branch family is exhausted (2026-07-26)

P2 closed. Hard exclusive assignment is **not** algebraically invariant — unlike
P1 — so it could have moved the metric. It did not:

| Endpoint | Effect | 95% CI |
|---|---:|---|
| SCPP++ active Swap | `+0.44` | `[-0.78, +1.66]` |
| SCPP++ active Replace | `-0.77` | `[-1.25, -0.32]` |
| Winoground active Group | `+0.90` | `[0.00, +2.11]` |

**The mechanism check is the valuable part.** Negative captions *do* collide more
often — the hypothesis was directionally correct — but penalty separation is
`0.00015` against a pre-registered `0.02`. **Right mechanism, wrong magnitude by
two orders.** That is far more informative than a bare null, and it is only
available because the protocol pre-specified the mechanistic quantity rather
than just the outcome.

**Development selected `alpha = 1.0` for both local methods** — the protocol
rejected the local branch entirely in favour of raw SigLIP2.

## Six mechanistically distinct probes, one answer

| Probe | Intervention | Outcome |
|---|---|---|
| SVIB | SAM3 proposals + directed sparse graph + VIB | every structural component ablatable; grid beats SAM3, attention beats graph, beta=0 matches |
| Patch-grid | pooled ViT patch tokens as local nodes | loses to crop re-encoding (CLIP `-1.32`) |
| Claim-level | deterministic caption decomposition | hurts on all four backbones (`-2.17` to `-3.51`) |
| Conformal dispersion | equivalence-class score spread | no signal beyond a margin-only control (0/4) |
| P1 | separable marginal calibration | `+3.0` COCO retrieval, `+0.25` Winoground |
| P2 | exclusive OT assignment | mechanism confirmed, magnitude 130x too small |

**The sharpest statement is not "our methods gave small gains."** It is:

> Under validation-locked operating-point selection, six mechanistically
> distinct local-evidence branches on a frozen dual encoder are all rejected in
> favour of the raw global score.

The field's own numbers agree independently: every end-to-end method scores
`1.2-1.9` on BiSCoR-Ctrl where random is `16.7`; C2LIP's headline architectural
contribution is worth `+0.3`; structured inference degrades I2T everywhere.

**Recommendation: stop probing this representation.** P4 (size/quantity) is in
the same family and swims against a documented current — crops are the mechanism
that *destroys* scale information. Expect the same outcome.

## What the six negatives are worth

This is an unusual asset: **six completed, pre-registered, provenance-hashed
experiments** on one question. Most negative-results papers are a single
experiment; this is a program. And unlike an audit, it examines **our own work**,
which sidesteps the reviewer-conflict objection entirely — nobody has to be told
they were wrong.

The distinctive contribution is the **selection protocol**: published local-branch
methods report gains under test-selected or per-benchmark operating points, and
the same methods are rejected at `alpha = 1.0` under validation-locked selection.
That is demonstrable from our own runs without auditing anyone.

Venue: TMLR (scope explicitly covers this) or ICBINB. **The experiments already
exist** — this is a writing task, not a compute task.

## New work should move areas

**Cluster 2 (KV-cache allocation)** is untouched, in a different area, with
cleaner "beat this number" framing and two failure modes documented by third
parties who explicitly did not fix the allocator. 2-4 GPUs, no training.

**Or the B-cluster** in [[Live-Research-Opportunities]] — diversity-collapse
isolation (Olmo 3 supplies open weights *and* open per-stage data) and visual
attention-sink emergence (one freeze/unfreeze experiment). Both are positive
mechanism contributions in areas unrelated to frozen dual-encoder scoring.

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

**T1 (VLM freeze x objective x stage)** is cheaper (~300-600 GPU-h), sits in the
domain where infrastructure already exists, and resolves a three-way
disagreement. **T4 (anneal window)** has the larger effect size, a genuinely
empty lane confirmed by three independent sweeps, and a more elegant design —
but costs ~1,000 H100-h and is further from prior experience.

**T1 is the better fit; T4 is the stronger paper.**

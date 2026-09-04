# Why VLMs Still Fail When Roles Are Swapped (2026-08-02)

This note starts from the basic problem instead of looking for another pattern
in the results. The owner asked for this kind of root-cause analysis.

The failure looks like this: a model must tell “dog chases cat” from “cat
chases dog.” Both sentences contain the same ideas, but the dog and cat have
switched roles. Models remain near random chance on these cases. This is still
true after researchers tried more interaction between image and text, local
matching, and extra training data.

## 1. Model size is not the basic cause

Take one group of four items:

- `I_dc`: an image where the dog chases the cat;
- `I_cd`: an image where the cat chases the dog;
- `T_dc`: the first caption; and
- `T_cd`: the second caption.

Even a rank-1 dot-product score can separate this one group. It only needs

`⟨f(I_dc) − f(I_cd), g(T_dc) − g(T_cd)⟩`

to have the right sign. This is one simple linear rule. With enough dimensions,
the model can satisfy any fixed number of swap rules. Therefore, it is false
that “one vector can never represent relationships.”

The hard part is **systematicity**: the model must use the same role rule on a
new combination such as `(A, relation, B)`. It cannot simply memorize a
different direction for every pair. The real question is why training does not
learn one dependable way to bind an object to its role.

## 2. Three possible causes and how to tell them apart

### A. The training data rarely requires role binding

In most image-caption pairs from the web, the set of objects is enough to find
the right image. Cases where both role orders make sense are extremely rare.
InfoNCE, CLIP's contrastive training goal, learns only the information needed
to choose the right pair. If the object list is enough, the model gets almost
no training signal about roles.

**Prediction:** training on many paired images with swapped roles should fix
the problem.

**Current evidence:** this explanation is hurt but not ruled out. Text-only
hard negatives, from NegCLIP through
[DeGLA (2504.16801)](https://arxiv.org/abs/2504.16801), improve whole-benchmark
scores but not swap cases. A clean test needs pairs of swapped **images**, such
as the VisMin or COCO-Counterfactuals type. We have filtered copies, but nobody
has run this test at large scale.

### B. The training goal hides role information at the shared interface

Contrastive training can hide features that are not needed to choose the right
pair. Theory calls this **feature suppression**. The usual
alignment-and-uniformity goal also has no rule that keeps sentence structure.

Our results fit this account. Before image patches are pooled into one vector,
their binding signal scores 4.79. The pooled global score is only 0.15. The text
tower remembers word order when tested alone, but the cross-modal map that
compares image and text acts like a bag of words
([LABCLIP, 2502.03566](https://arxiv.org/abs/2502.03566)). The information seems
to disappear exactly where InfoNCE trains the two sides to meet.

**Prediction:** a different readout or training goal should recover roles from
frozen image and text towers.

**Current evidence:** partly supported by
[TF_Local, 2604.11496](https://arxiv.org/abs/2604.11496) and generative scoring
on overall benchmark scores. However, if detailed readouts still fail the
hardest swap items, suppression cannot be the whole cause. The unpooled
features may show that two objects appear together without showing their roles.

### C. Adding word meanings cannot represent roles

A bag-of-words model adds meanings together:

`e_dog + e_chase + e_cat`.

Addition does not depend on order. In math, it is **commutative**. Therefore,
this rule makes `E(dog chases cat) = E(cat chases dog)` by design. This may be
the normal end point of an additive system, not a training bug.

CLIP text vectors are close to additive in real tests. A new research area is
forming around binding functions
([Tübingen, 2605.31503](https://arxiv.org/abs/2605.31503);
[2602.24264](https://arxiv.org/abs/2602.24264)). Math for order-sensitive
binding in a fixed-size vector has existed since the 1990s:

- Smolensky's Tensor Product Representations (TPR) bind a role to a value with
  `role ⊗ filler`.
- Plate's Holographic Reduced Representations (HRR) use circular convolution.
  They keep a fixed vector size and still work with dot products.
- Vector Symbolic Architectures (VSA), also called hyperdimensional computing,
  use related ideas.

For example,
`bind(agent, dog) ⊕ bind(patient, cat)` and
`bind(agent, cat) ⊕ bind(patient, dog)` point in very different directions in
a high-dimensional space. Swapping the roles changes the vector by design.

## 3. Research idea to check, not assume

**Role-bound contrastive vectors using an HRR- or TPR-structured CLIP head.**

On the text side:

1. Parse a caption into roles such as agent, action, patient, or
   attribute-object.
2. Bind each value to a fixed role vector with circular convolution.
3. Combine the bound parts into **one** vector.

On the image side, add a small head over frozen ViT patch tokens. It should
produce the same structure: object slots from patches and relationship
direction from geometry.

The score remains a plain dot product. Each image needs one cached vector and
one normal inference pass. We can create swap negatives for free by changing
the role bindings. No new swapped images are needed. This design attacks causes
A and C together, while the new readout also tests cause B.

The difficult and honest research question is whether image roles can be found
without detailed labels. Our evidence before pooling is directly useful here.

## 4. The cheap experiment that separates the three causes

Run a **linear role-decoding test** on swapped pairs from VisMin and COCO-CF in
`datasets/external_compositional`. Ask whether a linear model can tell who has
the agent role from:

1. image patch tokens;
2. the pooled image vector; and
3. the text vector.

The possible results mean different things:

- Roles appear in patch tokens but not pooled image or text vectors: pooling
  hides the answer, so an algebraic head on frozen towers may work.
- Roles do not appear even in patch tokens: the encoders never learned the
  relationship, so training must change before the final layers.
- Roles appear everywhere: the problem lies in scoring or carrying out the
  comparison, as the Similarity-Is-Not-Logic account predicts.

The test should cost about 20–50 GPU-hours with cached or newly encoded
features. It also gives Cell A a mechanism-level test beside its ladder of
readout methods.

## 5. Gate result on 2026-08-03: SURVIVES, BUT NARROWER — ★★½

**CORRECTION, 2026-08-06.** This section's survival claim was wrong when
written. A deeper check found [DisCoCLIP (2509.21287)](https://arxiv.org/abs/2509.21287)
(Sept 2025): a fixed-algebra trained composition head on a frozen CLIP
that keeps one vector per side and one dot product — the exact cell this
section declared empty. Our search was keyed to the HRR/VSA/TPR
vocabulary; DisCoCLIP calls the same mathematics a "tensor network."
Additionally, [2608.00726](https://arxiv.org/abs/2608.00726) (Aug 2026)
now publishes the patch-tokens-retain-binding finding of §6–8, and
[2510.24709](https://arxiv.org/abs/2510.24709) reports object binding
emerging in pretrained ViTs. The method direction built on this section
was killed on 2026-08-06 (Level 1 overlap); what remains uniquely ours
is the powered two-strata evidence, the inversion, and the cross-task
specificity result of §9. The original section is kept below as the
honest record of what we believed on Aug 3.

The closest work is
[OC-CLIP (Meta FAIR/Mila, 2502.14113)](https://arxiv.org/abs/2502.14113).
OC-CLIP parses captions into scene graphs, turns ViT patches into object slots,
uses an order-sensitive relationship score, and trains on swaps created by
algebra. It does not need swapped images. This covers most of the idea above.

One important difference remains. OC-CLIP's slots depend on the text, and its
score is a sum of structured parts. It therefore works more like a cross
encoder and does not produce one cacheable vector. We found no paper that puts
a fixed, order-sensitive binding operator inside each frozen tower and then
uses one vector and one dot product at about 1× cost. A search of 43 HRR papers
found zero that use it for CLIP or image-text matching.

The clearest risk and claim come from
[2604.16351](https://arxiv.org/abs/2604.16351). It shows that training for
compositional sensitivity lowers dense-retrieval quality by 8–40% in a
text-only setting. The testable claim is that algebra built into the model may
avoid that tradeoff.

Supporting theory:

- finding shared content: [2303.09166](https://arxiv.org/abs/2303.09166);
- shortcuts and feature suppression:
  [2106.11230](https://arxiv.org/abs/2106.11230) and
  [2011.02803](https://arxiv.org/abs/2011.02803);
- near-linear meanings: [2302.14383](https://arxiv.org/abs/2302.14383); and
- HRR capacity: [2301.10352](https://arxiv.org/abs/2301.10352).

We did not search OpenReview because its API limit was exhausted. Search it
again before any pre-registration.

## 6. Probe result on 2026-08-03: suppression is real, with two limits

This was a small pilot, not a full study. A pre-registered stop condition fired:
only 173 usable swap pairs were available, below the minimum of 300. The
resolution limit was 0.65 instead of the required 0.55. Still, the main
difference was far larger than that limit.

The pilot used 173 pairs, or 346 images, grouped 5-fold, with 5 seeds, 10,000
paired bootstrap samples, and Holm correction for multiple tests.

- **Frozen ROI or patch features:** role assignment was 93.2% decodable at
  layer 6, 99.0% at layer 9, and 99.9% at layer 12.
- **Pooled image vector:** 52.5%, which is chance after Holm correction
  (`p=.056`).
- **The model's own scoring direction:** 50.3%, exactly chance.
- **Text:** 67.5%, driven by word order. Scrambling the order reduced it to
  chance.
- **Unexpected reversal:** the untrained region-text readout points the wrong
  way. Flipping its sign, with no learned parameters, gives **100.0%** at
  layer 12. A localization check that does not use `roi_align` also found that
  similarity for an entity usually peaks inside the **other** entity's box.
  Only 3–13% of slots score their own box higher. The frozen encoder contains
  the answer, but the normal readout uses it backwards. This resembles a known
  CLIP dense-feature problem. Verify that earlier work before claiming it is
  new. It directly supports the A4/CLIPSelf-repair test.

### Limit 1: only spatial relationships

All 173 pairs use location words such as on, left, right, above, or below.
No agent-patient verb swaps remained in our filtered data.
COCO-Counterfactuals had 0 such swaps among 11,194 items because it changes
objects rather than swapping them. About 97.5% of VisMin's relationship subset
was removed by our strict source-tracking rule because those images are fully
synthetic. This pilot answers “who is where,” not “who chases whom.”

### Limit 2: synthetic images

The images were made by diffusion models from layouts and have clean
backgrounds. The almost perfect patch-feature result is probably an upper
bound, not the usual real-image result.

### Meaning of the result

For spatial roles, the pattern is patch features = yes, pooled image = no,
model score = no. The nearest cause is suppression at the readout. The claim
that encoders never find roles is false for spatial relationships. The first
stop condition for algebraic binding is cleared for spatial roles. Verb roles
remain unknown.

The owner considered three ways to get more data:

1. Use VisMin-bench's 622 human-checked exact swaps. It is only 1.5 GB and is
   ready now, but it is the lab's protected test-only set. Fitting probes on it
   would spend that clean test set.
2. Download the full VisMin relationship subset again: about 6,973 pairs and
   39 GB. Allow a documented exception to the strict source rule because the
   images are synthetic and cannot come from COCO. Measure caption overlap.
3. Do option 2, while keeping option 1 untouched as the final check.

The worker and coordinator both recommend option 3.

The most useful next experiment is the same for every open VLM thread: test
whether a head over frozen ViT patches can assign agent and patient roles
**without seeing the caption**. OC-CLIP avoided this question by letting text
choose its slots. The test should cost about 10 GPU-hours inside the larger
20–50 GPU-hour probe study.

One probe study would therefore:

- separate the three possible causes;
- test the key assumption behind the algebraic method; and
- provide the mechanism story for Cell A.

It is the cheapest experiment that moves every open VLM question forward. It
does not need pre-registration because it is an internal evidence study, and
it can run on OrangeGrid now. Run the probe first. Keep algebraic binding at
★★½, behind Cell A at ★★★½. Raise its rating only if the probe passes.

## 7. Powered replication on 2026-08-03: the finding holds at 40× scale

We rebuilt the corpus (6,868 spatial pairs re-streamed from VisMin under a
measured waiver — zero COCO ancestry, zero exact caption overlap with the
eval battery — plus 47 real-photo verb items from the owner's hand-verified
gold labels) and reran the probe with a fresh GPU encode inside one
interactive allocation (15 GPU-minutes; smoke gates passed; node released
automatically). Full tables:
cropdistill `runs/role_probe/20260803-corpus/full-gpu/`; corpus + waiver:
`datasets/role_probe_corpus_20260803/`.

**Spatial stratum (n=6,868 — now genuinely powered):**

- Patch/ROI features: 96.2% (layer 6) → 99.2% (layer 9) → **99.5%
  (layer 12)**, CI [99.3, 99.6].
- **The entity-disjoint split barely moves it: 99.4%.** The probe works on
  entity pairs it never saw in training — this is the systematicity
  evidence the pilot could not give: a general role rule, not memorized
  pairs.
- Pooled image vector: 52.5% [52.1, 52.9] — a real but tiny trace, far
  below the 0.55 bar. The model's own scoring axis: **50.1% — blind.**
- **The inversion replicates:** untrained readouts score 0.6–4.6%, i.e.
  the sign flip recovers 95–99.4% with zero fitted parameters.

**Verb/real-photo stratum (n=47 — still underpowered; resolution 0.73):**

- Patch/ROI layer 12: **76.2%** [64.3, 87.2] — decodable; entity-disjoint
  69.8% [58.3, 81.3], still clearing the bar at layer 12. Read this
  against the stratum's own ceiling: region-IDENTITY is only 80–85%
  resolvable on these real cluttered photos (vs 99.6% on synthetic), so
  role information sits close to the identity ceiling.
- The model's own scoring axis on real verb swaps: **29.8% [17.0, 42.6] —
  significantly BELOW chance.** On this small set CLIP does not merely
  ignore verb roles; it prefers the wrong caption. Same inversion theme,
  now at the behavior level. n=47 — treat as a lead, not a claim.
- Pooled vector: unresolved (pair split) and anti-generalizing under the
  6-cluster entity split — underpowered artifacts; do not interpret.

**Bottom line:** the suppression verdict is now a powered, replicated,
systematicity-tested fact for spatial roles, holds directionally on real
photos with verbs, and the inversion is confirmed at scale. The verb arm
needs more labeled data (the owner's blind UI over the 401-row pool is the
path) before its numbers carry weight on their own.

## 8. Verb stratum at full power on 2026-08-06: one claim corrected, one confirmed

The owner finished labeling all 401 verb candidates. After removing rejects
(71 wrong-both, 21 wrong-neither, 2 unclear) and 28 items where the box
could point to more than one person, **279 usable items** remained — six
times the pilot. Predicates now span 8 verbs (watching 155, looking at 63,
holding 29, and five more) instead of the pilot's 3. We reran the full
probe with a fresh GPU encode. A holdout check (a second, independent
program recomputing the numbers) matched exactly, and the spatial numbers
reproduced bit-for-bit across two different GPU nodes. Full tables:
cropdistill `runs/role_probe/20260805-verb/report/`.

One honesty note first: the 47 pilot items are contained inside the 279,
so comparing pilot to full is a precision gain, not an independent test.
The independent test is the **232 items the pilot never saw**.

**Corrected: the "below chance at the score" claim.** The pilot's most
dramatic number — the model's own scoring axis at 29.8%, seemingly
*preferring the wrong caption* — does not hold up. At n=279 it is 44.8%
[39.1, 50.9], and on the 232 unseen items alone it is 47.8% [41.4, 54.3],
p=0.56 — indistinguishable from a coin flip. The 29.8% was small-sample
noise. The score axis is *blind* to verb roles, not reversed.

**Confirmed: the inversion one level down.** Matching a region's features
directly to text with no training scores clearly BELOW chance on the
unseen items at every layer tested — 38.8% (layer 6), 33.2% (layer 9),
34.9% (layer 12), every interval fully under 50%. Anti-correlated role
information in the raw region-to-text readout is now an independently
replicated fact on real photos, and it gets worse in deeper layers.

**The rest of the picture at n=279 (pair split):** trained probes read
roles from patches at 64→70→74% (layers 6/9/12, all decodable), from text
embeddings at 74%, while the pooled image vector sits at 52–57% — below
the 0.55 bar on both split types. The region-identity control is healthy
(89–92%), and both scrambled controls sit at chance (leak check clean).
The pilot's scary entity-split collapse of the pooled vector was itself a
small-n artifact: with 23 entity kinds instead of 6, the split barely
moves anything.

**Bottom line, updated:** both strata — synthetic spatial (n=6,868) and
real-photo verbs (n=279) — now land in the SAME verdict cell:
role information is present in the patches, present in the text, and
destroyed at the pooled readout. The behavioral score is blind (50%), not
anti-aligned; the anti-alignment lives in the raw region readouts. This
is the cleanest version of the suppression story yet, and it is exactly
the situation the Cell A readout-budget experiment is built to exploit.

## 9. A cross-task contrast on 2026-08-06: suppression is specific, not universal

Two same-day results sharpen what the suppression story does and does not
claim.

**MOCHI (3D view consistency) shows the opposite pattern.** We ran our
probe design on [MOCHI](https://arxiv.org/abs/2409.05862), a benchmark
asking whether two images show the same object from different viewpoints.
There, nothing is suppressed: the model's own similarity score is the
*best* readout (SigLIP2: 70.0% vs patches 68.3%), pooling costs 3–5
points instead of 47, and no readout at any layer of any of three model
families is inverted. View identity is *never built* by these models —
there is no hidden signal for a better readout to recover. Role binding
is different in kind: the signal exists (99.5% / 74% in patches) and is
destroyed on the way out.

**The Cell A pilot killed the naive repair.** Simply reading the patches
with an untrained rich readout does NOT recover binding on the benchmarks:
it scores 21 points *below* the model's own pooled score, and more patches
help retrieval monotonically while never helping binding. It also showed
that an apparent sign inversion at battery scale was a caption-length
artifact of the scoring rule; the real inversion is confined to
unprojected token space, as §8 found.

**Together:** the suppression cell is real, specific to role binding, and
not fixable by readout choice alone — whatever recovers the patch-level
role signal must be *trained* (the algebraic-binding direction), not
merely wired differently.

## 10. The inversion in context (2026-08-07): known elsewhere as "opposite visualization"

The final method attempt built on this page — a tiny learned correction
of the sign-inverted region readout — was gate-checked and killed. The
inversion phenomenon has been known in the model-explainability
community since 2022 under different words: [ECLIP/RCLIP
(2209.07046)](https://arxiv.org/abs/2209.07046) found that CLIP's
region evidence points at the background ("opposite" regions), fixed it
with a parameter-free flip AND a small trained linear layer, and [CLIP
Surgery (2304.05653)](https://arxiv.org/abs/2304.05653) measures it as a
metric that is negative on all twenty of its model×dataset cells.
[LABCLIP (2502.03566)](https://arxiv.org/abs/2502.03566) — which §2B of
this very page cites for its diagnosis — already ships a 262K-parameter
frozen-CLIP linear correction that restores attribute binding. Our §6
note "verify that earlier work before claiming it is new" was right and
was not executed until now.

What this page still contributes, folded into the TMLR paper: the first
POWERED measurement of the phenomenon against role-binding items.
**CORRECTION (2026-08-26): the four-locus role probe ran on OpenAI CLIP
ViT-B/32, NOT on the SigLIP2 attention-pool path as this paragraph
previously said.** The run manifests
(`cropdistill/runs/role_probe/20260803-corpus` and `20260805-verb`,
both `"model_id": "openai/clip-vit-base-patch32"`) are the record; the
pooled-locus dimension (512) confirms it. The TMLR draft inherited the
wrong attribution from this paragraph and was corrected the same day.
What remains true: the probe is the first powered role-binding
measurement of this kind (prior work measured segmentation data), and
the separate benchmark-scale untrained-readout analysis (Cell A) did
run on SigLIP2. The caption-length warning (a "flip" at benchmark scale can
be pure length bias — a direct caution for anyone following DCSM's
advice to "recognize inverted patterns" from scores); the honest
29.8→44.8 self-correction; and the MOCHI specificity contrast. The
search lesson (our words "inverted/anti-aligned/below chance" return
zero hits; the field says "opposite visualization" / "prefers the
background") is now part of the gate rules.

## 11. Text encoders of text-to-image systems (2026-09-04): the token states keep roles, the pooled vector does not

Asked by the professor for the Sony proposal: does the text tower also lose
role information, and what does that mean for text-to-image generation? We
ran the recorded text-locus probe (linear, one classifier, no images) on the
prompt encoders of Stable Diffusion 1/2/XL/3 and FLUX: OpenAI CLIP ViT-L/14,
OpenCLIP ViT-H/14 and ViT-bigG/14, and the T5 v1.1 XXL encoder, after a gate
that reproduced our three recorded backbones within 5e-5. Evidence:
`cropdistill/runs/t2i_text_probe_20260904/report.md`.

- **Pooled sentence vector** (what the recorded instrument reads): 0.57 to
  0.74 on the synthetic spatial set (CLIP ViT-L/14 0.628, T5-XXL 0.569).
- **Token states** (what every diffusion model actually consumes): 0.94 to
  0.995 spatial and 0.985 to 0.993 on real-photo verb pairs, on all four
  encoders, intervals under 0.02 wide; controls at chance.
- The pooled loss is mostly representation (mean token state under the same
  linear probe lifts CLIP ViT-L/14 from 0.628 to 0.988) and partly readout (a
  one-hidden-layer probe on the pooled vector lifts it to 0.928): entangled,
  not empty, the same pattern as the image side.
- For T5 and SigLIP 2 (text towers not contrastively aligned with entity
  names) the recorded elementwise-product instrument is the wrong tool (0.57
  and 0.57 linear; 0.94 and 0.91 with a one-hidden-layer probe): any table
  must say which probe.

Reading for the proposal: the prompt encoder keeps who-does-what in the token
states the generator reads; a binding failure in generation therefore sits in
how the generator reads those tokens, the same readout pattern as the image
tower. We ran no generation experiment; that sentence is a hypothesis.

## Related

[[Method-Gates-Wave-3-2026-08]] · [[Prereg-Crop-Consistency-Distillation]] ·
[[Status-And-Survivors]] · [[Wiki-Citation-Audit-2026-08]] ·
[[Compositional-VLM-Survey]]

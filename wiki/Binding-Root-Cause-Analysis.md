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

## Related

[[Method-Gates-Wave-3-2026-08]] · [[Prereg-Crop-Consistency-Distillation]] ·
[[Status-And-Survivors]] · [[Wiki-Citation-Audit-2026-08]] ·
[[Compositional-VLM-Survey]]

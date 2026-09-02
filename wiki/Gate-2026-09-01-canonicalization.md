# Method Gate — Role-preserving canonicalization (2026-09-01)

**Verdict: SURVIVES, ★★★★, conditional on the week-1 step.** The idea is a
method, the cell is empty at Level 4 after a second independent search, and the
first step costs under 2 GPU-h on features our own pipeline encoded today.

**Loud problem first, and it changes the experiment.** The audit's proposed
first step measures the wrong side of the model. Our own probe already shows
that role information is at chance in the pooled *image* embedding: 0.5228 for
CLIP ViT-B/32, 0.5289 for SigLIP 2, 0.5437 for OpenCLIP B/32. There is nothing
there for the map to destroy. The same probe shows role information is strong in
the *text* embedding: 0.6926, 0.7861 and 0.7770 on the same three models. The
test has to run on the text side. Details in §5.

**Second loud problem.** The audit says the source code is "permissively laid
out". It is not. The repository has **no licence file at all**. We may read it
and compare against it. We may not fork it or copy code from it.

---

## Words used on this page

- **Method** — the owner's definition: a new mechanism, an improvement on a
  current method, or an old method used on a new problem. A diagnosis, an
  ablation, a replication, or a statistic alone is not a method.
- **Embedding** — the list of numbers a model produces for one image or one
  sentence.
- **Orthogonal map Q** — a pure rotation of a space. It turns every embedding
  by the same amount. It never stretches or squashes anything, so it keeps
  every angle and every distance exactly.
- **Orthogonal Procrustes** — the standard recipe that finds the single
  rotation bringing one set of points closest to another set. It has a
  closed-form answer from one matrix decomposition. No training is involved.
- **Role information** — who does what to whom. "The dog chases the cat" is not
  "the cat chases the dog". The two sentences use the same words.
- **Decodability** — how well a small linear classifier can read one fact out
  of an embedding. 0.50 is chance. Our pre-registered bar for "the fact is
  there" is a 95% confidence interval whose lower end is above 0.55.
- **Confidence interval (CI)** — the range of values the data still allow.
- **Scoop level** — how much of an idea is already published. Level 5 means
  nobody has it. Level 1 means it is fully published. An idea must beat Level 2.
- **Strata** — our two role test sets. The **spatial** set has 6,868 items and
  is large. The **verb** set has 279 items and is small.

---

## 1. The method filter, and the route

**As written in the scan, it fails.** "Find where Q fails on compositional
tests" is a diagnosis. Diagnosis is excluded by name.

**After the smallest reframing it passes on route 1, a new mechanism.** Treat Q
as an operator and repair it. The mechanism is a **role-anchored Procrustes
fit**: fit the rotation under an extra requirement drawn from role-swapped
sentence pairs, so the rotation must keep those pairs apart while it still lines
up everything else.

The claim has a negative half and a positive half.

- **Boundary.** Moving your stored embeddings into a new model's space with the
  published rotation keeps category accuracy and silently loses role
  information. That is a real cost of the upgrade path the source paper sells.
- **Repair.** A fit with role anchors added recovers it, with no training and
  almost no extra data.

**The method paper this unlocks, named now.** It is the method half of the
role-decodability programme. [[Unified-Direction-Ranking-2026-08]] Part 1
records an open owner decision: the probe work produced a measurement and needs
a mechanism. This is the mechanism. The released artifact is the fitting code
plus the role-anchor sets, which serve both halves.

---

## 2. What the source paper actually claims, read from the paper

The source is
[Canonicalizing Multimodal Contrastive Representation Learning](https://arxiv.org/abs/2602.17584)
(Sharut Gupta and Sanyam Kansal, with Stefanie Jegelka, Phillip Isola and Vikas
Garg; posted 2026-02-19; 78 pages). Project page:
[canonical-multimodal.github.io](https://canonical-multimodal.github.io/)
(verified live, HTTP 200).

Read from the full text, and all of it verified:

- **The model pairs are three.** "(i) CLIP ViT-B/32 (OpenAI) and CLIP ViT-B/32
  trained on LAION-400M; (ii) CLIP ViT-L/14 (OpenAI) and SigLIP; and (iii) CLIP
  ViT-L/14 (OpenAI) and FLAVA." SigLIP 2 is never used.
- **The datasets are five, all category tests.** Oxford-IIIT Pets, CIFAR-100,
  Caltech-101, STL10 and DTD.
- **The fit is centred.** "We therefore fit and apply Q on centered embeddings,
  and then re-add the target mean." This detail is load-bearing and the audit
  omitted it.
- **A few anchors are enough.** Accuracy "essentially saturates once N reaches a
  modest value (around 10-15 classes)".
- **Their retrieval is class-level, not instance-level.** "top-1 retrieval
  across models, evaluated for both image–image and text–text retrieval by
  nearest-neighbor matching at the class level." So even the audit's "plain
  retrieval" control measures a different thing from theirs.
- **Mismatched sizes are already handled, in Appendix E.8.** They align "CLIP
  ViT-B/32 (d=512) with CLIP ViT-L/14 (d=768) using a rectangular Q projected
  onto the Stiefel manifold, enforcing Q⊤Q ≈ I". This matters for us: our cached
  SigLIP 2 features are 768 numbers wide and our CLIP B/32 features are 512
  wide, so a plain square rotation does not exist between them. Their appendix
  gives the exact recipe.
- **They already tried looser maps, in Appendix E.9.** They compare the rotation
  against "an unconstrained linear map" and "a small MLP with a residual
  connection", and find the rotation wins on the metrics that matter. **Every
  alternative they tried is more flexible than the rotation. They never tried a
  tighter one.** Our method goes in the direction they left empty.

**The gap, in their own words.** From the Limitations paragraph, quoted exactly:

> "Our evaluation focuses on classification-style semantics; we do not establish
> gains for fine-grained retrieval or dense ranking. Although an orthogonal map
> preserves angles, we do not test whether fine-grained attributes remain easily
> decodable after alignment; a natural next step is to train lightweight
> decoders on the aligned space."

That second sentence is our programme, written into their paper as future work.

**No compositional benchmark appears anywhere.** Searching the full text for
Winoground, SugarCrepe, ARO and VALSE returns zero hits. The word
"compositionality" does appear, but it means composing two maps
(Q_BC after Q_AB versus Q_AC). It never means compositional language.

---

## 3. Second scoop pass — independent, with new vocabulary

The audit ran one pass. This is a second, run from scratch, and it found a
paper family the first pass missed.

### 3.1 Citation graph, re-checked live today

[2602.17584](https://arxiv.org/abs/2602.17584) still has **exactly one
citation**, confirmed through the Semantic Scholar API on 2026-09-01. Six months
old, one citation. The cell is quiet.

### 3.2 The family the first pass missed

The audit searched with our words: "compositional", "binding", "attribute
decodability". The idea of fitting a closed-form map between two frozen models'
latent spaces lives in a different community that uses different words:
**latent-space translation and model stitching**. Searching with their words
surfaced a line of work the audit never saw.

- [Relative representations enable zero-shot latent space communication](https://arxiv.org/abs/2209.15430)
  (Moschella et al., ICLR 2023, 199 citations).
- [Latent Space Translation via Semantic Alignment](https://arxiv.org/abs/2311.00664)
  (Maiorca, Moschella, Norelli et al., NeurIPS 2023, 52 citations). Its abstract:
  representations "can be translated between different pre-trained networks via
  simpler transformations than previously thought", estimated with "standard,
  well-understood algebraic procedures that have closed-form solutions", enabling
  "effective stitching of encoders and decoders without additional training".
- [Latent Space Translation via Inverse Relative Projection](https://arxiv.org/abs/2406.15057)
  (2024) and
  [Improving Relative Representations with Learned Anchors and Whitened Inner Products](https://arxiv.org/abs/2605.30596)
  (2026).
- [When Embedding Models Meet: Procrustes Bounds and Applications](https://arxiv.org/abs/2510.13406)
  (Maystre et al., 2025), which the source paper itself cites.

**Why this matters and why it does not kill us.** It means "fit a closed-form
map between two frozen models" is an established recipe, not a new one. Gupta et
al.'s contribution is the multimodal version with theory: one rotation, fitted
from images alone, that lines up both towers. **Not one paper in this family
asks what the map destroys, and not one fits the map under a structured
constraint.** So the family is a citation obligation, not an overlap. But our
writeup must open against the family, not against Gupta alone, or a reviewer
will say we missed the ancestors.

### 3.3 A second adjacent family: keeping a vector store working after an upgrade

This is the practical claim the source paper sells, and it has its own
literature that the audit missed.

- [Backward-Compatible Aligned Representations via an Orthogonal Transformation Layer](https://arxiv.org/abs/2408.08793)
  (ECCV Workshops 2024) — an orthogonal transformation used for exactly this.
- [Drift-Adapter: A Practical Approach to Near Zero-Downtime Embedding Model Upgrades in Vector Databases](https://doi.org/10.18653/v1/2025.emnlp-main.805)
  (EMNLP 2025).

Neither is multimodal and neither measures relational structure. They are
required related work for the practical half of our claim.

### 3.4 Most-recent-ten-weeks pass (2026-06-20 to 2026-09-01)

Run on three queries restricted to 2026. **Nothing in the window occupies the
cell.** The two strongest hits in the whole 2026 sweep were both older than the
window and both needed a deep read. They are §4.

---

## 4. Deep reads of the top three threats

### Threat 1 — Compositional Generalization Requires Linear, Orthogonal Representations in Vision Embedding Models

[2602.24264](https://arxiv.org/abs/2602.24264) (Arnas Uselis, Andrea Dittadi,
Seong Joon Oh; v2 dated 2026-07-06). Code at
[oshapio/necessary-compositionality](https://github.com/oshapio/necessary-compositionality).
**The audit missed this paper entirely.**

**Why it looks like a threat.** The title pairs "orthogonal" with
"compositional", on vision-language embedding models, and the empirical section
covers CLIP, SigLIP and DINO.

**Why it is not, read from the paper.** Its "orthogonal" means something else.
It is about the geometry *inside one model*: whether the per-concept parts of an
embedding are orthogonal to each other. Quoted from the abstract:
"representations must decompose linearly into per-concept components, orthogonal
across concepts". Our "orthogonal" is a rotation *between two models*. The two
uses share a word and nothing else. The paper fits no cross-model map.

**But it hands us the hardest question in this gate, so we state it plainly.**
See §5.1.

### Threat 2 — How can embedding models bind concepts?

[2605.31503](https://arxiv.org/abs/2605.31503) (Uselis, Koishigarina, Oh;
2026-05-29). Same group. Also missed by the audit.

**What it does, from the abstract.** It studies "the binding function, which maps
concepts to scene embeddings" inside CLIP, and finds "scene embeddings decompose
additively into object representations", so that "object information is
recoverable from its image and text embeddings separately" even though CLIP acts
like a bag of concepts in cross-modal retrieval.

**Why it does not close our gap.** It is a single-model study of object-attribute
binding. It fits no map between models, and it studies which colour goes with
which shape rather than who acts on whom. It is a required citation and the
closest independent statement of "the information is present but does not reach
the answer".

### Threat 3 — Back into Plato's Cave

[2604.18572](https://arxiv.org/abs/2604.18572) (Koepke, Zverev, Ginosar, Efros;
2026-04-20, revised 2026-06-02). The single paper citing our source.

**Its finding, quoted:** "Coarse agreement persists but fine-grained agreement
does not."

**Why it closes no gap, verified from the full text.** It measures mutual
nearest-neighbour agreement between separately trained encoders as the gallery
grows. It fits no map. Its own related-work paragraph puts Gupta et al. in a
separate line: "Finally, Gupta et al. [33] show that an orthogonal map can map
between independently trained multi-modal contrastive models." It runs no
compositional benchmark.

**The risk it creates is rhetorical.** A reviewer may say fine-grained failure
is already known. Our answer is one sentence: they show that *neighbour lists*
disagree between two models, while we show that a *fitted rotation* passes
category tests and fails relational ones, and then we repair it.

### One more check the brief required: our own killed list

[[Unified-Direction-Ranking-2026-08]] Part 3 killed "compositional merging" at
Level 2 against
[AlignMerge 2512.16245](https://arxiv.org/abs/2512.16245), recorded as row M4 in
[[Method-Gates-Wave-2-2026-08]]. The shape is close enough that we read
AlignMerge in full before writing this page.

**It is a different paper in every axis that matters.** It merges the *weights*
of several fine-tuned **large language models**, and its "alignment" means
**safety alignment**, scored with a toxicity-style index. Our operator changes no
weights, runs between two frozen image-text encoders, and our property is
relational role binding.

**What is genuinely shared is the recipe pattern**: notice that an operation
preserves the headline number while quietly losing a second property, then add a
term to the fit that protects it. That pattern is public. A reviewer can say so.
Our defence is that the pattern is not the contribution; the specific measurement
and the closed-form repair are. **This is a real reviewer risk and it goes in the
prereg risk list, not in a footnote.**

### Scoop verdict

**Level 4.** One axis matches for the nearest work, and different works match on
different axes. No single paper matches on two.

---

## 5. The two facts that reshape the experiment

### 5.1 A rotation cannot destroy anything, so the whole claim rests on the residual

This is the deepest issue in the gate and the audit does not mention it.

If Q were an **exact** rotation, role decodability after Q would be **exactly
unchanged**. The reason is simple. A linear probe is a direction in the space.
Rotate the data and you can rotate the probe with it, and every score stays the
same. Rotations preserve angles and distances, so they preserve everything a
linear reader can see. The source paper's own sentence hedges in exactly this
spot: "Although an orthogonal map preserves angles, we do not test whether
fine-grained attributes remain easily decodable after alignment."

**So what could possibly drop?** Only the part the fitted map gets wrong. Two
independently trained models are not *exactly* related by a rotation, and Q is
fitted from a finite anchor set. What is left over is the residual. **Our real
hypothesis is that the residual is lopsided**: it is small along the
high-variance directions that carry category identity, and large along the
low-variance directions that carry role information. That is why classification
could survive while roles die.

This matters in three ways.

1. It makes the mechanism statement honest. We are not claiming rotations
   destroy relations. We are claiming the *fitted* map does, in a specific and
   measurable way.
2. It tells us what to measure. Measuring a probe trained and tested on the same
   mapped data would be close to invariant and would show nothing. We must
   measure **transfer**: train the probe in the target model's own space, then
   test it on source embeddings carried across by Q. That is where the residual
   bites, and it is exactly the "train lightweight decoders on the aligned
   space" the source paper names.
3. It makes the repair well posed. Adding role anchors tilts the fit toward the
   directions the plain fit ignores.

**If the residual turns out to be even, the direction dies on day one.** That is
the point of the kill number.

### 5.2 The image side has no headroom; only the text side can be tested

Our own probe, re-run and reproduced to four decimal places this afternoon in
`cropdistill/runs/role_probe_review_20260901/full/review-report.md`:

| Model | Role decodability, pooled image | Role decodability, text embedding |
|---|---|---|
| CLIP ViT-B/32 (OpenAI) | 0.5228 | 0.6926 |
| SigLIP 2 ViT-B/16-256 | 0.5289 | 0.7861 |
| OpenCLIP ViT-B/32 (LAION-2B) | 0.5437 | 0.7770 |

Chance is 0.50 and the decodability bar is 0.55. **All three image numbers are
below the bar. All three text numbers are far above it.**

The audit's step reads the image side. There is at most 4 points of headroom
there, inside the noise of a small effect. **The experiment must read the text
side, where there are 19 to 29 points to lose.** This is the single most
important correction on this page.

Per-stratum text numbers of record for CLIP ViT-B/32, from
`runs/role_probe/20260805-verb/full-gpu/loci.md`, pair split:

| Stratum | n | Text-embedding role decodability | 95% CI |
|---|---|---|---|
| spatial | 6,868 | 0.6907 | 0.6837 – 0.6976 |
| verb | 279 | 0.7409 | 0.7104 – 0.7710 |

---

## 6. Live verification of every load-bearing fact

Every item below was checked today, 2026-09-01.

| Fact | Status |
|---|---|
| Source paper [2602.17584](https://arxiv.org/abs/2602.17584) | Exists, 78 pages, full text read |
| Its citation count | **Exactly 1**, via Semantic Scholar API |
| Project page [canonical-multimodal.github.io](https://canonical-multimodal.github.io/) | Live, HTTP 200 |
| Code [Sharut/canonical-multimodal-rep](https://github.com/Sharut/canonical-multimodal-rep) | Exists. 15 stars, 1 fork, last push **2026-02-25**. Files: `main.py`, `few_anchors.py`, `models.py`, `datasets.py`, `utils.py` |
| Its licence | **None. No licence file.** GitHub API returns `license: null` |
| [openai/clip-vit-base-patch32](https://huggingface.co/openai/clip-vit-base-patch32) | Exists, ungated, 20.5M downloads |
| [timm/ViT-B-16-SigLIP2-256](https://huggingface.co/timm/ViT-B-16-SigLIP2-256) | Exists, ungated, Apache-2.0, 84k downloads |
| [laion/CLIP-ViT-B-32-laion2B-s34B-b79K](https://huggingface.co/laion/CLIP-ViT-B-32-laion2B-s34B-b79K) | Exists, ungated, MIT, 3.4M downloads |
| [timm/vit_base_patch32_clip_224.laion400m_e32](https://huggingface.co/timm/vit_base_patch32_clip_224.laion400m_e32) | Exists, 15.5k downloads. This is the LAION-400M B/32 the source paper used, reached in open_clip with `pretrained='laion400m_e32'` |

**Cached features, checked file by file.** The audit lists four paths. All four
exist, but three of them are not what the audit implies.

| Path | What is really in it |
|---|---|
| `cellA_pilot/20260806-preencode-b32/features_b32.npz` | CLIP ViT-B/32 (OpenAI). 2,360 images, 29,293 captions. **Pooled only** — the manifest says "B/32 patch tokens NOT cached" |
| `cellA_pilot/20260806-preencode/features.npz` | SigLIP 2 ViT-B/16-256, same battery |
| `cellA_pilot/20260806-preencode-text/text_features.npz` | **SigLIP 2** text, 29,293 × 768. Not CLIP text, as the audit's phrasing suggests |
| `cellA_pilot/20260806-preencode-winoground/features.npz` | SigLIP 2, 800 pooled and 800 × 256 × 768 tokens, over 400 Winoground examples |
| `cellA_pilot/20260806-preencode-wino-text/wino_text_features.npz` | Both models' caption embeddings: SigLIP 2 at 800 × 768 and B/32 at 800 × 512 |
| `role_probe/20260803-corpus`, `role_probe/20260805-verb` | Probe **results**, not embeddings. The run config says `fresh_encode: true`, so the role corpora are re-encoded each run |
| `/anvil/projects/x-cis261253/datasets/role_probe_corpus_20260803/` | The corpora themselves: `vismin_relation`, `vg_real_verb`, `vg_real_verb_v2`, each with `images/`, `items.jsonl`, `manifest.json` |

**The asset that makes this nearly free, and the audit did not know about it.**
The review run at `cropdistill/runs/role_probe_review_20260901/` encoded the role
corpora **today** with all three models we need, and its reproduction gate passed
on all three. The encoding path for CLIP B/32, SigLIP 2 and OpenCLIP B/32
LAION-2B is proven working as of this afternoon. The embeddings themselves are
not kept on disk, but one re-encode of the full corpus took about 14 minutes on
one GPU in the August run.

**Probe code.** `src/cropdistill/probe/run_corpus.py`, function `build_loci`,
takes plain numpy arrays for each readout point. Adding a Q-mapped readout point
is a new entry in that list, not new machinery.

---

## 7. The week-1 decisive step, written exactly

**One day. Under 2 GPU-h. No training.**

### 7.1 Which two models

**Main pair: CLIP ViT-B/32 (OpenAI) and OpenCLIP ViT-B/32 (LAION-2B).** Both are
512 numbers wide, so Q is a true square rotation, which is the exact object the
source paper claims. This is their model pair (i), with LAION-2B standing in for
LAION-400M. Both were encoded on our role corpora today and both reproduce.

**Second pair, for the mismatched-size case: CLIP ViT-B/32 (OpenAI) into SigLIP 2
ViT-B/16-256**, 512 into 768, using the rectangular Stiefel-manifold fit of their
Appendix E.8. This pair reuses the cached SugarCrepe and Winoground features
directly.

### 7.2 How Q is fitted

Copy their recipe, which we reimplement ourselves because their repository has no
licence. This follows the precedent set for CLIPSelf in
[[Method-Gates-2026-08]] Gate 1: evaluate it, do not fork it.

1. Take paired **image** embeddings only. No text is used in the fit. This is
   their central claim and we must not weaken it.
2. Subtract the mean of each side.
3. Solve the orthogonal Procrustes problem in closed form: one singular value
   decomposition of the cross-covariance matrix, then Q = U Vᵀ.
4. Apply as `z → Q(z − source mean) + target mean`, with means taken per
   modality, exactly as their Training paragraph specifies.

**Number of samples: 1,000 image anchors for the headline fit.** Then sweep
N in {128, 256, 1000, 2360} and report the whole curve. This sweep is not
optional. It is the arm that answers the cheapest rival explanation, which is
"you just did not use enough anchors" (§7.5).

### 7.3 What is measured, before and after

Three things, on the same items, native versus carried across by Q.

**(a) Role decodability, the headline.** Use the existing probe on our two
strata, at the text readout point. Train the linear probe on the **target**
model's own text embeddings. Test it two ways: on the target model's own held-out
text embeddings, which is the ceiling, and on **source** text embeddings carried
across by Q, which is the transfer number. Run both directions. Report the
spatial stratum (n = 6,868) and the verb stratum (n = 279) separately, on both
the pair split and the entity split, with the paired bootstrap already in the
pipeline.

**(b) Classification, the control that says the map works.** Zero-shot accuracy
on CIFAR-100, which is one of their five datasets, so our number is directly
comparable to their published one. Plus their own paired-instance cosine
similarity between mapped and target embeddings.

**(c) Retrieval and compositional accuracy.** Image-to-text Recall@1 on the 1,542
cached SugarCrepe++ images, native versus mapped. Then SugarCrepe++ and
Winoground through the locked evaluator, native versus mapped.

The shape we are looking for is a **dissociation**: (b) and the retrieval half of
(c) barely move, while (a) falls hard.

### 7.4 The kill number, fixed before the run

**Decided on the spatial stratum, because it is the only one with the power to
decide.** Its confidence interval is about 0.7 points wide at n = 6,868. The verb
stratum at n = 279 has an interval about 6 points wide, so it can confirm a large
drop and nothing smaller. The verb stratum is confirmatory. It never decides.

Let **D** be the drop in role decodability, from the target model's own ceiling
to the transfer number, on the spatial stratum, with a 95% paired bootstrap
interval.

- **DEAD if the whole interval for D sits below 2 points.** Plain Q carries
  relational structure as well as it carries category structure. There is no
  boundary, so there is nothing to repair, and the direction closes the same day.
  This is the outcome the invariance argument in §5.1 predicts if the residual is
  even.
- **MEASUREMENT NOTE ONLY if D lands between 2 and 10 points.** Real but small.
  It becomes one paragraph in the TMLR binding paper. It does not become a
  method paper, and no further compute is spent.
- **ALIVE if D is at least 10 points and its interval excludes 2, and at the same
  time CIFAR-100 zero-shot accuracy and image-to-text Recall@1 each drop by less
  than 2 points.** That is the dissociation, and only then do we build the
  repair.

The second half of that rule is as important as the first. A drop in role
decodability that comes with a drop in classification is not a boundary. It is
just a bad fit.

### 7.5 The arm that can kill the method half, run in the same day

**More anchors.** Refit Q with every image we have, then re-measure. **If the
role drop shrinks below 2 points once the anchor set is large, there is no method
here, only advice to use more anchors.** In that case the boundary finding may
still stand as a warning about small anchor sets, but the repair is dead and the
direction falls to a note. This is pre-registered now, not decided later.

### 7.6 Cost

| Item | Cost |
|---|---|
| Fitting Q, every variant, every anchor count | Seconds, on CPU. One decomposition each |
| Re-encoding the role corpora with two models | About 0.5 GPU-h, using today's proven path |
| CIFAR-100 and the retrieval set | Under 0.5 GPU-h |
| Probes and bootstraps | CPU, minutes |
| **Week-1 total** | **Under 2 GPU-h** |
| Whole direction if it lives | 30–80 GPU-h, nearly all of it re-encoding a wider set of checkpoints for the generality claim |

Free on OrangeGrid. No new data. No human labelling.

---

## 8. The method, if the week-1 step says live

**Name: the role-anchored Procrustes fit.**

Plain Q answers one question: which single rotation brings the image anchors of
model A closest to those of model B. Our fit answers a second question at the
same time: keep the role-swapped pairs apart.

Build a set of role-swapped sentence pairs. For each pair, take the difference
between the two sentence embeddings, which is the direction that means "the roles
were swapped". Collect those difference vectors for the source model and for the
target model. Then solve

> minimise ‖A Q − B‖² + λ ‖D_A Q − D_B‖² over all rotations Q,

where A and B are the image anchors and D_A and D_B are the role-difference
vectors.

**The whole point is that this is still one decomposition.** Stack the
role-difference rows underneath the image-anchor rows, weighted by the square
root of λ, and the answer is the same closed-form Procrustes solution on the
stacked matrix. **No training, no gradient steps, one extra matrix.** That is
what makes "recovers it at no extra training and almost no extra data" a real
claim rather than a hope.

λ is chosen on a validation split from a grid fixed in the prereg. The
role-difference pairs used to fit are disjoint from every pair used to measure,
split on the existing pair and entity axes so that no entity leaks across.

### Baselines the paper must beat or match

1. **Plain Procrustes on image anchors.** The thing being repaired.
2. **Plain Procrustes with every available anchor.** The kill-arm of §7.5.
3. **Procrustes fitted on text anchors instead of image anchors.** Maybe the
   published method is only weak because it never looks at text.
4. **The unconstrained linear map and the small residual MLP.** Their own
   Appendix E.9 alternatives. They report these hurt the geometry; we must show
   ours does not.
5. **Full re-embedding with the target model.** The ceiling, and the expensive
   thing the whole upgrade path exists to avoid.
6. **Whitening before the rotation.** The standard stronger alignment baseline
   from the latent-translation family in §3.2.

### Venue

**CVPR 2027, roughly 2026-11-13.** ICLR's abstract deadline of 2026-09-18 belongs
to the RoboJudge audit and is too close for a project starting today. **ICML
2027, roughly 2027-01-28, is the fallback**, but the visibility hazard argues
against waiting that long.

---

## 9. Risks

**1. The invariance problem, and it is the biggest one.** A perfect rotation
changes nothing a linear probe can see. The entire finding depends on the fitted
map's residual being lopsided. If it is even, the direction dies. §7.4 tests this
first and cheaply, which is the right order.

**2. Visibility, and the clock is short.** The source paper's own limitations
paragraph names our next step in one sentence. Isola and Jegelka are famous and
the repository is public. Six months and one citation is a good sign. The most
likely person to close this gap is one of the authors. **Move in weeks, not
months.** Re-run the citation check before any commitment, and again at every
milestone.

**3. Licence.** No licence on the repository means all rights reserved by
default. Read it, compare against it, cite it. Do not fork it, do not vendor it,
do not copy a function from it. Procrustes is one decomposition; write it
ourselves.

**4. The "more anchors" explanation.** Pre-registered as a kill-arm in §7.5.

**5. The recipe-pattern objection.** "Preserve a property by adding a term to the
fit" is a public pattern, and our own Wave-2 gate killed a merging idea against
[AlignMerge](https://arxiv.org/abs/2512.16245) for occupying it. We answer with a
different operator, a different property and a different measurement, but a
reviewer can still raise it.

**6. The small verb set.** 279 items detects large drops only. The spatial set
decides. Do not let a null on the verb set be read as a null overall, and do not
let a hit on it be read as confirmation on its own.

**7. Two ancestor families to cite properly.** The latent-translation line of
§3.2 and the backward-compatibility line of §3.3. Missing either invites the
"you rediscovered model stitching" review.

---

## 10. What the direction unlocks

**It turns the role-decodability programme from a measurement into a mechanism.**
That conversion is the open owner decision recorded in
[[Unified-Direction-Ranking-2026-08]] Part 1. The programme is complete and
powered on both strata, and it has been waiting for a method to belong to.

It also gives the TMLR binding paper a forward-looking section, and it produces
one reusable artifact: the role-anchor sets plus the fitting code, which any
group can drop into their own alignment pipeline.

---

## 11. Rating

**★★★★, conditional.**

The audit rated this ★★★★½. Half a star comes off for the invariance problem of
§5.1, which the audit did not identify and which can end the direction on day
one.

The conditional, decided by the week-1 step:

- **Rises to ★★★★½** if the spatial drop is 10 points or more with classification
  and retrieval intact, and the more-anchors arm does not close it.
- **Falls to ★★** and becomes a paragraph in the TMLR paper if the drop is
  between 2 and 10 points, or if more anchors close it.
- **Dies** if the drop is under 2 points.

Run the step this week.

## Related

[[Direction-Audit-2026-09-01]] · [[Method-Gates-2026-08]] ·
[[Method-Gates-Wave-2-2026-08]] · [[Unified-Direction-Ranking-2026-08]] ·
[[Binding-Root-Cause-Analysis]]

**No prompt-injection text was found on any page fetched for this gate.**

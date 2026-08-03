# Binding Root-Cause Analysis — why swap cases stay unsolved (2026-08-02)

First-principles memo (owner request: reason about the root, don't mine
patterns). The target failure: "dog chases cat" vs "cat chases dog" —
identical concept bags, different role bindings — still near chance after
dual-tower interactivity, local alignment, and data augmentation have each
been tried.

## 1. Sharpening the puzzle: capacity is NOT the root cause

For one quadruple (I_dc, I_cd, T_dc, T_cd), a rank-1 dot-product score
solves the swap trivially: we need only ⟨f(I_dc)−f(I_cd), g(T_dc)−g(T_cd)⟩
with the right sign — a linear constraint, satisfiable in low dimension.
Any finite set of swap constraints is satisfiable with enough dimensions.
So "a single vector cannot represent relations" is false at the instance
level. **The real requirement is SYSTEMATICITY**: solving *novel*
(A, R, B) combinations requires the embedding to implement a consistent
binding *scheme*, not per-pair memorized directions. The question is why
training never finds such a scheme.

## 2. Three candidate root causes (each with a discriminating prediction)

**(a) Data: the coupling never demands binding.** In web-scale pairs, the
concept bag almost always uniquely identifies the matching image — swap
pairs where both orders are plausible images are vanishingly rare. InfoNCE
learns minimal sufficient statistics for the discrimination actually
posed; bags suffice; binding gets no gradient. *Prediction:* swap-dense
paired-image training fixes it. *Status:* wounded — text-side hard
negatives (NegCLIP → [DeGLA (2504.16801)](https://arxiv.org/abs/2504.16801))
improve benchmarks but not swaps; the clean test needs paired swap IMAGES
(VisMin/COCO-Counterfactuals class — we hold filtered copies), which
nobody has run at scale.

**(b) Objective: feature suppression at the shared interface.** Contrastive
objectives provably suppress features redundant for discrimination
(feature-suppression / shortcut results in the contrastive literature;
alignment–uniformity has no structure term). Our own evidence fits
precisely: binding information is present pre-pooling (dense-routing probe
4.79 vs 0.15 global) and the text tower keeps word order unimodally while
the CROSS-MODAL map is bag-of-words
([LABCLIP, 2502.03566](https://arxiv.org/abs/2502.03566)) — suppression
happens exactly at the interface InfoNCE trains. *Prediction:* readout/
objective changes on frozen towers recover binding. *Status:* partially
supported ([TF_Local, 2604.11496](https://arxiv.org/abs/2604.11496);
generative scoring) for benchmark aggregates — but if fine-grained
readouts still fail *swap-hard* subsets, suppression is not the whole
story: pre-pooled features may carry co-occurrence, not roles.

**(c) Algebra: additive composition cannot bind, by construction.**
Bag-of-words is not a bug of training; it is the fixed point of an
ADDITIVE composition scheme: e_dog + e_chase + e_cat is COMMUTATIVE —
E(dog chases cat) = E(cat chases dog) exactly. Empirically CLIP text
embeddings are near-additive, and the binding-function analysis lane
([Tübingen, 2605.31503](https://arxiv.org/abs/2605.31503);
[2602.24264](https://arxiv.org/abs/2602.24264)) is forming around this.
The mathematics for non-commutative binding in fixed dimension has
existed since the 1990s: Smolensky's Tensor Product Representations
(role ⊗ filler), Plate's Holographic Reduced Representations (circular-
convolution binding — fixed-d, dot-product-compatible), and the VSA /
hyperdimensional computing family. bind(agent,dog) ⊕ bind(patient,cat)
and bind(agent,cat) ⊕ bind(patient,dog) are near-orthogonal in high d —
**swap sensitivity by construction, in a single cacheable vector scored
by a plain dot product.**

## 3. The idea this yields (to gate, not to assume)

**Role-bound contrastive embeddings (HRR/TPR-structured CLIP head).**
Text side: parse caption → role slots (agent / predicate / patient;
attribute–object) → bind fillers to fixed role vectors via circular
convolution → superpose into ONE vector. Image side: a light head over
frozen ViT patch tokens (our lane; our region machinery) that emits the
same algebra — object slots from patches, relation direction from
geometry. Score = dot product (1× inference, cacheable). Swap negatives
are generated ALGEBRAICALLY for free (re-bind roles — no swap images
needed), attacking root-cause (a) and (c) simultaneously; the readout
placement attacks (b). Hard part (and the honest research question):
whether image-side role assignment is extractable without dense
supervision — exactly what our pre-pooling evidence bears on.

## 4. The discriminating experiment (cheap, ours, runs before any method)

Linear role-decodability probe on swap-paired items (VisMin/COCO-CF in
`datasets/external_compositional`): is role assignment (who-is-agent)
linearly decodable from (i) patch tokens, (ii) pooled image embedding,
(iii) text embedding? 2×2×2 outcomes discriminate the three accounts:
decodable at (i) but not (ii)/(iii) → suppression at readout, algebraic
head on frozen towers is viable; not decodable even at (i) → encoders
never extract relations, mid-training required; decodable everywhere →
scoring/execution problem (Similarity-Is-Not-Logic camp). ~20–50 GPU-h on
cached/re-encoded features. This also sharpens Cell A: the probe is the
mechanism-level companion of the readout-ladder frontier.

## 5. Status — gated 2026-08-03: SURVIVES-NARROWED, ★★½

**Closest work: [OC-CLIP (Meta FAIR/Mila, 2502.14113)](https://arxiv.org/abs/2502.14113)**
— parses captions to scene graphs, binds ViT patches into object slots,
scores relations with a deliberately NON-commutative function, and trains
against algebraically-generated swap negatives (no swap images). It
occupies most of §3 — EXCEPT: its slots are text-conditioned and its score
is a structured sum → cross-encoder-shaped, no single cacheable vector.
**The binding-algebra × contrastive-ITM cell is empty** (43 HRR papers
swept; zero touch CLIP/ITM). Surviving delta: a FIXED non-commutative
operator inside each frozen tower, collapsing to one vector + plain dot
product at ~1×. Sharpest risk & cleanest claim:
[2604.16351](https://arxiv.org/abs/2604.16351) shows TRAINING for
compositional sensitivity costs dense-retrieval generalization (8–40%
drops, text-only) — by-construction algebra escaping that tradeoff is the
testable headline. Verified theory IDs: identifiability
[2303.09166](https://arxiv.org/abs/2303.09166); shortcut/feature
suppression [2106.11230](https://arxiv.org/abs/2106.11230),
[2011.02803](https://arxiv.org/abs/2011.02803); linear meanings
[2302.14383](https://arxiv.org/abs/2302.14383). HRR capacity bound:
[2301.10352](https://arxiv.org/abs/2301.10352). Caveat: OpenReview unswept
(APIs exhausted) — re-check before any prereg.

## 6. Probe outcome (2026-08-03) — suppression confirmed, with two caveats

Run as an **underpowered pilot** (pre-registered blocked trigger fired:
only 173 usable swap pairs < 300 floor; resolution limit 0.65 vs the 0.55
bar — but the headline contrast sits far outside it). n=173 pairs/346
images, grouped 5-fold, 5 seeds, 10k paired bootstrap, Holm.

- **Locus (i) frozen ROI/patch features: 93.2% (L6) → 99.0% (L9) →
  99.9% (L12)** linearly decodable role assignment.
- **Locus (ii) pooled image embedding: 52.5% — chance** (Holm p=.056).
- **Locus (iv) the model's own scoring axis: 50.3% — exactly chance.**
- Locus (iii) text: 67.5%, order-driven (scrambled control at chance).
- **The inversion:** the untrained region-text readout is ANTI-correlated —
  a parameter-free sign flip recovers **100.0%** at L12, and a
  roi_align-free localization audit shows entity similarity peaks in the
  OTHER entity's box (own>other in only 3–13% of slots). The frozen
  encoder has the answer; the native readout holds it exactly backwards.
  (Known CLIP dense-feature pathology family — cite, verify novelty
  before claiming; it directly motivates the A4/CLIPSelf-repair arm.)

**Caveat 1 — scope:** all 173 pairs are SPATIAL relations (on/left/right/
above/below…). Zero verb agent-patient swaps survive in our filtered
corpora (COCO-Counterfactuals: 0/11,194 — it is substitution, not
permutation; VisMin filtered lost ~97.5% of its relation subset to the
fail-closed provenance rule because those images are fully synthetic).
This probe settles **who-is-where**, not who-chases-whom.
**Caveat 2 — synthetic images** (diffusion, layout-conditioned, clean
backgrounds): locus (i)'s near-perfect number is likely an upper bound.

**Verdict pattern (spatial roles): (i)=1, (ii)=0, (iv)=0 → suppression at
the readout is the proximal mechanism; "encoders never extract roles" is
refuted in the spatial case.** Algebraic-binding kill-arm A CLEARS for
spatial roles; the verb-argument case is open pending data (below).

**Unblock decision (owner):** (a) use VisMin-bench relation — 622
human-verified exact role swaps, 1.5 GB, immediate — but it is the lab's
protected test-only corpus and fitting probes burns it; (b) re-stream the
full VisMin relation subset (~6,973 pairs, 39 GB) under a documented
waiver of the fail-closed rule (image contamination impossible by
construction — synthetic, no COCO ancestry; caption overlap to be
measured); (c) both, holding (a) out as the untouched confirmation set.
Worker and coordinator both recommend **(b) with (a) held out**.

**The convergence that matters:** the gate's kill-arm A — "can a head over
frozen ViT patches assign agent/patient WITHOUT the caption?" (~10 GPU-h,
likely to fire; OC-CLIP dodged it via caption-queried slots) — is the SAME
experiment as §4's discriminating probe. One ~20–50 GPU-h probe study
therefore simultaneously: (a) discriminates the three root-cause accounts,
(b) gates the algebraic method's load-bearing assumption, (c) supplies
Cell A's mechanism story. It is the single cheapest experiment that
advances every open VLM thread, needs no prereg (internal evidence study),
and runs on OrangeGrid now. Disposition: probe first; algebraic binding
stays benched at ★★½ behind Cell A (★★★½) unless the probe clears its
crux, in which case re-rate.

## Related

[[Method-Gates-Wave-3-2026-08]] · [[Prereg-Crop-Consistency-Distillation]]
· [[Status-And-Survivors]] · [[Wiki-Citation-Audit-2026-08]]

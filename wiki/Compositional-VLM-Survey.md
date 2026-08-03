# The Compositional VLM Problem — a working survey (2026-08-03)

What we know about why contrastive VLMs fail composition — from every paper
gated in waves 1–3, the SVIB post-mortem, and the A4/A5 decisive checks.
Written for the lab: claims carry links or our own numbers; verdicts carry
the gate that produced them. Companion: [[Binding-Root-Cause-Analysis]]
(the first-principles memo), [[Method-Gates-Wave-3-2026-08]].

## 1. The problem, precisely

CLIP-class dual encoders score image-text pairs by a dot product of pooled
embeddings. They pass retrieval benchmarks while failing *composition*:
attribute binding ("red cube, blue sphere" vs "blue cube, red sphere"),
relation direction ("dog chases cat" vs "cat chases dog" — the swap case),
and word-order sensitivity generally
([ARO/NegCLIP, 2210.01936](https://arxiv.org/abs/2210.01936);
[SugarCrepe, 2306.14610](https://arxiv.org/abs/2306.14610);
[Winoground, 2204.03162](https://arxiv.org/abs/2204.03162);
[SVO-Probes, 2106.09141](https://arxiv.org/abs/2106.09141)). Swap cases are
the hard core: a decade of fixes moved benchmark aggregates while swap
subsets stay near chance.

The sharpest framing (ours, §1 of [[Binding-Root-Cause-Analysis]]): for any
single swap quadruple, a rank-1 dot product solves the task trivially — one
linear constraint. **Capacity is not the root cause. Systematicity is**:
solving novel (A, R, B) combinations requires a consistent binding scheme,
and the question is why training never finds one.

## 2. Our own experimental evidence (the lab's contribution to the picture)

- **Binding information exists pre-pooling and dies at the readout.** Dense
  routing over patch tokens scores 4.79 vs 0.15 for the pooled global
  score (SVIB probes). Re-encoding regions at full resolution buys +2.66
  SugarCrepe++ at 8× cost; reusing the ViT's own ROI-pooled patch tokens
  loses −1.32 (paired CI [−2.51, −0.12]) — the region information is
  simply not carried into the patch tokens' pooled form.
- **Six clever readouts lost to the raw global score** under
  validation-locked selection (patch pooling, claim decomposition,
  dispersion, marginal calibration, optimal transport, inductive
  calibration). Cheap beats clever until the readout gets *training
  signal*.
- **A4 (2026-08-02): the dense-prediction ancestor actively destroys ITM.**
  [CLIPSelf (2310.01403)](https://arxiv.org/abs/2310.01403)'s released
  checkpoint lands 12.8 points BELOW the base patch arm on SCPP++ (CI
  [−14.1, −11.5]); its objective (region→crop alignment for detection)
  overwrites whatever compositional signal the tower had. Lineage caveat:
  FineCLIP (NeurIPS 2024, no arXiv) shows region self-distillation CAN
  help composition when the backbone trains — the failure is specific to
  the frozen-ITM setting and CLIPSelf's objective.
- **A5 (2026-08-02): the published 1× execution fix does not close our
  gap.** The [LABCLIP-style linear map (2502.03566)](https://arxiv.org/abs/2502.03566)
  replicates its paper's +5.2 SugarCrepe gain but closes none of the
  (A2−A1) gap on strict SCPP++ (negative on all seeds) — evidence that
  benchmark-level "execution fixes" and the region-information gap are
  different quantities.
- **Protocol lessons that shaped everything above:** α=1 reproduction
  invariant; QuickGELU pairing; validation-locked selection; the
  two-positive-caption instability of benchmark rankings.

## 3. What people tried — five families, and where each stands

**(a) Data: hard negatives and augmentation.** NegCLIP
([2210.01936](https://arxiv.org/abs/2210.01936)) → LLM-generated negatives
([DeGLA, 2504.16801](https://arxiv.org/abs/2504.16801)), scene-graph-guided
negatives ([Structure-CLIP, 2305.06152](https://arxiv.org/abs/2305.06152)),
scene-graph supervision ([SGVL, 2305.06343](https://arxiv.org/abs/2305.06343)),
counterfactual image pairs (VisMin, COCO-Counterfactuals — we hold filtered
copies). *Progress:* solid benchmark gains (+3–5 typical). *Limit:*
text-side negatives teach caption-plausibility shortcuts; swap subsets
barely move; and [2604.16351](https://arxiv.org/abs/2604.16351) shows
training FOR sensitivity costs dense-retrieval generalization (8–40%
drops) — the data lever has a measured price.

**(b) Execution/readout fixes at 1× on frozen towers.** Linear text-side
maps ([2502.03566](https://arxiv.org/abs/2502.03566)), learned readout CNNs
over the patch×token cosine map ([DCSM, 2503.08723](https://arxiv.org/abs/2503.08723)),
training-free late interaction ([ABE-CLIP, 2512.17178](https://arxiv.org/abs/2512.17178)),
logic-constrained score editing (LCSE,
[2607.23052](https://arxiv.org/abs/2607.23052)), selective aggregation
against background-shortcut pooling
([LaSt-ViT, 2602.22394](https://arxiv.org/abs/2602.22394) — "lazy
aggregation," zero compositional evals yet). *Progress:* real, cheap,
retrieval-preserving. *Limit:* our A5 result — these fix benchmark
aggregates, not the region-information gap, and none is swap-headline.

**(c) Fine-grained interaction: pay compute at query time.**
Cross-attention ITM heads ([BLIP-2, 2301.12597](https://arxiv.org/abs/2301.12597)),
generative scoring ([DiffusionITM, 2305.16397](https://arxiv.org/abs/2305.16397)
at ~17×; [VQAScore, 2404.01291](https://arxiv.org/abs/2404.01291)), and the
current state of the art on frozen features:
[TF_Local (2604.11496)](https://arxiv.org/abs/2604.11496) — a 13M-param
fusion transformer over frozen patch+token embeddings, SugarCrepe
73.0→86.3 with the backbone frozen. *Progress:* large — composition is
mostly an interface problem given enough interaction compute. *Limit:*
O(|I|×|T|) per-pair cost, no cacheable embeddings, no efficiency analysis
anywhere in that literature (verified: TF_Local discusses cost zero times).

**(d) Structured/multi-vector representations.** Role-tagged component
sets with per-role MaxSim ([ComAlign, 2409.08206](https://arxiv.org/abs/2409.08206));
object-centric binding modules with non-commutative relation scores and
algebraic swap negatives ([OC-CLIP, 2502.14113](https://arxiv.org/abs/2502.14113)
— Meta FAIR; the strongest swap-targeted method to date, but its slots are
text-conditioned → cross-encoder-shaped); document-retrieval multi-vectors
([ColPali, 2407.01449](https://arxiv.org/abs/2407.01449);
[SaMer, 2607.04605](https://arxiv.org/abs/2607.04605);
[MetaEmbed, 2509.18095](https://arxiv.org/abs/2509.18095)) — which have
NEVER been evaluated on composition (verified, wave-3 Cell-A gate).
Cross→bi distillation exists but always to single vectors
([CPRD, 2407.07479](https://arxiv.org/abs/2407.07479);
[DCLIP, 2505.21549](https://arxiv.org/abs/2505.21549)).

**(e) Theory.** Contrastive objectives suppress discrimination-redundant
features ([2106.11230](https://arxiv.org/abs/2106.11230),
[2011.02803](https://arxiv.org/abs/2011.02803)); multimodal contrastive
learning identifies shared content under assumptions that plausibly exclude
roles ([2303.09166](https://arxiv.org/abs/2303.09166)); CLIP-space meanings
are near-linear/additive ([2302.14383](https://arxiv.org/abs/2302.14383)),
generalizing models use *multiplicative* binding
([2605.31503](https://arxiv.org/abs/2605.31503)), and compositional
generalization requires linear-orthogonal concept factors with surprisingly
low dimension ([2602.24264, ICML 2026](https://arxiv.org/abs/2602.24264)).
The Tübingen group owns this lane and is one empirical paper from several
cells we care about.

## 4. What is still open, and why

1. **Swap at 1× with cacheable embeddings.** Every swap-competent system
   (OC-CLIP, TF_Local, generative scorers) pays per-pair compute. Additive
   composition is commutative BY CONSTRUCTION — E(dog chases cat) =
   E(cat chases dog) under a bag scheme — so single-vector swap
   sensitivity requires a non-commutative binding algebra (HRR/TPR/VSA;
   capacity bounds: [2301.10352](https://arxiv.org/abs/2301.10352)).
   The algebra × contrastive-ITM cell is EMPTY (43 HRR papers, zero in
   ITM; wave-3 RB gate). The load-bearing unknown: image-side role
   extraction without text conditioning — OC-CLIP dodged it; nobody has
   measured whether frozen ViT features even contain it. ★★½ until the
   probe answers that.
2. **The efficiency–compositionality frontier is unmeasured** (Cell A,
   ★★★½, CVPR candidate): nobody has drawn binding-retention vs
   bytes-per-image; theory predicts no breakpoint; the compression line
   never evaluates composition. The method it unlocks —
   binding-preserving pooling via margin distillation — is also
   unoccupied.
3. **Which failures are even position-dependent?** No per-item image-side
   split of compositional benchmarks exists (Cell B, ★★½); aggregate
   evidence ([2503.17349](https://arxiv.org/abs/2503.17349): 0.2–2.7%
   shuffle drops) plus LaSt-ViT's lazy-aggregation mechanism suggest much
   of "composition" may be solvable by order-free readouts — meaning
   benchmarks under-test binding.
4. **The sensitivity–generalization tradeoff**
   ([2604.16351](https://arxiv.org/abs/2604.16351)): every training-based
   fix pays retrieval; whether an architectural (by-construction) fix
   escapes the tradeoff is open and testable.
5. **Benchmark validity itself:** rankings flip under a second valid
   positive caption (our lesson #4); memorization splits reorder
   leaderboards (CompLearn/CTB line); swap subsets are small. Any method
   claim needs subset-level, CI-carrying evaluation.

## 5. What is worth trying (ranked, with the decisive experiment first)

1. **The role-decodability probe** (running as of 2026-08-03; ~20–50
   GPU-h): is agent/patient linearly decodable from (i) frozen patch
   tokens, (ii) pooled image vector, (iii) text vector, on swap-paired
   data? One experiment discriminates the three root-cause accounts
   (data / objective-suppression / algebra), gates the algebraic method's
   crux, and supplies Cell A's mechanism story.
2. **Cell A: the readout-budget vs binding frontier + binding-preserving
   pooling** (★★★½, CVPR ~Nov 13) — our infra advantage, theory
   counter-prediction to test, method arm folds in our crop-distillation
   machinery. 1-day dynamic-range pilot first; Tübingen and the LaSt-ViT
   group are the scoop watch.
3. **Algebraic role-binding embeddings** (★★½, re-rate on probe outcome):
   fixed HRR/TPR operator in frozen towers, single cacheable vector,
   algebraic swap negatives; headline = escaping the 2604.16351 tradeoff
   by construction.
4. **If the probe says roles aren't in frozen features at all:** the
   honest conclusion is that no readout can fix binding — mid-training
   with role-bearing objectives becomes the only path, and the frozen-lane
   program narrows to Cell A's frontier measurement.

## Related

[[Binding-Root-Cause-Analysis]] · [[Method-Gates-Wave-3-2026-08]] ·
[[Method-Gates-Wave-2-2026-08]] · [[Prereg-Crop-Consistency-Distillation]]
· [[Status-And-Survivors]] · [[Wiki-Citation-Audit-2026-08]]

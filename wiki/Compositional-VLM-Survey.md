# Why Vision-Language Models Struggle to Combine Ideas (2026-08-03)

This page explains what we know about a major failure in contrastive
vision-language models (VLMs): they often notice the right objects and words
but connect them in the wrong way. The evidence comes from papers checked in
waves 1–3, the SVIB post-mortem, and the A4/A5 tests.

Claims link to a paper or give our own numbers. Each decision names the check
that produced it. Also read [[Binding-Root-Cause-Analysis]] and
[[Method-Gates-Wave-3-2026-08]].

## 1. The exact problem

Models such as CLIP use two separate encoders. One turns an image into one
vector. The other turns text into one vector. A dot product compares them.
These models do well at finding a matching image or caption, but often fail at
**composition**, which means putting ideas together correctly.

Examples include:

- **attribute binding:** “red cube, blue sphere” versus “blue cube, red
  sphere”;
- **relationship direction:** “dog chases cat” versus “cat chases dog”; and
- **word order** in general.

These failures appear in
[ARO/NegCLIP, 2210.01936](https://arxiv.org/abs/2210.01936),
[SugarCrepe, 2306.14610](https://arxiv.org/abs/2306.14610),
[Winoground, 2204.03162](https://arxiv.org/abs/2204.03162), and
[SVO-Probes, 2106.09141](https://arxiv.org/abs/2106.09141).

Role swaps are the hardest cases. Ten years of methods improved whole-test
averages, but scores on swap-only groups remain near chance.

Our clearest explanation appears in section 1 of
[[Binding-Root-Cause-Analysis]]. Any one swap case can be solved with a very
small dot-product model. The problem is not raw model size. The problem is
**systematicity**: using the same role rule for a new `(A, relationship, B)`
combination. Training does not reliably learn that rule.

## 2. What our experiments add

- **Role information exists before pooling but disappears at the final
  readout.** A detailed route over patch tokens scores 4.79, compared with
  0.15 for the pooled global score. Encoding each region again at full
  resolution improves SugarCrepe++ by 2.66 points but costs 8× more. Reusing
  ROI-pooled tokens from the ViT itself lowers the score by 1.32 points, with
  paired confidence interval `[−2.51, −0.12]`. The ViT's pooled patch form does
  not carry the needed region information.
- **Six complex readouts lost to the raw global score.** We tested patch
  pooling, splitting captions into claims, dispersion, marginal calibration,
  optimal transport, and inductive calibration. We chose settings only on
  validation data. A cheap readout wins until the readout receives its own
  training signal.
- **A4, 2026-08-02: a model trained for dense prediction actively hurts
  image-text matching.** The released
  [CLIPSelf (2310.01403)](https://arxiv.org/abs/2310.01403) checkpoint scores
  12.8 points below the base patch version on SCPP++, with confidence interval
  `[−14.1, −11.5]`. Its region-to-crop training helps detection but overwrites
  useful compositional information. Important limit: FineCLIP, NeurIPS 2024
  with no arXiv version, shows that region self-distillation **can** help
  composition when the backbone also trains. Our failure is specific to a
  frozen backbone, image-text matching, and CLIPSelf's training goal.
- **A5, 2026-08-02: a published 1× scoring fix does not recover our missing
  information.** A
  [LABCLIP-style linear map (2502.03566)](https://arxiv.org/abs/2502.03566)
  reproduces the paper's 5.2-point SugarCrepe gain. However, it closes none of
  the A2-minus-A1 gap on strict SCPP++; every seed is negative. Raising an
  overall benchmark score and recovering region information are different
  goals.
- **Protocol lessons:** keep the `alpha=1` reproduction check; pair models with
  the correct QuickGELU activation; choose settings only on validation data;
  and remember that adding a second valid caption can make benchmark rankings
  unstable.

## 3. Five kinds of attempted fix

### A. Better data and harder negative examples

This group includes NegCLIP
([2210.01936](https://arxiv.org/abs/2210.01936)), LLM-written negatives in
[DeGLA, 2504.16801](https://arxiv.org/abs/2504.16801), scene-graph negatives in
[Structure-CLIP, 2305.06152](https://arxiv.org/abs/2305.06152), scene-graph
training in [SGVL, 2305.06343](https://arxiv.org/abs/2305.06343), and paired
changed images from VisMin and COCO-Counterfactuals. We have filtered copies of
the last two.

**Progress:** usually 3–5 points on full benchmarks.

**Limit:** text-only negatives may teach the model to spot an unnatural
caption instead of understanding roles. Swap scores barely change. Also,
[2604.16351](https://arxiv.org/abs/2604.16351) finds that training for more
compositional sensitivity lowers dense-retrieval quality by 8–40%. Better
sensitivity has a measured cost.

### B. Cheap scoring fixes on frozen encoders

These methods keep the image and text towers frozen and aim for about 1× cost:

- linear maps on the text side
  ([2502.03566](https://arxiv.org/abs/2502.03566));
- learned CNN readouts over the patch-by-token similarity map
  ([DCSM, 2503.08723](https://arxiv.org/abs/2503.08723));
- training-free late interaction
  ([ABE-CLIP, 2512.17178](https://arxiv.org/abs/2512.17178));
- score editing with logic rules, called LCSE
  ([2607.23052](https://arxiv.org/abs/2607.23052)); and
- selective aggregation that avoids pooling background shortcuts
  ([LaSt-ViT, 2602.22394](https://arxiv.org/abs/2602.22394)). LaSt-ViT calls
  this “lazy aggregation” and has not yet tested composition.

**Progress:** these methods are cheap, produce real gains, and keep retrieval
quality.

**Limit:** our A5 result shows that they can raise the overall score without
recovering the missing region information. None has made role swaps its main
result.

### C. More image-text interaction at scoring time

This group spends more compute every time it compares an image and caption:

- cross-attention image-text matching heads such as
  [BLIP-2, 2301.12597](https://arxiv.org/abs/2301.12597);
- generative scoring, including
  [DiffusionITM, 2305.16397](https://arxiv.org/abs/2305.16397) at about 17× cost
  and [VQAScore, 2404.01291](https://arxiv.org/abs/2404.01291); and
- [TF_Local, 2604.11496](https://arxiv.org/abs/2604.11496), the best current
  method on frozen features. Its 13M-parameter fusion transformer uses frozen
  image patches and text tokens. It raises SugarCrepe from 73.0 to 86.3 while
  the backbone stays frozen.

**Progress:** large gains show that composition is mostly a problem at the
image-text interface when enough interaction is allowed.

**Limit:** cost grows with every image-text pair, written as
`O(|I| × |T|)`. The model cannot save one reusable image vector. This line of
work gives no efficiency analysis; TF_Local discusses cost zero times.

### D. Structured or multi-vector representations

Examples include:

- role-tagged parts with per-role MaxSim
  ([ComAlign, 2409.08206](https://arxiv.org/abs/2409.08206));
- object-based binding with order-sensitive relationship scores and generated
  swaps ([OC-CLIP, 2502.14113](https://arxiv.org/abs/2502.14113)). This Meta
  FAIR method is the strongest swap-focused method so far. Its object slots
  depend on the text, so it works more like a cross encoder;
- document-retrieval multi-vectors:
  [ColPali, 2407.01449](https://arxiv.org/abs/2407.01449),
  [SaMer, 2607.04605](https://arxiv.org/abs/2607.04605), and
  [MetaEmbed, 2509.18095](https://arxiv.org/abs/2509.18095). The wave-3 Cell A
  check found that none has tested composition; and
- methods that teach a two-sided interacting model to become a two-encoder
  model. [CPRD, 2407.07479](https://arxiv.org/abs/2407.07479) and
  [DCLIP, 2505.21549](https://arxiv.org/abs/2505.21549) do this only for single
  vectors.

### E. Theory about why the failure happens

- Contrastive goals can hide features that are not needed for choosing the
  right pair
  ([2106.11230](https://arxiv.org/abs/2106.11230),
  [2011.02803](https://arxiv.org/abs/2011.02803)).
- Theory says multimodal contrastive learning can find shared content only
  under assumptions that may leave out roles
  ([2303.09166](https://arxiv.org/abs/2303.09166)).
- Meanings in CLIP space are close to linear and additive
  ([2302.14383](https://arxiv.org/abs/2302.14383)).
- Models that generalize well use multiplication-like binding
  ([2605.31503](https://arxiv.org/abs/2605.31503)).
- Compositional generalization may need concept parts that are linear and
  orthogonal, but the required space can be surprisingly small
  ([2602.24264, ICML 2026](https://arxiv.org/abs/2602.24264)).

The Tübingen group leads this research area. One more empirical paper from
them could cover several questions we care about.

## 4. Important questions that remain open

1. **Can a model solve swaps at 1× cost with reusable vectors?** Every strong
   swap method—OC-CLIP, TF_Local, and generative scorers—spends compute on
   each image-text pair. Adding word vectors cannot show order because
   `E(dog chases cat) = E(cat chases dog)` under that rule. A single-vector
   method needs an order-sensitive binding rule such as HRR, TPR, or VSA. A
   capacity result appears in
   [2301.10352](https://arxiv.org/abs/2301.10352). The wave-3 check found zero
   work combining this algebra with contrastive image-text matching: 43 HRR
   papers, zero ITM uses. The key unknown is whether image roles can be found
   without text guiding the search. OC-CLIP avoids that problem, and nobody
   has measured whether frozen ViT features contain the roles. Rating: ★★½
   until the probe answers it.
2. **Nobody has measured the speed-versus-composition tradeoff.** Cell A rates
   ★★★½ as a CVPR candidate. No paper plots how much role information remains
   against bytes stored per image. Theory predicts no sudden breaking point,
   while compression papers do not test composition. This study could lead to
   binding-preserving pooling trained by margin distillation. Nobody has
   claimed that method.
3. **Which failures truly depend on position?** No benchmark splits items by
   whether the image-side error depends on position. Overall evidence from
   [2503.17349](https://arxiv.org/abs/2503.17349) shows only 0.2–2.7% loss when
   order is shuffled. LaSt-ViT's lazy-aggregation account also suggests that
   order-free readouts may solve much of what benchmarks call composition.
   The benchmarks may not test binding strongly enough. Cell B rates ★★½.
4. **Can a built-in structure avoid the sensitivity-versus-retrieval loss?**
   [2604.16351](https://arxiv.org/abs/2604.16351) finds that every training fix
   it tested loses retrieval quality. A role rule built into the architecture
   might avoid that tradeoff. This is open and easy to test clearly.
5. **Are the benchmarks themselves dependable?** Our rankings change when a
   second valid positive caption is used. Memorization-based splits from the
   CompLearn/CTB line also reorder leaderboards, and swap groups are small.
   Every method claim needs results by subset and confidence intervals.

## 5. What to try next, in order

1. **Run the role-decoding probe first.** As of 2026-08-03 it was running and
   expected to cost 20–50 GPU-hours. Test whether agent and patient roles can
   be decoded with a linear model from frozen patch tokens, the pooled image
   vector, and the text vector. One experiment separates the data, suppression,
   and algebra accounts. It also checks the key assumption of the algebraic
   method and gives Cell A its explanation of the mechanism.
2. **Run Cell A: readout budget versus role information, then test
   binding-preserving pooling.** Rating ★★★½; possible CVPR deadline around
   November 13. We already have useful infrastructure. Theory gives a clear
   prediction to challenge, and the method arm can reuse our crop-distillation
   tools. Start with a one-day pilot to check that the scores vary enough.
   Watch the Tübingen and LaSt-ViT groups for competing work.
3. **Reconsider algebraic role-bound vectors only after the probe.** Current
   rating ★★½. The idea uses a fixed HRR or TPR operator in frozen towers, one
   saved vector, and swaps made by changing the algebra. Its main claim would
   be avoiding the tradeoff found in 2604.16351 because the order rule is built
   in.
4. **If frozen features contain no roles, stop trying new readouts.** Training
   earlier layers with role-aware goals would be the only path. The frozen
   model program should then shrink to Cell A's measurement of the frontier.

## Related

[[Binding-Root-Cause-Analysis]] · [[Method-Gates-Wave-3-2026-08]] ·
[[Method-Gates-Wave-2-2026-08]] · [[Prereg-Crop-Consistency-Distillation]] ·
[[Status-And-Survivors]] · [[Wiki-Citation-Audit-2026-08]]

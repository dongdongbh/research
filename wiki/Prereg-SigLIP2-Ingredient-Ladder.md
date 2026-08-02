# Pre-registration: Does the SigLIP-2 Ingredient Ranking Survive Scale?

Status: **DRAFT v1, 2026-08-03 — for professor sign-off.** Engineering
(§6) starts on sign-off; the design locks at the week-2 engineering gate
(§8). Target venue: **ICML 2027** (deadline ~Jan 28, 2027 per our deadlines
table — confirm at lock; CVPR 2027 is the fallback if engineering lands
early).

Paper type: **training-science method study** (the deliverable is a
scale-conditioned recipe + upstreamed implementations). Sibling pages:
[[Prereg-1NFE-Diversity]], [[Prereg-RoboJudge-Audit]],
[[Prereg-Autoresearch-Accept-Rule]].

---

## 1. The problem, in plain language

Every lab that trains a vision-language encoder faces the same question:
which training ingredients matter? SigLIP 2 — the backbone we already use,
downloaded ~6.3M times/month — answered by *bundling*: "we extend the
original image-text training objective with several prior, independently
developed techniques into a unified recipe — this includes captioning-based
pretraining, self-supervised losses (self-distillation, masked prediction)
and online data curation." It shipped **no ablation isolating any
ingredient, at any scale.**

Worse, the field's standard practice — search recipes at small scale, keep
the winner, scale it up — rests on an assumption Beyer's own methodology
essay attacks: **small-scale architecture rankings routinely invert at
scale** ("for a 110-layer deep network, barely anything works except
strictly sticking to the identity residual"). If ingredient rankings invert
too, every cheap VLM recipe search in the literature is measuring the wrong
thing. Nobody has tested this for VLM recipes.

## 2. Current research state

- Only two 2026 papers engage the SigLIP-2 recipe at all — SigLIP-HD
  (2607.09488, a method on top) and "Similarity Is Not Logic" (2607.23052,
  inference-time) — **neither ablates ingredients**. No cross-scale ranking
  study of any VLM recipe exists (lane sweep, Aug 2026).
- A free hook sits in the paper itself: SigLIP 2 re-weights its auxiliary
  losses by **"0.25, 0.5, 1.0, and 0.5 for the B, L, So400m and g model
  sizes, respectively"** — an admitted, unexplained, *non-monotonic* scale
  dependence in ingredient weighting. The recipe's own authors hand-tuned
  what we propose to measure.
- Adjacent precedent that rank instability is real: DiffusionBench
  (2606.24888) trained 21 DiTs and found ImageNet-FID rankings
  **anti-correlate** with text-to-image rankings (Pearson −0.38 to −0.58) —
  benchmark-axis instability in the nearest lane.
- **Substrate:** big_vision is frozen (last push 2025-05-19; README
  disclaims maintenance) but ships the SigLIP-2 configs and all 16
  checkpoints — useful for validation, not training. **OpenCLIP is the live
  base** (pushed 2026-07-31, 14k★): SigLipLoss ✓, CoCa-style captioning ✓,
  NaFlex ✓, a designed `TrainingTask` extension seam, and a documented
  community request for SigLIP-2 training (issue #1041). **Missing, ours to
  build: EMA self-distillation (local-to-global), masked prediction, online
  data curation.** Data: DataComp-1B live.
- Why the lane stays open: the ladder costs ~1,000–1,400 H100-h — trivial
  for the recipe's owners (who have no incentive to ablate their own
  bundle) and prohibitive for most academic groups. **Delta's 8×H200
  30-day partitions are our moat** (~5,760 GPU-h per node-month).

## 3. Our method and novelty

A **cumulative ingredient ladder crossed with a scale ladder**, with the
*ranking* as the registered outcome:

- Configs C1–C5, matching SigLIP 2's staged order: sigmoid-only →
  +captioning decoder → +self-distillation → +masked prediction → +online
  curation.
- Scale rungs R1–R3: ViT-B/32 @128M samples (~3 H100-h/run), ViT-B/16
  @1.28B (~66 H100-h), ViT-L/16 @1.28B (~250 H100-h). All on DataComp-1B,
  matched samples-seen within a rung.
- Secondary: leave-one-out at R2 (marginal contribution vs cumulative
  position), and a captioning-weight mini-sweep at R1/R3 targeting the
  0.25/0.5/1.0/0.5 mystery.
- Eval battery: zero-shot IN-1k + COCO/Flickr retrieval (standard), **plus
  our compositional battery** (SugarCrepe++/Winoground, cached-feature
  infra) — the axis DiffusionBench says can invert and nobody measures for
  encoder recipes.

Novelty (verified): first per-ingredient ablation of the SigLIP-2 recipe at
any scale; first cross-scale rank-stability test for any VLM recipe; the
three missing OpenCLIP ingredient implementations as a community artifact.

## 4. Pre-registered design

**Primary registered outcome:** the config **rank order** per rung (Kendall
τ between rungs), on each eval axis separately. **The falsifier: does the
R1 ranking predict the R3 ranking?**

**Hypotheses (directional, locked):**
- **H1:** at least one adjacent-config pair swaps rank between R1 and R3 on
  at least one primary eval axis (predict TRUE — the Beyer thesis).
- **H2:** the optimal captioning-loss weight shifts with scale in the
  direction of SigLIP 2's own hand-tuning (predict TRUE).
- **H3:** the compositional-axis ranking differs from the IN-1k ranking at
  the same rung (Kendall τ < 0.8; predict TRUE — DiffusionBench analogue).
- **H4 (the useful null):** if no rank swap occurs anywhere, we register
  "VLM ingredient rankings are scale-stable to L/16 @1.28B" as the primary
  claim — it licenses small-scale recipe search and is arguably the more
  cited result.

**Seeds & decision rules:** 3 seeds at R1, 2 at R2, per-rung seed variance
reported **before** any ranking claim (no rank statement where the swap is
inside seed noise); bootstrap CIs on τ; Holm across H1–H3. Training
hyperparameters per config frozen at lock from SigLIP-2/OpenCLIP defaults —
no per-config tuning (tuning asymmetry is the classic ablation confound;
we state this rule in the paper).

**What we will NOT claim:** anything about So400m/g scales or WebLI-scale
data; that SigLIP 2's released checkpoints are suboptimal (different data,
different budget); anything about NaFlex (separate question, see the gate
record).

## 5. Expected outcomes

- **Rank inversion found (H1):** headline — "your small-scale VLM recipe
  search is measuring the wrong thing," plus the scale-conditioned recipe
  schedule (which ingredient enters at which scale) as the method — the
  un-derived thing SigLIP 2 hand-tuned.
- **Scale-stable (H4):** headline — the first license for cheap VLM recipe
  search, with measured bounds.
- Either way: three OpenCLIP ingredient implementations upstreamed (PR
  against issue #1041), the full ladder results database released, and the
  compositional-axis rankings the field has never had.

## 6. Resources and timeline (engineering-led)

**The long pole is engineering, not GPU-hours** — it starts now, in
parallel with the ICLR papers, since it consumes developer time, not the
clusters they need.

- Wk1–3 (now → late Aug): implement self-distillation + masked prediction
  + curation in OpenCLIP's TrainingTask seam; **parity gate** — reproduce
  SigLIP-2-B released-checkpoint evals within tolerance through our eval
  path.
- Wk4 (lock): design locked; R1 ladder launched (cheap, all configs × 3
  seeds).
- Sep–Oct: R2 on Delta H200-long; Nov: R3 (L/16) + leave-one-out; Dec:
  analysis + writing; Jan: submission. CVPR fallback decision point: Oct 15.

**Cost:** ~1,000–1,400 H100-h total. Cluster: **Delta gpuH200x8-long**
(30-day walls fit the R3 runs whole). Mind the proportional-charging rule
(request CPU/RAM ≤ GPU fraction). R1 runs fit OrangeGrid.

## 7. Risks and scoop watch

- **Tschannen/Zhai** own the recipe and could publish the ablation any
  time — but have had 18 months and no incentive; re-gate at the Wk4 lock
  and at each rung boundary.
- OpenCLIP community closing issue #1041 themselves — mitigated by
  upstreaming *our* implementations early (the artifact is also the claim
  stake).
- Engineering gate failure (self-distill parity) → descope to
  {sigmoid, captioning, curation} ladder; the claim narrows but survives.
- Delta allocation shortfall → drop R3 to B/16-with-2.56B-samples and
  re-word as data-scale ladder (the claim changes; pre-registered as the
  named fallback, not a silent switch).

## 8. Lock checklist

1. Professor sign-off on §4 (esp. no-per-config-tuning rule and H4 framing).
2. Parity gate passed (Wk2–3).
3. Confirmatory literature pass, most recent 8 weeks explicit.
4. ICML/CVPR 2027 exact dates confirmed. → LOCKED + git hash.

## Related

[[Unified-Direction-Ranking-2026-08]] · [[Prereg-1NFE-Diversity]] ·
[[GPU-Resources-Across-Clusters]] · [[Data-Transfer-Between-Clusters]]

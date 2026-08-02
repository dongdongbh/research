# Method Gates — 2026-08-03

Gates for the two REAL-method candidates (owner definition: a method is a new
approach — architecture, training procedure, representation, broken
assumption, changed data/workflow — not diagnosis/ablation/statistics).
Run directly (subagent budget exhausted); evidence from S2/HF/arXiv APIs +
full-page reads. Coverage caveat: single-pass sweep, arXiv/HF-heavy; run the
standard confirmatory pass (recent 8 weeks explicit) at prereg lock.

## Gate 1 — Crop-consistency distillation → **SURVIVES, ★★★★ conditional**

**The mechanism exists; the method target does not.**
- **CLIPSelf (arXiv 2310.01403, ICLR 2024)** is the ancestor: it "aligns a
  region representation extracted from its dense feature map with the
  image-level representation of the corresponding image crop" — crop→dense
  self-distillation. But: open-vocabulary **dense prediction** only
  (detection/segmentation), full ViT fine-tuning, **zero compositional/ITM
  evaluation**. Checked 100 citing papers for compositional applications:
  none applies the mechanism to caption-pair compositional matching.
- Nearest compositional competitor, **DeGLA (2504.16801)**, is a different
  mechanism (global EMA-teacher self-distillation to *preserve* knowledge
  during hard-negative fine-tuning + text-side IGC/TGC losses; +3.5% avg on
  VALSE/SugarCrepe/ARO). It becomes a **mandatory baseline**, not a scoop.
- No hit for crop-re-encoding→patch distillation on frozen backbones, nor
  for a structured multi-region teacher, in any 2024–2026 sweep query.

**Surviving novelty delta (one sentence):** distill full-resolution crop
re-encodings — and our *trained grid+self-attention multi-region teacher* —
into ROI-pooled patch features of a **frozen** backbone via a light adapter,
targeting measured compositional gains (+2.66 teacher / −1.32 patch gap,
our own paired-CI numbers) at ~1.1× inference, with retrieval preservation.

**What must be true to win:** the student closes ≥⅔ of the 1.32-pt gap
(ideally approaches the 8×-cost teacher). Named risks: distillation caps
below teacher; and the **aggregation-objection kill-arm** — "Similarity Is
Not Logic" (ICML 2026) claims binding failure is execution not
representation, so we pre-register: if an aggregation fix over raw patch
tokens closes the gap at 1×, our training method is unnecessary (that
comparison is an arm, and a kill criterion, not a threat).

**Baselines table:** raw frozen backbone · grid+self-attn @8× (upper
anchor) · patch-ROI @1.06× (lower anchor) · CLIPSelf checkpoint ROI-pooled
(repo `wusize/CLIPSelf`, 207★, frozen since 2024-02, **no license** —
evaluate, don't fork) · DeGLA · aggregation-fix @1×.

**Cost/venue:** 200–400 GPU-h (adapter training + our cached-feature
evals), OrangeGrid/Anvil. **ICLR 2027 feasible.**

**The condition on the star:** week-1 decisive check — run CLIPSelf's
released checkpoints through our compositional battery with ROI pooling.
If CLIPSelf *already* closes the compositional gap, our paper collapses
into a calibration of theirs (drop to ★★½, salvage = "what crop-distillation
does and doesn't transfer"). If it doesn't (expected — its objective is
dense-prediction alignment, not ITM), the lane is clean.

## Gate 2 — Epistemic contextualization (first implementation) → **SURVIVES, ★★★★**

**No implementation exists** (re-verified including recent weeks: no paper,
no LawZero code — their repo remains 404 and the hiring page still lists the
pipeline roles). The Scientist AI paper's own text stands: "contextualization
is specified solely at the level of requirements (i.e., we do not provide a
completed algorithm)."

**The dangerous neighbor family is tag conditioning, and it is distinct:**
- **Source-aware training (2404.01019, Khalifa et al.):** associates "unique
  source document identifiers with the knowledge in each document" then
  instruction-tunes to cite — **ID tags, synthetic data, attribution-only
  outcomes**; no text rewriting, no calibration/sycophancy/goal-adoption
  measurement. Their own emphasis on "pretraining data augmentation" being
  necessary is useful supporting evidence that data-side intervention is the
  active lever.
- CTRL-style control codes / conditional pretraining: prepended tags, not
  epistemic restructuring. GenProve (2601.04932) is generation-side
  provenance. None rewrites assertions into attributed-claim records
  (Bengio Def 3.22).

**Surviving novelty delta (one sentence):** the first pipeline that
*rewrites* pretraining text into epistemically scoped records ("X is true"
vs "S claimed X at T in V"), with a **tag-only-conditioning arm** that
separates rewriting from metadata conditioning — the exact control that
pre-empts the "metadata conditioning rediscovered" objection.

**Evaluation assets verified live:** ConflictBank (2408.12076, knowledge
conflicts), "How LLMs Balance Internal Knowledge with User and Document
Assertions" (2604.22193 — near-perfect fit), sycophancy suites, standard
capability battery for the tax. Placebo-rewrite is established hygiene
(2603.24826) — ours adds the tag-only arm as the scientific control.

**What must be true to win:** contextualized models beat raw AND tag-only
AND placebo on calibration-under-conflict + sycophancy at a small capability
tax; rewriter fact-fidelity passes a pre-registered sample-audit gate
(rewriting corrupting facts is the main technical risk; adjacent works
suggest manageable but we measure it first).

**Cost/venue:** 900–1,400 GPU-h (rewriter inference ~400+ H100-h dominates;
pretraining arms at 0.5–1.5B), Anvil H100 / Delta. **ICML 2027.** Window:
LawZero 6–12 months, NVIDIA-backed — re-gate at each milestone.

## Verdict

Both method candidates survive their gates. Proceed to pre-registrations:
crop-consistency distillation as the **ICLR method flagship** (replacing the
parallel-RL factorial, which moves to bench), epistemic contextualization as
the **ICML method flagship** (replacing the SigLIP-2 ladder, which moves to
bench). The judge-audit (ICLR) and 1-NFE (CVPR) diagnostics are unchanged.

## Related

[[Unified-Direction-Ranking-2026-08]] · [[Prereg-RoboJudge-Audit]] ·
[[Prereg-1NFE-Diversity]] · [[Status-And-Survivors]]

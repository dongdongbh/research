# Pre-registration: Epistemic Contextualization — the First Implementation

Status: **DRAFT v1, 2026-08-04 — for professor sign-off.** Pipeline
engineering starts on sign-off; design locks at the fidelity gate (§8).
Target venue: **ICML 2027** (~Jan 28 — confirm at lock).

Paper type: **METHOD** (owner definition: changed data/workflow — breaking
the assumption that all pretraining text is a ground-truth assertion). Gate
record: [[Method-Gates-2026-08]]. Companion CVPR diagnostic:
[[Prereg-1NFE-Diversity]].

---

## 1. The problem, in plain language

Language models are trained as if everything in the corpus were true. Every
rumor, sales pitch, and disputed claim enters the model the same way as a
physics fact — which is a plausible root of miscalibration on contested
topics, sycophancy (agreeing with the loudest source), and models "adopting"
goals expressed in text they merely read.

Bengio's Scientist AI program — the [Feb 2025 position paper
(2502.15657)](https://arxiv.org/abs/2502.15657) and the [Jun 2026 technical
paper (2606.29657)](https://arxiv.org/abs/2606.29657) — names the fix:
rewrite the corpus so every record distinguishes *"X is true"* from
*"source S claimed X at time T in venue V"*. The technical paper formalizes
it as **Def. 3.22** and states, verbatim: **"contextualization is specified
solely at the level of requirements (i.e., we do not provide a completed
algorithm)."** *(Citation corrected 2026-08-02: the quote and Def. 3.22 are
in 2606.29657, not the position paper — verified against both PDFs.)*
Eighteen months after the direction was named, and with the requirements
now formal for a month, nobody — including [LawZero](https://lawzero.org)
(no public code; pipeline team still hiring) — has built it. We build it,
and measure what it actually buys.

## 2. Current research state (gated 2026-08-03)

- **No implementation exists** (re-verified incl. the most recent weeks).
- **The dangerous neighbor is tag conditioning, and it is distinct:**
  source-aware training ([2404.01019](https://arxiv.org/abs/2404.01019)) associates *document ID tags* with
  content for attribution, on synthetic data — no rewriting, no
  calibration/sycophancy outcomes; [CTRL-style codes](https://arxiv.org/abs/1909.05858) likewise; [GenProve](https://arxiv.org/abs/2601.04932) is
  generation-side provenance. Nothing restructures the text itself into
  epistemically scoped records.
- Corpus rewriting at scale is established practice (densifying rewrites,
  ["Synthetic Rewriting as a Quality Multiplier"](https://arxiv.org/abs/2603.24826) with placebo controls) — so
  the *pipeline* is feasible and the placebo arm is hygiene; the epistemic
  schema and its measured effects are the contribution.
- **Ready evaluation assets (verified live):** ConflictBank ([2408.12076](https://arxiv.org/abs/2408.12076),
  knowledge conflicts), "How LLMs Balance Internal Knowledge with User and
  Document Assertions" ([2604.22193](https://arxiv.org/abs/2604.22193) — near-exact fit), standard sycophancy
  suites, standard capability battery for the tax.

## 3. The method

**Schema** (Def 3.22-inspired, ours to concretize): each document is
transformed into records where factual observations remain assertions and
communicated claims become attributed: source, venue, date, stance, and
scope markers — in natural language (no special tokens required), so any
architecture trains on it unchanged.
**Rewriter:** an open 4–8B model with constrained prompting + rule
validators; a **fact-fidelity audit** on stratified samples (LLM-judge +
human spot-check) with a pre-registered pass threshold.
**Corpus:** ~2B tokens — a [DCLM](https://arxiv.org/abs/2406.11794)/[FineWeb](https://arxiv.org/abs/2406.17557) subset enriched with a
contested-claims-rich slice (news/forums/reviews) so the intervention has
signal to change.
**Training regime (revised 2026-08-04, owner concern: from-scratch
pretraining too costly):** **continued pretraining (mid-training) on a
fully-open base model — [OLMo-2-1B](https://huggingface.co/allenai/OLMo-2-0425-1B)** — whose raw pretraining data is public,
so the pre-intervention diet is known. This is also the deployable form of
the method: no lab re-pretrains from scratch; contextualization would enter
real pipelines as a mid-training/annealing intervention. A from-scratch
arm (0.5B × 8B tokens) is retained as a **gated escalation** (§4), run only
if Phase-1 effects are positive.

## 4. Pre-registered design

**Arms (× 2 seeds = 8 continued-pretraining runs on OLMo-2-1B, 2B tokens
each):**
- C0 raw corpus.
- C1 **contextualized** (full epistemic rewriting).
- C2 **tag-only** — identical metadata prepended as tags, text unchanged.
  *This is the decisive control:* it separates epistemic rewriting from
  metadata conditioning and pre-empts the "source-aware training
  rediscovered" objection.
- C3 placebo rewrite — paraphrase at matched token budget and perplexity
  (controls for "any rewriting helps").

**Hypotheses (directional, locked):**
- **H1:** C1 beats C0, C2, and C3 on calibration-under-conflict
  (ConflictBank: accuracy + ECE on conflicting-source items).
- **H2:** C1 reduces sycophancy versus all controls.
- **H3:** C1 improves assertion-balancing behavior ([2604.22193](https://arxiv.org/abs/2604.22193) metrics)
  beyond C2 — i.e., rewriting adds over tags.
- **H4:** capability tax of C1 ≤ 2.0 points average on the standard suite
  vs C0.
- **H5 (escalation gate, replaces the old scale hypothesis):** if H1–H3
  are positive, the from-scratch escalation arm (0.5B × 8B tokens, 4 arms ×
  2 seeds, ~+300 GPU-h incl. extra rewriting) reproduces the C1 > C2
  ordering — testing whether the effect also forms without a raw-text
  pretraining history. Run ONLY on a positive Phase-1; its result extends
  the paper, it does not gate it.

**Decision rules:** bootstrap CIs over seeds; Holm across H1–H5; all
metrics and prompts frozen at lock; the fidelity threshold (≥97%
fact-preservation on audited samples) is fixed before any rewriting at
scale.

**Branch, not kill:** if C2 ≈ C1 on all primaries, the honest headline is
"metadata conditioning suffices — the cheap version of Scientist AI's data
mechanism," which is itself a method result (tags are ~free; rewriting is
not). Pre-registered as the alternative branch.

**Kill criteria:** (i) rewriter fails the fidelity gate after two
prompt-engineering rounds → stop and report the pipeline difficulty (a
short honest note, not a paper). (ii) LawZero or anyone ships an
implementation before our lock → re-gate within 48h; our tag-only control
and measured-effects framing likely survive as the *evaluation* of their
method, but the framing changes.

**What we will NOT claim:** any safety *guarantee* (we measure premises and
benefits, not the Scientist AI theorem); anything at frontier scale;
goal-adoption prevention beyond the probe suite's proxy.

## 5. Expected outcomes

- **Central:** contextualized models are measurably better calibrated under
  source conflict and less sycophantic at a small capability tax — the
  first empirical grounding for a widely cited but never-built proposal,
  with the pipeline, corpus, and checkpoints released.
- **Tag-suffices branch:** the community gets the cheap version validated
  and the expensive version honestly priced.
- **Null branch:** the first evidence that the Scientist AI data mechanism
  does not deliver its promised benefits at academic scale — publishable,
  and the field's only measurement either way.

## 6. Resources and timeline

**Cost (revised): ~300–450 GPU-h for Phase 1** — rewriting 2B tokens
through a 4–8B rewriter ≈ 150–250 H100-h; 8 continued-pretraining runs at
1B × 2B tokens ≈ 100 H100-h; evals ≈ 30. The gated from-scratch escalation
adds ~300 GPU-h only if Phase 1 is positive. **Clusters: OrangeGrid/Anvil
suffice for Phase 1** (Delta H200 optional for rewriter batch throughput;
no longer required). Engineering (schema, rewriter, validators, fidelity
audit) starts now — it needs no GPUs the ICLR papers want.

**Timeline:** Aug–Sep schema + rewriter + fidelity gate → Oct lock +
rewrite at scale → Nov pretraining arms → Dec evals + analysis → Jan
write-up and submission.

## 7. Risks and scoop watch

- **LawZero (NVIDIA-backed) shipping first** is the structural risk —
  window estimated 6–12 months; they cannot publish an honest negative
  about their own thesis, and they will ship a system, not a
  placebo-and-tag-controlled study. Watch: lawzero.org, their hiring page,
  citations of [2606.29657](https://arxiv.org/abs/2606.29657). Re-gate at every milestone.
- Rewrite fidelity is the main technical risk → gated first, before any
  scale spend.
- Effect sizes unknown (nobody has measured any of this) → scales and the
  contested-claims-rich corpus slice are chosen to maximize detectable
  signal; H5 guards against small-model artifacts.

## 8. Lock checklist

1. Professor sign-off on §4 (esp. the C2 tag-only control, fidelity
   threshold, branch-vs-kill framing).
2. Fidelity gate passed on the pilot slice.
3. Confirmatory literature pass, most recent 8 weeks explicit.
4. ICML 2027 exact deadline confirmed. → LOCKED + git hash.

## Related

[[Method-Gates-2026-08]] · [[Unified-Direction-Ranking-2026-08]] ·
[[GPU-Resources-Across-Clusters]] · [[Data-Transfer-Between-Clusters]]

# Pre-registration: Epistemic Contextualization — the First Implementation

**EXECUTION START (2026-08-10, owner-initiated: "we may run the basic
part on anvil, then move to delta for full training").** The three
readiness choices proceed on their recommended defaults unless the
owner/professor overrides: (1) revised budget accepted (~165 GPU-h
training + rewriting, measured); (2) rewriter yield handled by the
built `align` repair step, with retained-and-aligned counts both
reported; (3) the probability-mass secondary metric for H1 is in.
Placement per the verified balanced plan: setup + pilots + the C1/C3
REWRITING passes on Anvil (vLLM recipe proven there; the current h006
allocation is drained for this as 1-NFE finishes); the EIGHT TRAINING
RUNS on one Delta gpuH200x8 node in a single ≤48 h window (torch cu128
pin; mixture staged via Globus; Delta submission requires the owner's
Duo login when reached). Hypothesis-bearing training does not start
until rewriting completes and the pilot gates pass.

**Deviation question 1 (2026-08-10, PENDING OWNER DECISION — campaign
continues, nothing prejudged):** with the align step ON, the corpus all
four arms can share is capped by the strict placebo arm at ~315–521 M
tokens (measured on 623 held-out documents; the exact figure depends on
the proper-name recall floor and a detector bug fix), i.e. 4–6× short
of the pre-registered 2 B per arm; closing the gap by rewriting more
source would cost 340–510 GPU-h and exceeds the staged corpora anyway.
Options: (A) equal-budget training at what the aligned corpus supports
(~475 M/arm at a 0.60 floor with the detector fixed; training cost
drops to ~41 GPU-h; smaller effects; "2 B each" changes) — worker and
coordinator RECOMMEND; (B) keep 2 B by ~4 passes over the aligned
corpus (repeated data, normal for mid-training) — fallback if effects
look too small; (C) drop align (reintroduces the composition confound
— not recommended); (D) buy the tokens (breaks budget and mixture —
not recommended). Already coordinator-ratified as a BUG FIX, not a
loosening: the proper-name detector counting "Just"/"There"/"About" as
names is corrected (standard same-passage lowercase test). To be
stated in the paper regardless: the aligned corpus is biased toward
short documents (all arms share it, so arm-vs-arm validity holds).
The campaign separates generation from filtering, so every option
above is producible from the same ~108 GPU-h of generation with no
regeneration. Prompt-improvement rounds: both spent per §4 (v2 lost
and was discarded; v3 won: C1 chunk pass 0.637→0.708, placebo
unchanged).

Status: **DRAFT v1, 2026-08-04 — for professor sign-off.** Start building the
pipeline after approval. Lock the design after the fact-accuracy check in §8.
Target venue: **ICML 2027**, around Jan 28; confirm the date before lock.

Paper type: **METHOD**. Under the owner's definition, this changes the data and
workflow. It challenges the usual assumption that every sentence in pretraining
text is a true statement. Check record: [[Method-Gates-2026-08]]. Related CVPR
diagnostic: [[Prereg-1NFE-Diversity]].

---

## 1. The problem

Language models train as if every sentence in their data is true. A rumor, a
sales pitch, a disputed claim, and a physics fact all enter the model in the
same form. This may help explain why models can be poorly calibrated on
disputed topics, flatter people by agreeing with the loudest source, and
"adopt" goals that appeared in text they merely read.

Bengio's Scientist AI program—the [Feb 2025 position paper
(2502.15657)](https://arxiv.org/abs/2502.15657) and the [Jun 2026 technical paper
(2606.29657)](https://arxiv.org/abs/2606.29657)—proposes a fix. Rewrite the data so each record shows the
difference between *"X is true"* and *"source S claimed X at time T in venue
V."* The technical paper defines this in **Definition 3.22** and says:
**"contextualization is specified solely at the level of requirements (i.e., we
do not provide a completed algorithm)."** The citation was corrected on
2026-08-02. The quote and Definition 3.22 are in 2606.29657, not the position
paper; we checked both PDFs.

The direction was named eighteen months ago, and its requirements have been
formal for one month. Nobody has built it, including [LawZero](https://lawzero.org). LawZero has no
public code and is still hiring a pipeline team. We will build the first
version and measure what it actually changes.

## 2. What recent work has shown (checked 2026-08-03)

- **No working implementation exists.** We checked again, including the most
  recent weeks.
- **The closest idea adds tags, which is different.** Source-aware training
  ([2404.01019](https://arxiv.org/abs/2404.01019)) connects *document ID tags* with content so a model can name its
  sources. It uses synthetic data and does not rewrite the text. It does not
  measure calibration or sycophancy. [CTRL-style codes](https://arxiv.org/abs/1909.05858) also add labels to
  the input. [GenProve](https://arxiv.org/abs/2601.04932) tracks the source of generated text. None changes the
  text itself into records that show the limits and source of a claim.
- Rewriting large datasets is already practical. Examples include denser
  rewrites and ["Synthetic Rewriting as a Quality Multiplier"](https://arxiv.org/abs/2603.24826), which uses
  placebo controls. This shows that the *pipeline* can work. A placebo arm is a
  basic control. Our new contribution is the schema for claims and the measured
  effects of using it.
- **Ready evaluation tools, checked directly:** ConflictBank ([2408.12076](https://arxiv.org/abs/2408.12076)) for
  conflicting knowledge; "How LLMs Balance Internal Knowledge with User and
  Document Assertions" ([2604.22193](https://arxiv.org/abs/2604.22193)), which almost exactly matches our
  question; standard sycophancy test sets; and standard ability tests to
  measure any cost.

## 3. The method

**Data format, inspired by Definition 3.22 and made concrete by us:** keep
direct factual observations as statements. Rewrite communicated claims to name
their source, venue, date, stance, and limits. Write the records in normal
language, with no required special tokens, so the method works with any model
design.

**Rewriter:** use an open 4–8B model with a tightly limited prompt and automatic
rule checks. Before rewriting the full dataset, check fact accuracy on samples
from different groups. Use both an LLM judge and a human spot-check. Fix a pass
threshold before the check.

**Dataset:** about 2B tokens from a [DCLM](https://arxiv.org/abs/2406.11794) or [FineWeb](https://arxiv.org/abs/2406.17557) subset. Add a
larger share of news, forums, and reviews because they contain many disputed
claims and give the method something to change.

**Training plan, revised 2026-08-04 after the owner's concern that training
from scratch costs too much:** continue pretraining, also called mid-training,
on the fully open [OLMo-2-1B](https://huggingface.co/allenai/OLMo-2-0425-1B) base model. Its original training data is public, so we
know what the model saw before our change. This is also how the method would be
used in practice. Labs do not restart pretraining; they would add
contextualized data during mid-training or learning-rate annealing. Keep one
from-scratch arm, 0.5B × 8B tokens, as a **later test allowed only by the rule
in §4**. Run it only if Phase 1 helps.

## 4. Exact experiment plan

**Four data arms, each with 2 seeds, giving 8 continued-pretraining runs on
OLMo-2-1B with 2B tokens each:**

- C0: raw dataset.
- C1: **contextualized**, with the full claim-aware rewrite.
- C2: **tags only**. Add the same metadata as tags but leave the text unchanged.
  *This is the deciding control.* It tells us whether rewriting helps beyond
  metadata. It also answers the likely objection that we rediscovered
  source-aware training.
- C3: placebo rewrite. Paraphrase the text while matching token budget and
  perplexity. This controls for the possibility that any rewrite helps.

**Predictions, fixed before the run:**

- **H1:** C1 beats C0, C2, and C3 on calibration during source conflict. On
  ConflictBank, measure accuracy and expected calibration error (ECE) on items
  with conflicting sources.
- **H2:** C1 reduces sycophancy more than every control.
- **H3:** C1 improves the assertion-balancing measurements from
  [2604.22193](https://arxiv.org/abs/2604.22193) more than C2. In other words, rewriting adds a benefit beyond
  tags.
- **H4:** C1 loses no more than 2.0 points on average across the standard
  ability test set compared with C0.
- **H5, rule for allowing the extra phase, replacing the older scale
  hypothesis:** if H1–H3 are positive, train the from-scratch 0.5B × 8B-token
  experiment with 4 arms × 2 seeds. It costs about +300 GPU-h including extra
  rewriting. Test whether C1 still beats C2 when the model has no earlier
  history of raw-text pretraining. Run this **ONLY** after a positive Phase 1.
  Its result can make the paper stronger, but does not decide whether the Phase
  1 paper succeeds.

**Decision rules:** use bootstrap confidence intervals over seeds and Holm
correction across H1–H5. Freeze every measurement and prompt when the design
locks. Before rewriting at scale, fix the fact-preservation pass threshold at
≥97% on the checked samples.

**An alternate result, not a reason to stop:** if C2 ≈ C1 on every main
measurement, report the honest result: "metadata conditioning is enough—the
cheap version of Scientist AI's data mechanism." This is still a method result
because tags cost almost nothing and rewriting does not. This branch is fixed
in advance.

**Reasons to stop:** (i) if the rewriter fails the fact-accuracy check after two
rounds of prompt improvement, stop. Report the pipeline difficulty as a short,
honest note, not a paper. (ii) If LawZero or another group releases an
implementation before our lock, re-check the research space within 48h. Our
tag-only control and effect measurements may still work as an *evaluation* of
their method, but the paper's framing must change.

**Claims we will not make:** any safety *guarantee*. We test starting claims and
possible benefits, not the Scientist AI theorem. We also will not claim results
at frontier scale or prevention of goal adoption beyond what the test suite can
stand in for.

## 5. What each possible result means

- **Main result:** contextualized models are better calibrated during source
  conflict and less sycophantic, with a small loss of general ability. This is
  the first evidence for a widely discussed but never built proposal. Release
  the pipeline, data, and checkpoints.
- **Tags are enough:** release and validate the cheap approach, and report the
  real cost of the more expensive rewrite.
- **No effect:** provide the first evidence that the Scientist AI data idea does
  not give its promised benefits at academic scale. This is publishable and is
  the field's only measurement either way.

## 6. Resources and schedule

**Revised Phase-1 cost: about 300–450 GPU-h.** Rewriting 2B tokens with a 4–8B
model costs about 150–250 H100-h. Eight continued-pretraining runs at 1B × 2B
tokens cost about 100 H100-h. Evaluations cost about 30. The from-scratch second
phase adds about 300 GPU-h only after a positive Phase 1.

**OrangeGrid and Anvil are enough for Phase 1.** Delta H200 may help with
rewriter batch speed but is no longer required. Work on the schema, rewriter,
automatic checks, and fact-accuracy audit can begin now without using the GPUs
needed by ICLR papers.

**Schedule:** Aug–Sep: schema, rewriter, and fact-accuracy check → Oct: lock and
rewrite at scale → Nov: pretraining arms → Dec: evaluations and analysis → Jan:
write and submit.

## 7. Risks and competing work to watch

- The main risk is that **LawZero, backed by NVIDIA, releases first**. The
  estimated window is 6–12 months. They cannot publish an honest negative about
  their own main idea, and they are likely to release a system rather than a
  study with placebo and tag controls. Watch lawzero.org, its hiring page, and
  citations of [2606.29657](https://arxiv.org/abs/2606.29657). Check again at every milestone.
- Fact accuracy after rewriting is the main technical risk. Test it before
  spending at scale.
- Nobody knows the effect sizes. The chosen scales and the disputed-claim-heavy
  part of the dataset make a measurable effect more likely. H5 checks whether a
  result is only an artifact of small models.

## 8. Checklist before locking

1. Professor approval of §4, especially C2, the fact-accuracy threshold, and
   which results lead to an alternate claim versus stopping.
2. Pass the fact-accuracy check on the pilot data.
3. Repeat the literature search with the most recent 8 weeks named directly.
4. Confirm the exact ICML 2027 deadline.
5. Mark the page LOCKED and record the git hash.

## Related

[[Method-Gates-2026-08]] · [[Unified-Direction-Ranking-2026-08]] ·
[[GPU-Resources-Across-Clusters]] · [[Data-Transfer-Between-Clusters]]

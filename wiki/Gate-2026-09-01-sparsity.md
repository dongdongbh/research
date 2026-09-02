# Gate — Sparsity-premise instrument (`sparsityprem`) → **SURVIVES, ★★★ conditional**

Pre-registration gate for rank 4 of [[Direction-Audit-2026-09-01]], run
2026-09-01 under the two-pass rule of the prereg workflow. This is the second,
independent pass. The first pass is the Stage-1 record in the project repo
(`runs/20260806-stage1/STAGE1.md`, dated 2026-08-06) and the audit item in
`research/.orchestrator/audit-20260901/intact-c.md`, item 3.

Design under gate: `/anvil/projects/x-cis261253/code/sparsityprem/DESIGN.md`,
status DRAFT, never authorized, never run.

**Rating moved down from ★★★½ to ★★★.** Two new facts caused this, both found
in this pass and both explained below: the training rule that the item needs in
order to pass the method filter has a published neighbour with 31 citations,
and the design cannot produce the threshold that rule needs without an extra
dose level. Neither fact kills the item. Both change what the owner is
approving.

---

## Words used on this page

- **Loss band** — a set of models that fit the training data about equally
  well. Moving inside a band changes the model in a way the loss barely
  notices.
- **R_shell** — the largest share of models inside any one band that are
  dangerous, measured before training starts. Definition 5.22 of the source
  paper.
- **No-enrichment** — the claim that training cannot multiply that share by
  more than a fixed constant. Requirement 5.23 of the source paper. The
  constant is called C_bad.
- **Mutual information** — a number that says how much knowing one thing tells
  you about another. Zero means the two are unrelated.
- **Paired bootstrap** — resample the same data many times to show how stable a
  number is, comparing arms on the same resamples each time.
- **GPU-h** — one hour of one graphics card.

---

## 1. The method filter, applied honestly

**As written, the design is not a method, and it says so itself.** Section 2 of
the design states the study "can produce exactly two kinds of result": a
refutation, or "no refutation found, at this scale, with this harm surrogate".
A refutation is a finding. The owner's definition admits a new mechanism, an
improvement on a current method, or an old method on a new problem. A finding
is none of the three.

**It passes only through the released-instrument clause.** That clause admits an
audit-shaped item when two things are true at once. First, the deliverable must
be a working instrument that people will run. Second, the audit must name, now,
the method paper it unlocks.

**The first half is satisfied.** The instrument exists and runs. It is a sampler
that wanders inside a fixed loss band, a coordinated-failure estimator, and a
curvature probe that measures whether the loss can even see the harmful
direction. I ran it today; see §3.3.

**The second half fails in the design as written.** Design §9 says the study
unlocks nothing: "a result here buys that paper one paragraph, not a whole
experimental arm ... judge this as a standalone safety-empirics paper."

### 1.1 The missing sentence, drafted

Arm C of the design is the one part that can become a rule. Arm C adds a
feature that the optimizer can see and that happens to line up with the
dangerous query set, then asks how much lining-up is enough to break the safety
bound. Turn that into a shipped training rule. Add this to the design as a new
§9.1, headed "The method paper this unlocks":

> **Safety-critical feature admission.** Before training, measure the mutual
> information between every training-visible feature and the safety-critical
> query set. Drop any feature whose mutual information is above the measured
> threshold, or decorrelate it from that query set with an independence penalty
> while keeping it. Release the measurement and the filter as one command-line
> tool. The paper it produces is "a measured admission rule for training
> features, derived from the point where a published safety bound breaks."

That is a changed data-and-training workflow, which is route 1 of the owner's
definition. It is a named method paper, so the released-instrument clause is
satisfied.

### 1.2 Which experiment produces the threshold

**M2, the Arm-C dose-response curve.** Design §6, M2 already says it: "Arm C is
reported as a dose-response curve: how much accidental correlation (measured as
the mutual information between the training-visible feature and the
critical-query set) is needed before Ĉ_bad crosses 10." The mutual-information
value at which the measured C_bad crosses 10 is the threshold the rule ships.
The crossing point is measured once per width, so the rule also learns whether
the threshold moves as the model gets bigger.

### 1.3 A design gap this pass found: one dose is not a curve

**State this loudly, because it changes the budget.** The sweep in §4 is 4
widths × 3 arms × 25 seeds = 300 models. That gives Arm C exactly **one**
correlation strength. You cannot fit a dose-response curve through one dose, so
the design as written cannot produce the threshold that §1.1's rule needs.

The fix is small. Run Arm C at **four** correlation strengths instead of one.
That adds 300 trained models and their curvature probes. Using the design's own
measured numbers (300 models at 20,000 steps ≈ 9.7 GPU-h; 300 curvature probes
≈ 1 GPU-h), the extra cost is about **11 GPU-h**. The band-sampling chains cost
the same either way, because they are run per width and not per model.

So the honest total becomes **about 81 GPU-h**, not 70. That still fits inside
the design's existing 35 GPU-h contingency line, but it eats a third of it.
Budget ~80 GPU-h and say so.

### 1.4 The gotcha in the rule, which must be written into the prereg

**The threshold number will not transfer, and claiming it does would be false.**
The world here is 32 features and 4-token scenarios. A mutual-information
crossing point measured in that world is not the crossing point for a real
model. What ships is the **procedure**: measure the mutual information, run a
small local dose sweep, find your own crossing point, then filter. The prereg
must say that in the "what we will NOT claim" section. Any draft that ships a
single universal threshold number should be rejected at review.

---

## 2. Second scoop pass

Run today with the paper-search tool across arXiv, DBLP, OpenAlex, Semantic
Scholar and Crossref. The arXiv module throttles itself to one request every
four seconds through a file lock, so the requests were paced. The
most-recent-10-weeks pass covers 2026-06-23 to 2026-09-01 and was run
separately with a date filter, as required.

**The premise test and the training rule were scooped separately**, because the
rule can be taken even if the instrument is not.

### 2.1 The premise test — still empty, Level 4

Queries run: sparsity of dangerous predictors loss band · no-enrichment
requirement training safety bound · empirical test safety theorem assumption
language model · Langevin dynamics unsafe set training · harm coordination
dimension loss landscape · loss level set sampling Langevin unsafe region volume
· density of states neural network loss landscape safety. The last two were the
adjacent-vocabulary pass, using the statistical-physics community's words
instead of ours.

The only relevant hits were papers our own design already names:

- [Avoiding unsafe sets when training with Langevin Dynamics
  (2607.07538)](https://arxiv.org/abs/2607.07538) — the theory follow-up, still
  zero experiments.
- [Revisiting the Volume Hypothesis
  (2606.31282)](https://arxiv.org/abs/2606.31282) — already budgeted as a
  cross-check sampler, not a competitor.
- [Scherlis & Belrose (2501.18812)](https://arxiv.org/abs/2501.18812) — already
  budgeted as a third cross-check.

The 10-week pass returned 76 papers and none of them touched the premise. **No
one has measured R_shell or the no-enrichment constant.** Level 4 stands, and
it is now verified twice by two independent passes.

### 2.2 The training rule — Level 3, and this is the new bad news

Queries run: feature decorrelation safety training language model · spurious
feature removal mutual information filter training data · decorrelate training
features from safety evaluation set · pretraining data filtering safety critical
queries mutual information · shortcut feature removal alignment training rule ·
safety data filtering training feature correlation · remove features correlated
with harmful query set. Adjacent-vocabulary pass, using the causal-inference and
fairness communities' words: HSIC independence penalty remove spurious
correlation training · invariant risk minimization safety guarantee language
model.

Two real threats, both deep-read from the full PDF.

**Threat 1, the close one.** [Layer-Aware Representation Filtering: Purifying
Finetuning Data to Preserve LLM Safety Alignment
(2507.18631)](https://arxiv.org/abs/2507.18631), EMNLP 2025, **31 citations**,
code live at [LLLeoLi/LARF](https://github.com/LLLeoLi/LARF) (checked today,
HTTP 200). It does the same *shape* of thing our rule does: it removes training
data that would damage safety, before training. Its own words on the mechanism:

> "We pinpoint these layers by selectively parameter scaling and evaluating
> safety behavior shifts. Subsequently, we rank the samples based on their
> bidirectional representations in the safety-sensitive layers — upranking truly
> safe samples while downranking safety-degrading samples that weaken the
> model's rejection capability."

**The gap sentence, quoted from its own Limitations section:**

> "Our filtering strategy relies on representational similarity between samples
> and a chosen reference set, so its effectiveness is inherently tied to the
> quality and composition of that reference data. While we acknowledge that
> carefully curated reference datasets could further improve results, exploring
> optimal reference selection lies beyond the scope of this work and represents
> a promising direction for future research."

That is the opening, stated by the competitor: they rank by distance to a
hand-chosen reference set and admit they have no principled way to choose the
cut. Our rule measures mutual information against a declared safety-critical
query set and takes its cut from the point where a published safety bound
breaks. **Three differences hold:** we filter features, not samples; our cut is
measured, not chosen; our object is pretraining-time feature admission, not
post-hoc fine-tuning hygiene on an already-aligned model.

But the slot is occupied, and LARF's own related-work names three more
occupants: Bi-Gradient (He et al., 2024), SEAL (Shen et al., 2025) and DABUF
(Pan et al., 2025). I could not verify arXiv identifiers for those three, so
they are named here as LARF cites them and not as links.

**Threat 2, the mechanism one.** [Rectifying Shortcut Behaviors in
Preference-based Reward Learning
(2510.19050)](https://arxiv.org/abs/2510.19050), NeurIPS 2025. Its abstract:
models "achieve high reward scores by exploiting shortcuts, that is, exploiting
spurious features ... that correlate with human preference labels in the
training data". Its fix, named PRISM, "learns group-invariant kernels with
feature maps in a closed-form learning objective". That is feature
decorrelation for alignment, done in reward learning rather than pretraining,
with no mutual-information measurement and no threshold. Mechanism overlap is
partial; framing and object differ.

**The 10-week pass on the rule returned nothing new.** No 2026-06-23-onward
paper measures mutual information against a safety-critical query set.

**Verdict on the rule: Level 3 (medium overlap), not Level 4.** Two of the four
axes match LARF: the problem framing (protect safety by removing training
material) and the application domain (safety alignment of trained models). The
core mechanism and the key insight differ.

### 2.3 What this means for the gate

The instrument is Level 4. The unlock is Level 3. Because the item passes the
method filter **only** through the unlock, the item's effective rating is set by
the weaker of the two. That is the half-star.

The unlock paper is still worth writing. It must be positioned against LARF
from its first paragraph, and it must carry LARF as a baseline, or a reviewer
will ask why measured mutual information beats measured representation
distance and the paper will have no answer.

---

## 3. Live verification

Everything below was checked today, 2026-09-01, not read from our own records.

### 3.1 The two source papers

| Paper | Version now | Posted | Last updated | Citers (Semantic Scholar) | Citers (OpenAlex) |
|---|---|---|---|---|---|
| [Safety from Honesty in a Disinterested AI Predictor (2606.29657)](https://arxiv.org/abs/2606.29657) | **v2** | 2026-06-28 | 2026-07-10 | 1 | 0 |
| [Avoiding unsafe sets when training with Langevin Dynamics (2607.07538)](https://arxiv.org/abs/2607.07538) | **v2** | 2026-07-08 | 2026-07-15 | 0 | 0 |

The single citer of the source is the Oberman follow-up itself. OpenAlex has
still not linked it, which is why the two columns disagree. **Nothing new cites
either paper.** The slot has not moved in the four weeks since our Stage-1 pass.

I also searched arXiv for every paper mentioning "Scientist AI" or "LawZero",
newest first. The most recent relevant item is the source paper itself.
**LawZero has posted no empirical follow-up.**

### 3.2 The version record, corrected

The audit asked us to record which version of the source the decision rules
were read from. That record **already exists**, in the Stage-1 file, which opens
with "Source: **arXiv 2606.29657 v2**". So the correction is narrower than the
audit stated: the version is recorded in `runs/20260806-stage1/STAGE1.md` but
not in `DESIGN.md`. Copy the string "2606.29657v2" into DESIGN.md §1 before
locking.

One small mismatch to fix while doing it: the Stage-1 file says the source is
"Dated July 14, 2026", while arXiv's own v2 timestamp is 2026-07-10. Use the
arXiv timestamp.

New fact: the Oberman follow-up is now at **v2 (2026-07-15)**, which is before
our 2026-08-06 read, so our quotes are from v2. Record that string too.

### 3.3 The harness runs

I ran the smoke script on the Anvil login node today. This was not a
performance test; the login node was carrying a load average of 376 across 32
cores, and the job ran on CPU only. The point was to prove the code still runs
end to end and reproduces the design's stated behaviour.

```
cd /anvil/projects/x-cis261253/code/sparsityprem
OMP_NUM_THREADS=2 MKL_NUM_THREADS=2 uv run python scripts/run_smoke.py \
  --out <scratch>/smoke-tiny --n-models 1 --steps 60 --sgld-steps 100
```

Result, in full:

```
wall 104.3s | train/model 65.58s | sgld/1k 13.72s | curv 24.88s
params/model: 104867
final J: [0.2749]
dangerous at end: [False]
sgld in-band: 50 / 50
curvature: tangent ratio 11704.4268 (harm 3.2117e+00 vs random 2.7440e-04); cos(harm, gradJ) = -0.9579
```

Three things this establishes.

- **`uv` resolves and the package imports.** Environment recorded by the script
  itself: host `login07.anvil.rcac.purdue.edu`, torch 2.13.0+cu130, Python
  3.14.6, no GPU.
- **The band sampler still works.** 50 of 50 recorded samples stayed inside the
  band, matching the design's smoke claim exactly.
- **The curvature probe computes and has range.** The ratio here is 1.2 × 10⁴
  against the design's 7 × 10⁴, which is expected and means nothing: this model
  trained for 60 steps and the design's trained for 3,000. Do not quote either
  number as a result.

One harmless warning appears from PyTorch about nested tensors in the
transformer encoder. It does not affect the computation.

---

## 4. The week-1 decisive step

Taken from design §6 and §11, unchanged, with the numbers stated as numbers.

**Run the pilot block only.** Seeds 900 to 924, which are disjoint from the
analysis seeds 100 to 124. The pilot fixes and then freezes five quantities: the
band half-width Δ, the harm-reward strength α_h, the dangerous-score threshold
υ_bad, the percentile cut q, and the six checkpoint steps. Record the freeze in
the run directory before any analysis seed is touched.

**Cost: about 10 GPU-h.** Time to a decision: days.

**Placement: OrangeGrid, as short independent jobs under HTCondor.** OrangeGrid
charges no ACCESS credits and has no wall-time limit
([[GPU-Resources-Across-Clusters]]). The design's measured reason for this
choice is decisive and worth repeating: three copies of the same job took 112
seconds run one after another, and 112 seconds run all at once on one H100.
Packing work onto a big card buys exactly nothing here, so throughput scales
with the number of machines, not their size. OrangeGrid's two-GPU-per-node limit
does not bite, because no job needs more than one GPU.

**Kill number 1, the positive control.** In Arm B we deliberately push the model
toward coordinated guardrail failures. Arm B's measured C_bad must exceed Arm
A's, with a paired bootstrap interval on the difference that excludes zero. **If
Arm B does not separate from Arm A on the pilot seeds, stop, and do not run the
sweep.** The instrument cannot see an effect it was handed, so a clean Arm-A
result would mean nothing. This stop is already pre-stated in design §6, M2.

**Kill number 2, the sanity invariants.** The band chains must stay inside the
band for at least 90% of recorded samples. The marginal false-negative rate must
be strictly between 0 and 1 in every cell; a cell where it is 0 or 1 is void,
because the harm score is then degenerate. Today's run was at 100% in-band on
50 samples, so this invariant is not expected to bite.

**Add one step to the checklist before locking.** The four-dose Arm C from §1.3
must be written into design §4.2 and §7 before the pilot runs, because the
pilot must fix the four correlation strengths as well.

---

## 5. Baselines, venue, hazard, unlock

### 5.1 Baselines the paper must carry

For the measurement:

- **Two independent samplers, not one.** Stochastic Gradient Langevin Dynamics
  and a Replica-Exchange Wang-Landau estimate must agree on R_shell at the two
  smallest widths, within their intervals. A single sampler's bias would look
  exactly like a result. Wang-Landau follows [Revisiting the Volume Hypothesis
  (2606.31282)](https://arxiv.org/abs/2606.31282).
- **A third, cheaper cross-check** from [Scherlis & Belrose
  (2501.18812)](https://arxiv.org/abs/2501.18812) on the same two widths.
- **The isotropic prediction as the null.** Lemma B.5 of the source predicts a
  Beta distribution with mean m_H divided by d_B. Measured against predicted is
  the readout, not a count of dangerous models.
- **The independence null.** At initialization, measured coordination must match
  what independence predicts from the model's own marginal miss rate. If it does
  not, we are measuring the miss rate and not coordination.
- **The nearest existing evidence:** [What Shapes Emergent Misalignment?
  (2606.20814)](https://arxiv.org/abs/2606.20814) found that at matched or lower
  training loss, alignment scores barely varied across runs. That points to a
  small within-band spread and is the reason M1 reads a whole distribution
  rather than counting rare events.

For the unlock paper:

- **[LARF (2507.18631)](https://arxiv.org/abs/2507.18631) is now mandatory.**
  Same task, released code, 31 citations. Our rule must be compared against
  representation-distance filtering on the same data, or the delta is only
  asserted.
- **[PRISM (2510.19050)](https://arxiv.org/abs/2510.19050)** as the
  decorrelation-side comparison.
- A **no-filter control** and a **random-filter placebo** that drops the same
  number of features at random. Without the placebo, any gain could be the
  effect of training on less data.

### 5.2 Venue

**Post to arXiv first, within six weeks of the go decision.** The expiry risk in
§5.3 dominates every other consideration, and the study is short enough to
finish inside that window. Then submit the full paper to ICLR 2027 or ICML
2027; the measurement and the unlock rule can go as one paper or as two, and
that call should wait until Arm A's result is known.

Frame the title and the abstract around **academic scale**, with the scale said
out loud. Nothing here transfers to frontier models by itself, and the source
paper's own Remark 5.29 says probes cannot certify the premise anyway.

### 5.3 The visibility hazard

**This is the largest risk and it has not changed.** The idea is rooted in a
named group's stated agenda. LawZero has the theory, the motive and the funding,
and the Oberman follow-up names our exact hypothesis as untested: "the energy-gap
link J ≥ ψ(a_H) is a hypothesis, not a theorem: it asserts that unsafe
parameters are visible to the loss."

Today's check says the risk has **not yet fired**: zero new citers on either
paper, no LawZero empirical follow-up on arXiv. But four weeks passed between
our Stage-1 pass and this one, and nothing moved in our favour either. The
mitigation is the same one the design gives, and it is the reason to decide now:
this is an 80 GPU-h study with a working harness, so it happens in weeks or not
at all.

**A second hazard, new in this pass.** The unlock rule sits in a fast, crowded
corner. LARF collected 31 citations in about thirteen months and ships code from
a large industrial lab. Treat the unlock paper's clock as shorter than the
measurement paper's.

### 5.4 What it unlocks

One method paper, named in §1.1: a measured admission rule for training
features, derived from the point where a published safety bound breaks,
released as code.

It does **not** unlock a theory arm for
[[Prereg-Epistemic-Contextualization]]. Design §9 already corrects our ranking
row on this, and the correction stands. That prereg implements Definition 3.22
of the same source paper, the data-rewriting mechanism. This study tests
Definition 5.22 and Requirement 5.23, the training-dynamics premises. Neither
depends on the other. A result here buys the contextualization paper one framing
sentence, and only if this study finds a refutation.

---

## 6. Verdict

**SURVIVES, ★★★, conditional.**

The measurement is genuinely unclaimed and now verified twice: Level 4, one
citer worldwide, no empirical follow-up in the nine weeks since the source paper
was posted. The instrument exists
and ran today. The cost is measured, not guessed, and the placement is free. Of
everything in this audit wave, this is the item closest to being able to start
tomorrow.

It is held to three stars, down from the audit's three and a half, for two
reasons found in this pass. The training rule it must name in order to pass the
method filter has a published, code-released, 31-citation neighbour in LARF, so
that rule is Level 3 and not new territory. And the design as written cannot
produce the threshold that rule needs, because Arm C runs at one correlation
strength and a dose-response curve needs several.

### The condition, as one sentence to accept or reject

> **I approve the sparsity-premise sweep on the condition that, before the pilot
> runs, the design is edited to (a) name the feature-admission training rule of
> §1.1 as the method paper it unlocks, (b) run Arm C at four correlation
> strengths instead of one, raising the budget from ~70 to ~80 GPU-h, (c) carry
> [LARF (2507.18631)](https://arxiv.org/abs/2507.18631) as a mandatory baseline
> for that rule, and (d) record the source version strings 2606.29657v2 and
> 2607.07538v2 in DESIGN.md; and I accept that the study can only refute the
> premise, never confirm it.**

Reject that sentence and the item fails the method filter, because a refutation
instrument with no named unlock is a finding, not a method.

---

## 7. Edits this gate puts on the record

1. `sparsityprem/DESIGN.md` §9 — add §9.1, the named method unlock, as drafted
   in §1.1 above.
2. `sparsityprem/DESIGN.md` §4.2 and §7 — Arm C runs at four correlation
   strengths; budget rises to about 81 GPU-h.
3. `sparsityprem/DESIGN.md` §1 — record "2606.29657v2" and "2607.07538v2".
4. `sparsityprem/DESIGN.md` §11 — add the four Arm-C doses to the list of
   quantities the pilot must fix and freeze.
5. `sparsityprem/DESIGN.md` §2 — add to the "what we will not claim" list: the
   measured mutual-information threshold is a procedure, not a portable number.
6. [[Unified-Direction-Ranking-2026-08]] Part 2 — the sparsity row's cost
   becomes ~80 GPU-h, its scoop level becomes 4, and its star becomes ★★★.
7. `intact-c.md` item 3 said the design never records the source version. It is
   recorded in the Stage-1 file, just not in DESIGN.md. Narrow the correction.

## Related

[[Direction-Audit-2026-09-01]] · [[Unified-Direction-Ranking-2026-08]] ·
[[Method-Gates-2026-08]] · [[Prereg-Epistemic-Contextualization]] ·
[[GPU-Resources-Across-Clusters]]

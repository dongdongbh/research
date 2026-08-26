# Pre-registration: Epistemic Contextualization — the First Implementation

## Where this stands right now

**The study is STOPPED, and the amended gate re-run is COMPLETE — all
seven arms judged (2026-08-26), no arm passes.** A check we wrote into
the plan before running anything came back FAIL on 2026-08-10; the
owner amended the method (Deviation 3) and we re-ran the gate across
seven arms and three rewriter models. Final picture: the amendment's
grounding fix **works at both model scales** (it fixes attribution
precision on the 7B, and unlocks 6.3× more contextualisation at
unchanged precision on the 70B), scale alone **fails** (the 109B model
is worst on both axes), and the frozen fact-preservation gate **fails
every arm** — substantially because it scores rewrites against the
chunk alone, penalising the retrieved provenance the method exists to
add. The full record is in the "Amended gate re-run" sections below.
One owner decision is pending: report as pre-registered, or ratify
Deviation 5 and re-score against chunk plus retrieved metadata.

## Words used on this page

Read this list once. Every word below is used the same way everywhere on this
page.

- **Token** — a piece of text, roughly a short word or part of a word. Model
  training is measured in tokens. "2 B" means two billion tokens.
- **Corpus** — the whole body of text we train on.
- **Shard** — one small chunk of the corpus, processed as a unit.
- **Arm** — one version of the data we train a model on. This study has four
  arms, named C0 to C3.
- **Rewriter** — the model that rewrites raw text into our new format.
- **Placebo arm** — an arm that gets rewritten text with no real change in
  meaning. It shows whether any rewrite at all would have helped.
- **Align step** — a repair step that makes the four arms cover the same
  documents, so an arm-versus-arm comparison stays fair.
- **Mid-training** — continuing to train a model that someone else already
  pretrained, instead of starting over.
- **Blinded** — the person labelling does not know which arm an item came from.
- **Wilson lower bound** — the low end of a confidence interval for a
  percentage. It is the cautious reading of a pass rate.
- **Deviation** — any change made after the plan was locked. Every deviation is
  written down on this page.

## Execution start (2026-08-10, owner-initiated)

The owner started this phase. Verbatim: "we may run the basic part on anvil,
then move to delta for full training".

Three readiness choices proceed on their recommended defaults, unless the owner
or the professor overrides them.

1. The revised budget is accepted: about 165 GPU-h for training plus rewriting.
   That figure is measured, not guessed.
2. Rewriter yield is handled by the `align` repair step we built. Both counts
   are reported: how many documents were retained, and how many were aligned.
3. The probability-mass secondary metric for H1 is in.

**Where the work runs**, following the verified balanced plan:

| Work | Cluster | Details |
|---|---|---|
| Setup, pilots, and the C1/C3 rewriting passes | Anvil | The vLLM recipe is proven there. The current h006 allocation is drained for this as 1-NFE finishes. |
| The eight training runs | Delta | One gpuH200x8 node, in a single window of ≤48 h. Pin torch cu128. Stage the mixture through Globus. Delta submission needs the owner's Duo login when we reach it. |

Training that carries a hypothesis does not start until rewriting is finished
and the pilot gates pass.

## Deviation question 1 (2026-08-10, PENDING OWNER DECISION)

The campaign continues while this is open. Nothing is prejudged.

**The problem.** With the align step ON, the corpus that all four arms can
share is capped by the strict placebo arm at about 315–521 M tokens. We
measured that on 623 held-out documents. The exact figure depends on the
proper-name recall floor and on a detector bug fix. That cap is 4–6× short of
the pre-registered 2 B tokens per arm. Closing the gap by rewriting more source
text would cost 340–510 GPU-h, and it would exceed the staged corpora anyway.

**The four options.**

| Option | What it does | Recommendation |
|---|---|---|
| A | Train at equal budget on whatever the aligned corpus supports. Now measured on real shard-0 rewrites and scaled: about 351 M per arm at floor 0.80 with the corrected detector, or about 529 M per arm at floor 0.60 with the corrected detector. That is about a quarter of the pre-registered 2 B, and the training cost drops with it. The "2 B each" figure changes. | Worker and coordinator RECOMMEND this. Floor 0.60 is the recommended setting. |
| B | Keep 2 B by making about 4 passes over the aligned corpus. Repeating data is normal for mid-training. | Fallback, if the effects look too small. |
| C | Drop the align step. | Not recommended: it reintroduces the composition confound. |
| D | Buy the tokens. | Not recommended: it breaks both the budget and the mixture. |

**One fix is already ratified.** The coordinator ratified the detector
correction as a BUG FIX, not as a loosening of the rule. The proper-name
detector was counting "Just", "There", and "About" as names. The fix is the
standard same-passage lowercase test.

**One limitation goes in the paper no matter which option wins:** the aligned
corpus is biased toward short documents. All four arms share that same corpus,
so arm-versus-arm validity still holds.

**No work is wasted.** The campaign separates generation from filtering. So
every option above can be produced from the same ~108 GPU-h of generation, with
no regeneration.

**Prompt-improvement rounds: both are spent**, as §4 allows. Version v2 lost and
was discarded. Version v3 won: the C1 chunk pass rate went 0.637→0.708, and the
placebo was unchanged.

## Sequencing note (2026-08-10, worker self-reported and self-corrected)

**What went wrong:** the rewriting campaign started before the pre-registered
fact-accuracy gate had run.

**What the worker did:** halted at the next shard boundary, ran the audit
first, and resumed only after a pass.

**How much was exposed:** one atomic shard, about 8 GPU-minutes. Nothing
entered any arm before the gate.

## Audit outcome (2026-08-10): the pre-registered gate FAILED

**The stopping rule in §4 is triggered. The study is STOPPED, pending the
owner's and the professor's decision.**

The threshold was frozen at 0.97 before any judging began.

| Arm | Fact-preservation rate | Wilson lower bound | Verdict |
|---|---|---|---|
| C1, the treatment | 0.9119 | 0.8809 | FAIL |
| C3, the placebo | 0.9714 | 0.9507 | Misses on the lower bound only |

Both pre-registered prompt rounds were already spent, so we cannot simply try
again with a better prompt.

**The finding inside the failure.** The attribution mechanism itself INVENTS
sources when a document's metadata is thin. The judge's own wording: "adds a
source that was not present in the original". This happens at 3× the placebo's
rate, so it is specific to the treatment. It is the method's own fact-integrity
cost, and it lands exactly on what the treatment does. Worst group: academic
text, at 0.833.

**Caveat on the judge.** The LLM judge comes from the same model family as the
rewriter. That brings a leniency bias, so the true C1 rate is more likely lower
than 0.9119, not higher.

**The human spot-check the plan requires now awaits the owner.** It is 50
blinded items, at ctxprereg
runs/2026-08-09/fact-audit-01/human_spotcheck.csv. The key sits in the sibling
json file.

**Two decision branches for the owner.**

- (i) Stop, and publish the short honest note that §5 anticipated. This is a
  sharper negative result than "no effect": contextualization's data mechanism
  has a measured fact-integrity cost that the placebo does not pay.
- (ii) Amend the method — a stronger rewriter, or a retrieval-grounded source
  field instead of a generated one — log it as a deviation, and re-run the
  gate.

The corpus-size question above is downstream of this choice. It is moot unless
the owner picks (ii).

**Exposure before the stop:** one 9-minute shard. Nothing in any arm.

## Work approved to continue during the stop

One check is arm-blind and is needed under every branch, so it was approved:
the raw-text learning-rate check, about 3 GPU-h.

**The LR check is DONE and LOCKED at 1e-5.** The honest reading is recorded:
1e-5 versus 2e-5 is not a real separation, and only 5e-5 is clearly worse. The
value of the lock is not that 1e-5 is proven best. It is that the choice was
fixed in advance, on validation data.

## Fixes and hand-over state

**The de-blinding key was repaired.** The human spot-check's key could not
attribute 17 of the 50 items to an arm. We regenerated the key from the seed,
and verified that the blind CSV is byte-identical. The provenance is in
KEY_REGENERATED.json. The human still labels blind.

**DELTA_LAUNCH.md is complete, as far as it can be.** It opens with a blocking
notice: do not follow it until the owner rules on the gate.

**Both Anvil allocations were released on 2026-08-10.** The queue is empty
while the owner decides.

**Restart-cost note, at the worker's correct insistence.** Releasing the
allocations also surrendered the queue position of the pending follow-on job.
So "continue rewriting" now means: request a card → wait in the queue, which
has historically taken multi-hour to multi-day on this account → resume from
the finished shard. Nothing is lost except the wait. The campaign resumes
cleanly whenever a card appears.

**Integrity after the cancellation:** all four run directories are verified
complete, with manifests. Nothing of record lived on scratch.

**OWNER DECISION (2026-08-15): AMEND THE METHOD AND RE-RUN THE GATE —
Deviation 3, ledgered here.** The amendment tests two fixes for the
invented-sources failure, separately and together: (a) a STRONGER
REWRITER (candidates served through AnvilGPT, Purdue's hosted
open-model API at anvilgpt.rcac.purdue.edu — the owner's suggestion;
access requires an owner-side ticket if not yet enabled), and (b) a
RETRIEVAL-GROUNDED SOURCE FIELD: where the corpus carries real
metadata (news outlet, date, forum username), the rewrite USES it
instead of asking the model to produce one — attacking the failure at
its root, since the gate showed the model invents sources exactly when
metadata is thin. The re-run keeps the frozen 0.97 threshold and the
same audit design; one improvement is REQUIRED this time: the judge
must be a DIFFERENT model family from the rewriter (round 1's judge
shared the rewriter's family — a recorded leniency bias). The round-1
human spot-check (50 blinded items) is still owed and applies to the
new round as well. Campaign economics note: an API can serve the GATE
(≈2,100 documents) cheaply, but the full 2 B-token rewriting campaign
needs self-hosted throughput — if a stronger model passes the gate,
the campaign serves that model with vLLM on Anvil GPUs as before.

**Amendment progress note (2026-08-15) — two findings that REFRAME
round 1; gate re-run in flight, verdicts pending:**
1. **Our staging fed the rewriter FALSE source facts.** 88.6% of the
   2,100 audit documents carried at least one wrong or junk metadata
   field: the crawl date presented as the publication date (64.9%),
   corpus slugs as outlets ("reddit-forum"), the literal word "Edu" as
   the outlet on 264 academic pages, raw Unix timestamps as dates. The
   worst round-1 group (academic, 0.833) is the group with 99.7% false
   dates. Real metadata existed upstream and was dropped at staging;
   recovery is built and joined 100% on all three joinable corpora.
   So part of "the model invents sources" was "we handed the model
   invented sources." An early n=8 signal even suggests stronger
   rewriters do WORSE under bad metadata — they copy it more
   faithfully.
2. **Round 1's judge leniency was worth 26–35 points, not a
   footnote.** An independent-family judge re-scoring round 1's own
   saved rewrites: C1 0.9119 → 0.65, C3 0.9714 → 0.62 — and the
   treatment-vs-placebo separation DISAPPEARS (the placebo fails as
   often, via omissions/changed values instead of invented sources).
   Under the independent judge, round 1 reads as a general
   rewriting-fidelity problem at 7B scale, not a
   contextualization-specific cost. The round-1 headline must not be
   quoted without this caution until the human spot-check arbitrates.
**Consequence: the 50-item human spot-check is now load-bearing** — two
judges disagree by 26–35 points on identical text, so only human labels
decide which is closer to truth (the owner is labeling via the blinded
web sheet). Gate re-run: five arms (incl. a serving-control arm), same
frozen 0.97, independent judge throughout, invented-source count
reported as a diagnostic beside the criterion. AnvilGPT: working, 16
models, account-wide ~80 req/min cap that returns HTTP 400 (not 429).
**HUMAN SPOT-CHECK COMPLETE (owner, 2026-08-15; scored record in
ctxprereg `runs/2026-08-09/fact-audit-01/human_spotcheck_SCORED.json`):
the human pass rate is 35/50 = 70%**, far below the round-1 judge's 94%
on the same items and close to the independent judge's overall rate —
**round 1's leniency is confirmed by ground truth** (13 items the judge
passed and the human failed, versus 1 the other way). The human labels
also show no meaningful treatment-vs-placebo separation (C1 0.67 vs C3
0.73 at n=24/26), supporting the "general rewriting-fidelity problem at
7B scale" reading. Blind-integrity note: one item (20) received a
coordinator explanation before the owner's verdict. Implication for the
gate: with humans finding ~30% of rewrites fact-lossy, the 0.97 bar is
far from ANY current arm — the negative finding is human-anchored, not
a judge artifact.

**JUDGE CALIBRATION AGAINST THE HUMAN LABELS (2026-08-15; record in
ctxprereg `runs/2026-08-14/judge-calibration-01/`):** five judges
scored on the owner's 50 labels (15 true fact losses). The round-1
judge misses 87% of real fact losses (sensitivity 0.13; on the placebo
it caught 0 of 7 — why C3 "looked" like 0.97). **gpt-oss:120b is
essentially unbiased as a rate (+0.020 overall; 0.000 on C1)** and is
the calibration judge of record. Per-item join to the human sheet was
reconstructed from the exporter's seeded shuffle and verified
byte-identical on all 50 items.
**DECISION REQUIRED (owner): the frozen 0.97 threshold is unreachable
through ANY judge that can actually detect fact loss** — every such
judge fails some genuinely faithful rewrites (specificity 0.82–0.89),
capping even a PERFECT rewriter below 0.97; the only judges that reach
0.97 are the ones that pass real damage. Three readings, reported side
by side by the gate summary with the literal verdict always first:
(1) LITERAL — no arm can ever pass; the study stops partly for
instrumentation reasons; (2) CALIBRATED — read 0.97 as ≥0.813 on the
gpt-oss scale (a true 97% maps there through its measured specificity
0.824, CI 0.665–0.917); needs owner ratification as a deviation;
(3) through the round-1 judge — reachable but indefensible (it would
certify ~30% document damage). **OWNER DECISION (2026-08-15,
verbatim: "Calibrated") — Deviation 4, ratified:** the gate is read
through the CALIBRATED mapping — an arm passes if its gpt-oss-judged
rate is ≥0.813 (where a true 97% lands through the judge's measured
specificity 0.824, calibrated on the owner's 50 human labels). The
literal 0.97 verdict is still computed and reported FIRST beside every
calibrated verdict, so any reader can apply either rule; the
calibration record (judge_calibration.json) and its 50-item basis are
cited wherever the mapping is used, including its CI (0.665–0.917) and
the sensitivity trade-off that motivated it. **Round-1 headline correction, now
statistically grounded:** human C1−C3 difference −0.064 (CI −0.304 to
+0.181, p=0.76) — no contextualization-specific cost; the phenomenon is
general 7B rewriting infidelity. Transferable design lesson (mirrors
RoboJudge's blind-floor lesson): a pre-registered pass threshold must
be set BELOW the judge's own measured specificity ceiling, or it tests
the judge, not the method.

## Amended gate re-run — decisive bf16 results (2026-08-15)

This section records the finished matched pair: the grounded prompt
(called **v4**) against the round-1 prompt (called **v3**), same model
(Qwen 7B), same weights, same random seed, same 5,860 chunks, same
serving (bf16 through vLLM), same judge (gpt-oss:120b). Only the prompt
differs, so any difference between the two is caused by the prompt.
Records live in ctxprereg `runs/2026-08-15/` (run directories named in
each paragraph).

**1. Gate verdicts — every arm FAILS, under both readings.**

| Arm | Pass rate | Literal (≥0.97) | Calibrated (≥0.813, Deviation 4) |
|---|---|---|---|
| v4 grounded | 0.5381 | FAIL | FAIL |
| v3 control | 0.6452 | FAIL | FAIL |
| C3 placebo | 0.6190 | FAIL | FAIL |

All three sit below the robustness band as well, so the verdict does
not depend on judge-uncertainty choices. The v4-vs-v3 gap is −10.7
points (p = 0.0016 — a real difference, not chance).

**2. The invented-sources problem — the thing Deviation 3 was built to
fix — is fixed.** We first used a model-based probe to count invented
sources, and we RETRACT its number (32.4%): reading its flagged items
showed 134 of 136 flags were the *real* outlet and date that the
grounded prompt had correctly supplied — a 98.5% false-alarm rate,
because the probe compared the rewrite against the chunk alone. The
replacement is a deterministic check (no model involved): a detail
counts as invented only if it appears in neither the original chunk nor
the retrieved metadata for that document. Result, n = 420 per arm:

| Arm | Invented sources | 95% range |
|---|---|---|
| v3 control | 11/420 = 2.62% | 1.47–4.63% |
| **v4 grounded** | **2/420 = 0.48%** | 0.13–1.72% |
| C3 placebo | 6/420 = 1.43% | 0.66–3.08% |

The control reproduces round 1's direction (treatment invents ~1.8× the
placebo). The grounded fix inverts it: the treatment now invents at
0.34× the placebo rate — 5.5× fewer inventions than the control
(p = 0.0215). The grounded arm used real retrieved metadata in 54
domains and 22 dates; the control in almost none.

**3. Why the grounded arm scores WORSE on the gate: mostly the
instrument.** The judge compares each rewrite against the original
chunk only. Moving real retrieved provenance into the prose — the whole
purpose of contextualization — therefore reads to the judge as "added
content." Measured, not guessed: 62% of v4's failures are "the rewrite
added something"; 24.5% are added attribution that the frozen rubric
explicitly says must PASS; and within v4, chunks that correctly
injected retrieved metadata pass 16 points lower than chunks that did
not (p = 0.019). Correcting only the explicit rubric violations puts v4
at 0.6452 — identical to the control. (That correction is a keyword
sort of the judge's stated reasons, not a re-judgement, so treat it as
indicative.) Honesty check: this bias explains part of the drop, not
all of it — v4 would still robustly fail without it. Note v4's raw
fidelity is *better* than the control's (number recall 0.9227 vs
0.9150; proper-name recall 0.9015 vs 0.8869).

**4. One genuine cost, not an artifact.** The grounded prompt's
restrictions made the model skip attribution entirely on 17.8% of
chunks — 3.2× the control (1,043 vs 325). A treatment that stops
contextualizing drifts toward the placebo, which would weaken the
treatment-versus-placebo comparisons in any later training run. This is
a real weakness of the v4 prompt to fix or accept.

**5. New instrument finding: the judge disagrees with itself on 19% of
identical inputs.** The two runs' placebo arms produced 99.88%
byte-identical rewrites (same prompt, weights, seed). Judging the same
texts twice at temperature 0 agreed on only 329/405 = 0.8123. That
self-agreement is the ceiling on the judge's agreement with anything —
and it agreed with the human labels at 0.735, close to that ceiling, so
most human-judge disagreement is judge noise, not hidden bias. Scale
check: judge noise moves a whole-arm rate by roughly ±2 points; the
10.7-point v4-vs-v3 gap is about 6× that, so it stands. The invention
result is deterministic and unaffected. This confirms the standing
rule: use this judge only as a rate instrument, never to adjudicate
single items.

**6. Reproducibility, three ways.** The placebo re-run matches round 1
to 0.24 points (0.6190 vs 0.6214); the control matches round 1's
re-judged C1 to 0.5 points (0.6452 vs 0.65); and a serving comparison
banked before trimming the hosted arm shows bf16 scores 3.8 points
above the hosted 4-bit quantization with identical invention — the
hosted gate was mildly conservative, changing no verdict.

**7. DECISION PENDING (owner).** The amendment split cleanly: invention
fixed and proven; fact-preservation-vs-chunk still failing,
substantially for instrument reasons. The two honest ways to proceed:
(a) report exactly as pre-registered — the invention fix succeeded, the
frozen gate remains unmet; or (b) amend the gate to score rewrites
against the chunk PLUS the retrieved metadata, which would be a new
deviation (Deviation 5) requiring owner ratification before any use.
Nobody acts on (b) without the owner's word. Three stronger-rewriter
arms (Llama 3.3 70B and Llama 4 Scout, via AnvilGPT) are still running
and report under the same rules.

## Amended gate re-run — FINAL results, all seven arms (2026-08-26)

The three stronger-rewriter arms finished (records in ctxprereg
`runs/2026-08-23/` through `runs/2026-08-25/`; run order and every
outage, restart, and re-order is in the run manifests). Execution notes
of record: llama3.3:70b went down service-side on AnvilGPT once,
costing one 13-hour rewriting pass (the rerun is the arm reported
here); AnvilGPT serves 70B-class models at ~8 calls/min regardless of
client workers; hosted decoding is not seeded, and an accidental
repeat of one arm's first pass measured the run-to-run noise at 4
chunks out of 5,860. Arms 2 and 3 and the rerun of arm 1 ran C1 only —
a coordinator-approved evidence ordering (the pre-registered rule
"every rewritten arm must meet the threshold" means the first failing
criterion decides); any C1 pass would have required its C3 before
being called a pass, and none passed.

**1. Gate verdicts, final — seven arms, no pass.**

| Arm | Model | Prompt | Gate | Literal | Calibrated | Robustness |
|---|---|---|---|---|---|---|
| Stronger v3 | llama3.3 70B | v3 | 0.7429 | FAIL | FAIL | nominal_fail |
| Control | qwen 7B | v3 | 0.6452 | FAIL | FAIL | robust_fail |
| Placebo | qwen 7B | C3 | 0.6190 | FAIL | FAIL | robust_fail |
| Grounded | qwen 7B | v4 | 0.5381 | FAIL | FAIL | robust_fail |
| Stronger+grounded | llama3.3 70B | v4 | 0.5286 | FAIL | FAIL | robust_fail |
| Stronger v3 | llama4 109B | v3 | 0.5119 | FAIL | FAIL | robust_fail |

The llama3.3-with-original-prompt arm (0.7429) is the only arm to land
INSIDE the calibration band (0.653–0.911) rather than below it — a
nominal fail, not a robust one. It is the one arm where the 50-item
human calibration is load-bearing; a larger calibration set could move
that verdict either way.

**2. Metric correction, self-caught and recorded.** The raw
invented-source rate is confounded by how much sourcing an arm
attempts: an arm that never names a source cannot invent one.
(llama3.3+v3 emitted only 29 specific source details in 420 chunks —
its near-zero invention rate was abstention, not accuracy.) The honest
measure is **attribution precision**: of the specific source details an
arm emits, how many are real.

| Arm | Details emitted | Wrong | Precision | 95% CI |
|---|---|---|---|---|
| qwen 7B + v3 | 34 | 11 | 0.676 | 0.508–0.809 |
| qwen 7B + v4 grounded | 78 | 2 | 0.974 | 0.911–0.993 |
| llama3.3 70B + v3 | 29 | 1 | 0.966 | 0.828–0.994 |
| llama3.3 70B + v4 grounded | 182 | 9 | 0.951 | 0.909–0.974 |
| llama4 109B + v3 | 164 | 25 | 0.848 | 0.785–0.895 |

**3. What the amendment concludes.**

- **Grounding (workstream b) succeeds at both scales, by different
  routes.** On the 7B it fixes precision (0.676 → 0.974, p = 2.7×10⁻⁵)
  and doubles volume. On the 70B it unlocks volume — 6.3× more
  contextualisation (29 → 182 details), the most of any arm — at
  statistically unchanged precision (0.966 → 0.951, p = 1.00).
- **Scale alone (workstream a) fails.** llama4 with the original
  prompt has the worst precision of any metadata-using arm (0.848),
  the worst gate score (0.5119), and 2.3× the 7B's raw invention.
  Both large models fabricated only dates — every single invention
  was a year, zero domains — exactly the field the crawl-date staging
  defect poisons.
- **The frozen gate fails every arm**, substantially for instrument
  reasons: it compares rewrites against the chunk alone, penalising
  the retrieved provenance the method exists to add (measured −16
  points for metadata-injecting chunks, p = 0.019), and the judge's
  own test-retest ceiling is 0.81.

The owner decision in item 7 above is now the only open item for this
study: report as pre-registered (a), or ratify Deviation 5 (b) and
re-score against chunk plus retrieved metadata.

## Original draft status

Status: **DRAFT v1, 2026-08-04 — for professor sign-off.** Start building the
pipeline after approval. Lock the design after the fact-accuracy check in §8.
Target venue: **ICML 2027**, around Jan 28; confirm the date before lock.

Paper type: **METHOD**. Under the owner's definition, this changes the data and
the workflow. It challenges the usual assumption that every sentence in
pretraining text is a true statement. Check record: [[Method-Gates-2026-08]].
Related CVPR diagnostic: [[Prereg-1NFE-Diversity]].

---

## 1. The problem

Language models train as if every sentence in their data is true. A rumor, a
sales pitch, a disputed claim, and a physics fact all enter the model in the
same form. This may help explain three known problems. Models can be poorly
calibrated on disputed topics. They flatter people by agreeing with the loudest
source. And they "adopt" goals that appeared in text they merely read.

Bengio's Scientist AI program proposes a fix. It is set out in the [Feb 2025
position paper (2502.15657)](https://arxiv.org/abs/2502.15657) and the [Jun 2026 technical paper
(2606.29657)](https://arxiv.org/abs/2606.29657). The fix is to rewrite the data so that each record shows the
difference between *"X is true"* and *"source S claimed X at time T in venue
V."* The technical paper defines this in **Definition 3.22** and says:
**"contextualization is specified solely at the level of requirements (i.e., we
do not provide a completed algorithm)."** We corrected this citation on
2026-08-02. The quote and Definition 3.22 are in 2606.29657, not in the
position paper. We checked both PDFs.

The direction was named eighteen months ago, and its requirements have been
formal for one month. Nobody has built it, including [LawZero](https://lawzero.org). LawZero has no
public code and is still hiring a pipeline team. We will build the first
version and measure what it actually changes.

## 2. What recent work has shown (checked 2026-08-03)

- **No working implementation exists.** We checked again, including the most
  recent weeks.
- **The closest idea adds tags, which is a different thing.** Source-aware
  training ([2404.01019](https://arxiv.org/abs/2404.01019)) connects *document ID tags* with content, so a model
  can name its sources. It uses synthetic data and does not rewrite the text.
  It does not measure calibration or sycophancy. [CTRL-style codes](https://arxiv.org/abs/1909.05858) also add
  labels to the input. [GenProve](https://arxiv.org/abs/2601.04932) tracks the source of generated text. None of
  them changes the text itself into records that show the limits and source of
  a claim.
- Rewriting large datasets is already practical. Examples include denser
  rewrites and ["Synthetic Rewriting as a Quality Multiplier"](https://arxiv.org/abs/2603.24826), which uses
  placebo controls. That shows the *pipeline* can work. A placebo arm is a
  basic control. Our new contribution is the schema for claims, and the
  measured effects of using it.
- **Ready evaluation tools, checked directly:** ConflictBank ([2408.12076](https://arxiv.org/abs/2408.12076)) for
  conflicting knowledge; "How LLMs Balance Internal Knowledge with User and
  Document Assertions" ([2604.22193](https://arxiv.org/abs/2604.22193)), which almost exactly matches our
  question; standard sycophancy test sets; and standard ability tests, to
  measure any cost.

## 3. The method

**Data format**, inspired by Definition 3.22 and made concrete by us: keep
direct factual observations as statements. Rewrite communicated claims so that
they name their source, venue, date, stance, and limits. Write the records in
normal language, with no required special tokens, so the method works with any
model design.

**Rewriter:** use an open 4–8B model with a tightly limited prompt and
automatic rule checks. Before rewriting the full dataset, check fact accuracy
on samples from different groups. Use both an LLM judge and a human
spot-check. Fix a pass threshold before the check runs.

**Dataset:** about 2B tokens from a [DCLM](https://arxiv.org/abs/2406.11794) or [FineWeb](https://arxiv.org/abs/2406.17557) subset. Add a
larger share of news, forums, and reviews, because they contain many disputed
claims and give the method something to change.

**Training plan**, revised 2026-08-04 after the owner's concern that training
from scratch costs too much: continue pretraining — that is, mid-training — on
the fully open [OLMo-2-1B](https://huggingface.co/allenai/OLMo-2-0425-1B) base model. Its original training data is public, so
we know what the model saw before our change. This is also how the method would
be used in practice. Labs do not restart pretraining. They would add
contextualized data during mid-training or during learning-rate annealing. Keep
one from-scratch arm, 0.5B × 8B tokens, as a **later test allowed only by the
rule in §4**. Run it only if Phase 1 helps.

## 4. Exact experiment plan

**Four data arms, each with 2 seeds.** That gives 8 continued-pretraining runs
on OLMo-2-1B, with 2B tokens each.

- C0: raw dataset.
- C1: **contextualized**, with the full claim-aware rewrite.
- C2: **tags only**. Add the same metadata as tags, but leave the text
  unchanged. *This is the deciding control.* It tells us whether rewriting
  helps beyond metadata. It also answers the likely objection that we
  rediscovered source-aware training.
- C3: placebo rewrite. Paraphrase the text while matching token budget and
  perplexity. This controls for the possibility that any rewrite helps.

**Predictions, fixed before the run:**

- **H1:** C1 beats C0, C2, and C3 on calibration during source conflict. On
  ConflictBank, measure accuracy and expected calibration error (ECE) on items
  with conflicting sources. ECE says how far a model's stated confidence is
  from how often it is actually right; lower is better.
- **H2:** C1 reduces sycophancy more than every control does.
- **H3:** C1 improves the assertion-balancing measurements from
  [2604.22193](https://arxiv.org/abs/2604.22193) more than C2 does. In other words, rewriting adds a benefit
  beyond tags.
- **H4:** C1 loses no more than 2.0 points on average across the standard
  ability test set, compared with C0.
- **H5, the rule for allowing the extra phase**, which replaces the older scale
  hypothesis: if H1–H3 are positive, train the from-scratch 0.5B × 8B-token
  experiment with 4 arms × 2 seeds. It costs about +300 GPU-h, including the
  extra rewriting. It tests whether C1 still beats C2 when the model has no
  earlier history of raw-text pretraining. Run this **ONLY** after a positive
  Phase 1. Its result can make the paper stronger, but it does not decide
  whether the Phase 1 paper succeeds.

**Decision rules:** use bootstrap confidence intervals over seeds, and the Holm
correction across H1–H5. A bootstrap re-samples the same data many times to
show how stable a result is. The Holm correction guards against false positives
when several hypotheses are tested at once. Freeze every measurement and prompt
when the design locks. Before rewriting at scale, fix the fact-preservation
pass threshold at ≥97% on the checked samples.

**An alternate result, and not a reason to stop:** if C2 is about equal to C1
on every main measurement, report that honestly: "metadata conditioning is
enough—the cheap version of Scientist AI's data mechanism." That is still a
method result, because tags cost almost nothing and rewriting does not. This
branch is fixed in advance.

**Reasons to stop.** (i) If the rewriter fails the fact-accuracy check after
two rounds of prompt improvement, stop. Report the pipeline difficulty as a
short, honest note, not as a paper. (ii) If LawZero or another group releases
an implementation before our lock, re-check the research space within 48h. Our
tag-only control and effect measurements may still work as an *evaluation* of
their method, but the paper's framing must change.

**Claims we will not make:** any safety *guarantee*. We test starting claims
and possible benefits, not the Scientist AI theorem. We also will not claim
results at frontier scale, and we will not claim we prevent goal adoption
beyond what the test suite can stand in for.

## 5. What each possible result means

- **Main result:** contextualized models are better calibrated during source
  conflict and less sycophantic, with a small loss of general ability. That is
  the first evidence for a widely discussed but never built proposal. Release
  the pipeline, data, and checkpoints.
- **Tags are enough:** release and validate the cheap approach, and report the
  real cost of the more expensive rewrite.
- **No effect:** provide the first evidence that the Scientist AI data idea does
  not give its promised benefits at academic scale. That is publishable, and it
  is the field's only measurement either way.

## 6. Resources and schedule

**Revised Phase-1 cost: about 300–450 GPU-h.** Rewriting 2B tokens with a 4–8B
model costs about 150–250 H100-h. Eight continued-pretraining runs at 1B × 2B
tokens cost about 100 H100-h. Evaluations cost about 30. The from-scratch
second phase adds about 300 GPU-h, and only after a positive Phase 1.

**OrangeGrid and Anvil are enough for Phase 1.** Delta H200 may help with
rewriter batch speed, but it is no longer required. Work on the schema, the
rewriter, the automatic checks, and the fact-accuracy audit can begin now,
without using the GPUs that the ICLR papers need.

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
- Nobody knows the effect sizes. The chosen scales, and the part of the dataset
  that is heavy in disputed claims, make a measurable effect more likely. H5
  checks whether a result is only an artifact of small models.

## 8. Checklist before locking

1. Professor approval of §4, especially C2, the fact-accuracy threshold, and
   which results lead to an alternate claim rather than to stopping.
2. Pass the fact-accuracy check on the pilot data.
3. Repeat the literature search, and name the most recent 8 weeks directly.
4. Confirm the exact ICML 2027 deadline.
5. Mark the page LOCKED and record the git hash.

## Related

[[Method-Gates-2026-08]] · [[Unified-Direction-Ranking-2026-08]] ·
[[GPU-Resources-Across-Clusters]] · [[Data-Transfer-Between-Clusters]]

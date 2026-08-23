# Pre-registration: Do Robot Policy Evaluators Recover the Human Ranking?

## Words used on this page

Read this short list once. Every word below is used the same way everywhere on
this page.

- **Policy** — the program that controls the robot. We compare policies.
- **Episode** — one recorded attempt by a robot at one task.
- **Evaluator**, also called a **judge** — a program that watches an episode
  and scores it, so a person does not have to watch it.
- **Ranking** — the list of policies ordered from best to worst.
- **Kendall τ** (say "tau") — a number between −1 and +1 that says how well two
  rankings agree. +1 means the two orders match exactly. 0 means no relation.
  −1 means one order is the exact reverse of the other.
- **Bootstrap** — a way to show how stable a result is. It re-samples the same
  data many times and re-computes the answer each time.
- **Confidence interval (CI)** — the range of values the data still allow. If
  the interval does not contain zero, chance alone is an unlikely explanation.
- **Bradley–Terry model** — a method that turns many head-to-head comparisons
  into one ranking.
- **Arm** — one version of the experiment, run to answer one question. The
  arms in this study are named A1 to A5.
- **Blind arm** — a run in which the evaluator never sees the video pixels. It
  shows how much agreement is possible with no picture at all.
- **Shard** — one small chunk of the work, processed as a unit.
- **Manifest** — the file that lists everything a run produced, so an auditor
  can check that nothing is missing.
- **Contamination** — the evaluator was trained on some of the very episodes we
  now test it on. Its score on those episodes is not a fair test.
- **Kernel** — a small program that runs on the GPU.
- **Noise floor** — how much a score moves when nothing meaningful changes, for
  example when a different but numerically equivalent kernel runs. A result
  smaller than this floor means nothing.
- **Deviation** — any change made after the plan was locked. Every deviation is
  written down on this page.

## Campaign result (2026-08-10)

The full table is in robojudge `runs/judge/full-2026-08.MANIFEST.md` §RESULTS.

**The main prediction H1 is TRUE, and it is not a close call.** Four of the six
evaluators put a different policy on top than humans do.

**Headline: SigLIP2 frame-text similarity disagrees with humans, and it
disagrees in the wrong direction.** Its agreement with human judgment is τ
−0.714, CI [−0.905, −0.238]. The minus sign is the point. SigLIP2 puts the
policy that humans rank last in first place.

**These evaluators are not noise.** All six video-arm CIs exclude zero. Five
are positive and one is negative. So the evaluators carry real signal, and one
of them points backwards.

**Only the two RoboReward judges pick the same best policy as humans.**
Contamination raised RoboReward-4B by about 0.10 τ. The double-reporting rule
caught this. Every RoboReward number is reported twice: once with all episodes,
and once with the contaminated episodes removed.

**H5 holds in its strongest form.** RoboRewardBench scores judges with MAE
(mean absolute error), where lower is better. That MAE varies by 1.8× across
judges whose ranking faithfulness cannot be told apart statistically. So the
benchmark number does not measure whether a judge gets the ranking right.

**The judges are over-confident.** They confidently order 20–60% of the pairs
that humans cannot separate.

**The blind floor could not be computed as planned.** Blind τ is undefined for
a structural reason (§6b; see also Deviation 2 below). The duration-only floor
carries the no-pixel finding instead, and its CI spans zero.

Three caveats must travel with the results table.

| Caveat | What it means |
|---|---|
| Intervals span about 0.24–0.91 | The data separate the sign of the agreement and the top-1 choice. They do NOT separate one judge's ranking from another judge's ranking. |
| Cosmos | Read its number against its own 32.8% kernel-noise floor. |
| Every floor is axis-named | For InternVL3 the floor is 1.6% from the torch version and 17.2% from the attention implementation. |

**One near-miss, recorded on purpose.** A bug in one metric would have reported
the mute blind arms — the arms that get no video — as "most over-confident."
The worker caught the bug, wrote a unit test, and kept the superseded values.

## 72B appendix arm (2026-08-23; lock item 8 option exercised, not a deviation)

Lock item 8 dropped the 72B judge from the headline roster on cost and
allowed it only as an optional appendix arm if spare compute appeared.
Spare compute appeared (the tail of Anvil job 19933424), so the option was
exercised: Qwen2.5-VL-72B ran the full protocol unchanged — same frozen
index, human ranking, aggregation code, and pinned environment — for 6.4
H100-hours, one seed, post lock. Record: robojudge
`runs/judge/appendix72b-2026-08-15.MANIFEST.md`; the paper carries it as
Appendix app:72b (ICLR repo commit 199c32a).

**Result: the cost decision cost nothing measurable.** The 72B's agreement
with the human ranking is identical to the 7B's — tau +0.714, rho +0.857,
to three decimals. Ten times the parameters bought no extra rank
agreement.

**Guardrail, written before anyone quotes the top-1.** The 72B picks the
human top-1 and the 7B does not. This must NOT be read as "scale fixes
top-1": with only 7 policies, tau moves in steps of about 0.095, the two
confidence intervals overlap almost entirely, and one more evaluator
landing on the human top-1 is another instance of H1's top-1 instability,
not a correction to it. Single unreplicated post-lock arm, one seed.

**Second finding: the larger judge commits less.** It scores 1 or 5 in
77.0% of episodes (middle of the scale: 5.1%) and therefore ties on 48.1%
of sessions against the 7B's 36.1% — it expresses no preference on nearly
half its comparisons. It is also less over-confident (1 of 5
humanly-inseparable pairs ordered, versus 2 for the 7B). An observation,
not a calibration claim.

Parse compliance was perfect under both the locked and the widened rule
(the judge emitted exactly five distinct strings in 10,212 replies; zero
rows rescued), and the blind arm was degenerate exactly as every campaign
blind arm is — all sessions tie with byte-identical text, valuable only as
proof the blind path never opened a video.

Housekeeping found during this arm: `configs/judge-env-requirements.txt`
still pinned torch 2.9.1, which silently downgraded fresh rebuilds off the
locked 2.11.0. No recorded number is affected (all campaign shards recorded
2.11.0). The file is corrected in the working tree; if the paper's artifact
release ships it, it must ship the corrected copy.

## A3 addendum: the confound controls (campaign closed)

A3 is the set of control runs that test hidden factors.

**Frame order does not matter to these judges.** Shuffling the frames changes
18.2% of InternVL3's episode scores. Yet the ranking it produces stays
IDENTICAL. Evaluators that judge task progress are indifferent to the order of
the frames. That is a confound finding: a factor we worried about does not
drive the result.

**The one-frame arms confirm what we predicted about SigLIP2.** One early frame
carries little information about how the episode ends. It gives τ +0.524, with
an interval that spans zero and with the correct top-1 policy. Three frames
that include the outcome give τ −0.714, which is significant. So the
anti-correlation lives in the outcome frames.

**We report sign flips carefully.** Both one-frame sign flips have intervals
that cross zero. We report them as "agreement lost," never as "reversed." The
only reversal we can assert is SigLIP2's video arm.

**Record keeping.** All jobs finished with zero failures. Archives were
verified by manifest count on both clusters. The campaign manifest is the
complete record for the writing phase; its §5b approval table carries the
commit hashes.

## Status: LOCKED 2026-08-08

The professor signed off at the meeting and adopted every recommended default.
Lock hash: `ad85987d4b5e13c2e59c2afd4aa557fc5178338a`. Any change after this
point is a deviation and must be logged as one.

Here are the locked answers to the nine open questions.

1. Camera view: use one exterior camera view. Take the right view first, then
   the left, then the wrist.
2. Cosmos: keep the 32-frame cap. Log this as our deviation from its model
   card.
3. InternVL3: use the `-hf` conversion, with the 20-episode comparison smoke
   test.
4. Aggregation: use induced-preference Bradley–Terry, which is the layer we
   validated. Never use the mean score.
5. H5: restate it to the real statistic of RoboRewardBench, which is MAE, where
   lower is better. Limit H5 to the four judges on that leaderboard.
6. Contamination: report every RoboReward number twice, full and clean. State
   this in the abstract.
7. Parse failures: if more than 5% of an arm's replies cannot be parsed, report
   that arm as non-compliant and keep it out of the headline.
8. The 72B judge is DROPPED from the headline roster. The 7B judge covers that
   model family. Run the 72B only as an optional appendix arm, and only if
   spare Anvil time exists.
9. Environment: pin torch 2.11.0+cu128, transformers 4.57.1, and PyAV/decord.
   Cosmos also gets three extra things — a stated reproducibility caveat, a
   measured kernel-noise floor printed beside its agreement statistic, and a
   64-episode pre-shard before its budget is final.

The four choices pinned in week 1 stand unchanged.

## Deviation 1 (2026-08-09, owner-ratified)

**What went wrong:** lock item 7 fixed a parse regex. Applied literally, that
regex marked good outputs as failures. Cosmos follows its own published format:
an `<answer>` tag holding a bare digit. The "ANSWER:" prefix that our regex
looked for inside the tag was our harness's own addition. 100% of these
"failures" carried valid scores in the correct tag.

**What changed:** inside a declared answer tag, a bare 1–5 now parses. For
judges that declare a tag, this widened rule gives the headline number. BOTH
parse rates are reported side by side. The >5% non-compliance threshold is
unchanged.

**Who decided:** the owner ratified it. Verbatim: "widen the rule for
tag-declaring judges, ratified".

## Deviation 2 (2026-08-09, coordinator-approved, owner-ratified)

**Who decided:** the coordinator approved it. The owner ratified it on
2026-08-09. The date in the prose was corrected to the commit date of 1cdd0d2.
Owner ratification, verbatim: "Deviation-2 ratification, and follow the best
practice for those decisions".

**What went wrong:** the pre-registered language-only blind floor cannot say
anything on this dataset. Both policies in a comparison share the same task
instruction. So the blind prompts are byte-identical, and every session ties
by construction. We verified this with a digest over all 2,552/2,553 sessions.
H2 therefore resolves as trivially false by design, and we will report it that
way. We will never report it as "language priors carry no signal."

**What we approved instead.** All three items were already declared in §4 and
A3 of this plan.

- Run the four confound cells that can be implemented: first_frame and
  shuffled, for each of the two frames-judges.
- Run the duration-only regression on the analysis side. Only 19.1% of sessions
  are informative, so the interval is printed beside it.
- Extend lock item 9: measure a kernel-noise floor for every judge, not only
  for Cosmos. The reason: a six-token judge moved 17.2% of its scores under a
  pure attention swap, so short answers cannot be assumed stable.

**What we did NOT approve.**

- swapped_instruction. It needs a seeding and exclusion rule that we never
  designed. It is named as future work.
- Frame-fed versions of the judges that have their own native video pipeline.
  Feeding them frames would audit a different judge. This runs only if the
  owner explicitly asks for it, and it would be labeled as a modified judge.

**Process note:** the worker held the coordinator's first A3 order, because it
would have produced corrupt evidence for the four native-pipeline judges. The
order was then revised. The hold is part of this record.

**Timing note for auditors:** the ratification reached the running worker
through the coordinator session before the campaign was submitted. This ledger
entry was committed about seven minutes after that submission.

## Ledger addendum (2026-08-09)

**What is recorded:** the coordinator ratified three environment fixes as the
implementation of lock item 9. The three fixes are the PyAV decoder override,
the PATH in the submit template, and the env.sh pins.

**Who decided, and when:** the coordinator, ratified 2026-08-09. This line
gives that ratification its durable place in the ledger.

**Why:** the lock pins the environment, and these code changes are what it
takes to honour that pin. The full test suite validated them.

## Judge-arm bring-up result (2026-08-08; read before locking)

**The harness works.** It is built, verified by dry run, and it runs the moment
a GPU appears. The first measured costs were about 46 L40S-hours on OrangeGrid
plus about 12 H100-hours for the 72B judge, which fits only on Anvil. The
throughput measurement below supersedes both numbers.

**Two sanity checks passed.** Fed the human preferences, the aggregation layer
reproduces the frozen human ranking to 5e-07. The blind arms structurally
cannot open video files.

**Material finding: RoboReward was trained on 25.9% of our audit episodes.**
That is 1,322 of 5,106 episodes, counted from the data recipe in its own paper.
The rate per policy runs 20–31%, uneven enough to move a ranking. So every
RoboReward number will be reported twice, full and contamination-clean. The
clean list is already computed.

**Correction needed at lock: H5 names the wrong statistic.** The
RoboRewardBench metric is MAE, where lower is better, not accuracy. Two of our
judges are not on that leaderboard. So H5 must be restated for four judges.

**Owner authorizations (2026-08-08):** `trust_remote_code=True` is approved for
exactly three pinned revisions, never for a floating one.

| Model | Pinned revision | Note |
|---|---|---|
| RoboReward-4B | 4dec8af8 | — |
| RoboReward-8B | 3a185b4f | Later found unnecessary: the snapshots ship no custom code |
| OpenGVLab/InternVL3-8B | 853e3a79 | For the locked Q3 conversion comparison only |

## Throughput measurement result (2026-08-08, late)

This section supersedes the cost guesses above.

**The full OrangeGrid judge campaign costs 23.7 L40S-hours** on torch 2.11. On
the older torch 2.9.1 pin it would cost 112 hours. That old pin has a
pathologically slow conv3d kernel. It is also the numerical outlier: torch 2.11
and an independent reimplementation agree byte-for-byte against it.

**Pin at lock:** torch 2.11.0+cu128, transformers 4.57.1, and PyAV/decord for
decoding, which give bit-identical pixels. Note that torchvision 0.26 removed
read_video.

**New lock item: Cosmos-Reason2 scores are not reproducible across environments
that are numerically equivalent.** One bf16 rounding step, amplified over about
200 chain-of-thought tokens, changes 4–7 of 20 scores. Short-answer judges were
far steadier at 19–20 of 20, and all blind arms were stable at 20 of 20. So
Cosmos needs three things: a stated caveat, a measured kernel-noise floor
reported next to its agreement statistic, and a 64-episode shard run before its
budget is fixed. Cosmos is 82% of the campaign.

The eight open lock questions and their recommendations are in
`robojudge/SMOKE_PLAN.md` and in the 2026-08-08 worker report
(`runs/judge/2026-08-08/`).

## Original draft status

Status: **DRAFT v1, 2026-08-03 — for professor sign-off.** Lock after the
week-1 go/no-go check in §8. After lock, do not change the predictions, the
experimental arms, the measurements, or the decision rules. Report any later
change as a deviation from the plan.

Paper type: **diagnostic**. A diagnostic paper checks whether current tools
work. Standing rule 5 also requires a constructive method part, so this study
includes one. Target venue: ICLR 2027. Abstracts are due Sep 18 and full papers
Sep 25. Related method paper: [[Prereg-Crop-Consistency-Distillation]].

---

## 1. The problem

A proper test of a robot policy needs many runs on a real robot. That is slow,
expensive, and out of reach for many labs. So researchers are trying to replace
real tests with **automatic evaluators**. These include reward models that
score episode videos, vision-language models (VLMs) that act as judges, and
world models that simulate an episode with no robot at all.

These evaluators now affect every later decision: which policy to deploy, which
training recipe won, and which paper's method is better. **If an evaluator is
wrong, the field's ranking is wrong.** We may not notice, because the same
groups often build the evaluators and test them.

## 2. What recent work has shown

Eleven evaluator systems appeared in about ten months: [WorldEval](https://arxiv.org/abs/2505.19017),
[WorldGym](https://arxiv.org/abs/2506.00613), [Ctrl-World](https://arxiv.org/abs/2510.10125), [RobotArena∞](https://arxiv.org/abs/2510.23571), [PolaRiS](https://arxiv.org/abs/2512.16881),
[SC3-Eval](https://arxiv.org/abs/2606.18610), [RoboWorld](https://arxiv.org/abs/2607.01060), [dWorldEval](https://arxiv.org/abs/2604.22152), [PiL-World](https://arxiv.org/abs/2606.05773),
[GigaWorld-1](https://arxiv.org/abs/2607.02642), and [RoboReward](https://arxiv.org/abs/2601.00675). Each group tests its own system on a
small set of policies. SC3-Eval reports "Pearson correlation of 0.929" on
**seven** policies. RoboWorld similarly reports "Pearson's r = 0.989." Our
area-wide search in Aug 2026 found **no independent audit of any of them**.

Three facts show exactly what is missing.

1. A position paper ([arXiv 2606.15032](https://arxiv.org/abs/2606.15032)) names the checks the field needs:
   "policy-ranking agreement," "model exploitability," and "uncertainty
   calibration." It runs none of them and has "zero experiments."
2. This kind of audit can be published in nearby areas. A check of a tool-use
   benchmark ([2607.02577](https://arxiv.org/abs/2607.02577)) found that evaluators and humans disagreed 18.5% of
   the time. Our study asks the same kind of question for embodied AI.
3. **Public human ground truth already exists.** [RoboArena](https://arxiv.org/abs/2506.18123) released raw test
   data in [`RoboArena/DataDump_02-03-2026`](https://huggingface.co/datasets/RoboArena/DataDump_02-03-2026) under the MIT license. It
   contains 3,284 double-blind human pair comparisons and 9,589 real-robot
   episodes from 15 policies. Each episode includes `binary_success`,
   `partial_success`, video, and a written reason.

   We corrected the counts from the parsed data on 2026-08-02, before lock.
   Exactly 7 policies have ≥600 episodes, with 1,068–1,431 each. The eighth,
   `pi05_droid`, has 564. The earlier statement "8 policies with 600–1,650"
   was wrong. The ≥600 RULE does not change. We correct the count rather than
   lowering a threshold to reach the number we expected. The system builders
   cannot give a fully independent audit of themselves. We can run one, and we
   need no robot to do it.

## 3. What is new in our study

We apply **one plan, fixed in advance, to several evaluators, and then compare
them with human ground truth.**

We gave up one novelty claim on 2026-08-02, before lock. The final search found
**[2606.01036](https://arxiv.org/abs/2606.01036) (ICML 2026, Tian/Wu/Bajcsy)**. That paper already uses
RoboArena's human pair labels as ground truth for three reward models. Its
agreement on rollout pairs falls from 0.72–0.77 on easy tasks to 0.52–0.62 on
Tool Use. So we no longer claim to be the first to use this data as evaluator
ground truth. We cite that paper instead, as evidence that motivates our study.

Their paper leaves the following pieces unanswered. It has:

- no policy leaderboard,
- no RoboReward and no VLM-judge family,
- no blind floor,
- no confidence intervals,
- no bad-episode injection, and
- no judge swap.

RoboArena's 68 citing papers and RoboReward's 37 citing papers contain no
independent audits either. So four contributions remain ours.

- **First audit of whether robot evaluators recover the policy-level ranking.**
  Test whether the evaluator's whole leaderboard matches the human preference
  ranking, not just its agreement on one pair. Show honest uncertainty for the
  number of policies people normally use.
- **The blind floor:** measure how much agreement an evaluator can reach when
  it never sees the image pixels and uses only language or episode duration.
  Nobody has measured this for an embodied evaluator.
- **Sensitivity to the number of policies:** repeatedly sample n=7 policies
  from our n=8-with-600-episodes set, and compute the CI behind claims like
  r≈0.93. One figure changes how every evaluator paper in §2 should be read.
- **Method part:** change RoboReward's episode scores so that they match the
  ranking goal on held-out policies. Also release an "evaluator report card"
  with rank-flip rate, blind floor, and minimum n, as a common standard.

## 4. Exact experiment plan

**Fixed data:** the RoboArena dump. The main set contains the policies with
≥600 episodes: exactly **7** on 2026-08-02, as corrected in §2. Before running
any evaluator, compute the human ranking once from the pairwise preferences
with a Bradley–Terry model. Count a tie as half a win for each side. Use a
session-level bootstrap with 10,000 draws, then freeze the ranking.

The ranking was **frozen on 2026-08-02**. It was then rebuilt after review with
the flip meanings corrected. The θ values are bit-for-bit identical. The files
are in robojudge `runs/ranking_freeze/2026-08-02/`. The top policy,
`pi0_fast_droid`, stays top in **94.0%** of draws. The bottom policy stays
bottom in 100% of draws. Neighbor flip probabilities are 2↔3 = 0.38, 3↔4 =
0.44, and 4↔5 = 0.22. So the middle of the human ranking cannot be separated
statistically.

**Evaluator arms:**

- A1: RoboReward-4B and RoboReward-8B, released as `teetone/*`.
- A2: ready-made VLM judges Qwen2.5-VL-7B/72B and InternVL3-8B, plus a SigLIP2
  similarity score from our cached-feature tools. Use a fixed prompt,
  temperature 0, and 3 sampled frames per view, unless the model reads video
  directly.
- A3, controls for hidden factors: language only with no pixels, first frame
  only, shuffled frames, swapped instruction, and a duration-only regression.
- A4, obviously bad episodes built offline from the dump. Two kinds: pairs
  where the instruction does not match the video, and no-progress pieces cut
  from failed episodes but shown as complete episodes. A good evaluator should
  put these at the bottom.
- A5, a [DreamGen](https://arxiv.org/abs/2505.12705) component: recreate the DreamGen Bench videos for the 4
  published video models, and score them with 5 judges — the A2 set plus
  Cosmos-Reason. Keep the published vector of later policy success fixed. Test
  whether the paper's "strong correlation" survives a change of judge.

**Main measurements:** Kendall τ and Spearman ρ against the human ranking, both
with bootstrap CIs. Spearman ρ is a second rank-agreement score, also running
from −1 to +1. Also report MMRV, so the numbers can be compared with earlier
work; MMRV is the rank-violation score those earlier papers use. MMRV was
introduced by [SIMPLER (2405.05941)](https://arxiv.org/abs/2405.05941) and used by SC3-Eval; we corrected
this credit on 2026-08-02. Measure the top-1 flip rate when the evaluator
changes. Define the blind floor as τ(A3-language-only)/τ(A1).

**Predictions, fixed before the run:**

- **H1:** at least one evaluator in A1–A2 changes which policy is on top. In
  other words, top-1 is NOT fixed across evaluators.
- **H2:** blind floor ≥ 0.5. We predict TRUE, because prior clues in the
  language carry most of the agreement.
- **H3:** the n=7 bootstrap CI for Pearson r contains 0.5. We predict TRUE.
- **H4:** at least one evaluator scores at least one obviously bad episode type
  above the middle real episode. We predict TRUE.
- **H5:** accuracy on [RoboRewardBench](https://crfm.stanford.edu/helm/robo-reward-bench) does not order the evaluators by
  ranking faithfulness τ. We predict TRUE, because benchmark accuracy and
  ranking validity are different things.
- **H6, DreamGen:** the ordering of video models changes with the judge, and
  the later-policy correlations across judges cover a range at least 0.4 wide.

**Decision rules:** judge each hypothesis with its named statistic and a 95%
bootstrap CI. Apply the Holm correction across H1–H6; the Holm correction
guards against false positives when several hypotheses are tested at once. **If
every predicted failure is absent, that full null is still a positive
publishable result.** It would be the first independent evidence that these
evaluators recover the ranking.

**Claims we will not make:** claims about evaluators or policies we did not
test; claims about running policies in a closed control loop; and claims that
one tested factor *causes* a failure, beyond what the A3 comparisons show.

## 5. What each possible result means

- **Expected from H1–H4:** rankings depend partly on the evaluator, the blind
  floor is large, the n=7 CIs are wide, and at least one obviously bad episode
  fools an evaluator. Main message: "current robot-evaluator agreement numbers
  are largely driven by prior clues and have too little statistical power."
  Release the report card and the adjusted evaluator that fixes the measurable
  part.
- **No predicted failures:** the evaluators reliably recover the human ranking.
  This becomes the first independent validation. The report card still ships,
  and the sample-size figure still improves reporting practice.
- In either case, release the `RoboJudgeAudit` tools, the report card, and the
  Bradley–Terry human-ranking reference, for future evaluator papers to use.

## 6. Resources and schedule

Cost: **250–500 GPU-h, inference only.** A5 adds 60–120 GPU-h for generating
video. Use OrangeGrid's 2×A100/L40S, which is free and has no time limits. Use
an Anvil A100 for the 72B judge if needed. The RoboArena download size is
**unverified**; checking it is part of week 1. Store the data on `$SCRATCH` and
keep manifests as described in [[Data-Transfer-Between-Clusters]].

Schedule: week 1 check and ranking freeze → weeks 2–3 A1/A2 → week 4 A3/A4 →
week 5 A5 → week 6 analysis and packaging → week 7 writing. The Sep 18 abstract
needs only the frozen plan and the H3 sample-size figure, and both can be made
from the dump in week 1.

## 7. Risks and competing work to watch

- If the RoboArena dump is too large, or has too few labels for Bradley–Terry,
  use the 2025-08 dump with 7,513 files, or use binary success per episode.
- Students near the evaluator authors Levine, Finn, Pertsch, and Liang could
  run their own audit. Search again every 6 weeks. Watch citations of
  [2606.15032](https://arxiv.org/abs/2606.15032) and of the tool-use audit.
- The original search did not cover enough work published only in venue
  proceedings. Run one clean confirmation search before lock, as §8 requires.

## 8. Week-1 go/no-go checklist

1. ~~Check download size and hydration, and parse one full session.~~ **DONE
   2026-08-02.** The size is 18.5 GB, checked through the API before download.
   All 3,284/3,284 sessions parse, with 0 failures. Preferences are exactly
   {A:1404, B:1401, TIE:479}.
2. ~~Freeze the human ranking with bootstrap CIs and make the H3 figure.~~
   **DONE 2026-08-02.** The files are in robojudge
   `runs/ranking_freeze/2026-08-02/` and `runs/h3_figure/2026-08-02/`. The H3
   figure was rebuilt after review, using unit-variance normal scores. At n=7,
   the raw-strength band is **[0.51, 0.96]** for r*=0.929. It does NOT contain
   0.5. The earlier band of [0.40, 0.95] used scores with too little spread.
   The copula band is [0.86, 0.99]. Use this wording in the abstract: "a true
   r*=0.929 yields measured r anywhere in [0.51, 0.96] at n=7."
3. **DECIDE BEFORE LOCK which Pearson scale H3 uses.** Option one is raw
   strength: the binning-droid outlier reduces it, and it keeps the
   pre-registered prediction. Option two is copula/normal scores: it is
   unbiased, and it reverses the prediction. Both are saved in `h3_draws.npz`.
   Decide with the professor, then record the choice and the reason here.
4. **DECIDE BEFORE LOCK whether to keep `paligemma_binning_droid` in the main
   set.** It passes the ≥600 rule, but it is a nearly useless outlier: 17W/511L
   and θ=−2.47. It controls most of the strength range. Default: keep it,
   because that follows the fixed rule.
5. ~~Repeat the literature search and name the most recent 8 weeks.~~ **DONE
   2026-08-02.** The partial conflict [2606.01036](https://arxiv.org/abs/2606.01036) changed our framing in §3.
   RoboWorld v4 now claims r=0.989, which makes the sample-size figure more
   useful. [ArmnetBench](https://arxiv.org/abs/2607.24481), with 3,118 human-scored episodes, could help a
   competitor move quickly, which makes our timing argument stronger.
   **Warning: OpenReview limited requests during the search. Run one manual
   OpenReview search of ICLR'27 and NeurIPS'26 submissions before lock.**
6. Get professor approval, mark the page LOCKED with the date, and record the
   git hash.

## Related

[[Unified-Direction-Ranking-2026-08]] · the removed autoresearch accept-rule draft (git history) ·
[[GPU-Resources-Across-Clusters]] · [[Data-Transfer-Between-Clusters]]

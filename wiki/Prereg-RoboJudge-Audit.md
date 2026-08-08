# Pre-registration: Do Robot Policy Evaluators Recover the Human Ranking?

**Judge-arm bring-up RESULT (2026-08-08; read before locking):** the
harness is built, dry-run-verified, and runs the moment a GPU appears
(measured costs: ~46 L40S-hours on OrangeGrid + ~12 H100-hours for the
72B judge, which fits only on Anvil). The aggregation layer reproduces
the frozen human ranking to 5e-07 when fed the human preferences, and
blind arms structurally cannot open video files. **Material finding:
RoboReward was trained on 25.9% of our audit episodes** (1,322 of
5,106, from its own paper's data recipe; per-policy rates 20–31%, uneven
enough to move a ranking) — every RoboReward number will be reported
twice, full and contamination-clean (the clean list is computed).
**Correction needed at lock: H5 names the wrong statistic** — the
RoboRewardBench metric is MAE (lower is better), not accuracy, and two
of our judges are not on that leaderboard, so H5 must be restated for
four judges. Eight open lock questions with recommendations:
`robojudge/SMOKE_PLAN.md` + the 2026-08-08 worker report
(`runs/judge/2026-08-08/`).

Status: **DRAFT v1, 2026-08-03 — for professor sign-off.** Lock after the
week-1 go/no-go check in §8. After lock, do not change the predictions,
experimental arms, measurements, or decision rules. Report any later change as
a deviation from the plan.

Paper type: **diagnostic**, meaning a study that checks whether current tools
work, plus a constructive method part required by standing rule 5. Target venue:
ICLR 2027, with abstracts Sep 18 and full papers Sep 25. Related method paper:
[[Prereg-Crop-Consistency-Distillation]].

---

## 1. The problem

A proper test of a robot policy needs many runs on a real robot. This is slow,
expensive, and out of reach for many labs. Researchers are therefore trying to
replace real tests with **automatic evaluators**. These include reward models
that score episode videos, vision-language models (VLMs) that act as judges,
and world models that simulate an episode without any robot.

These evaluators now affect every later decision: which policy to deploy, which
training recipe won, and which paper's method is better. **If an evaluator is
wrong, the field's ranking is wrong.** We may not notice because the same groups
often build and test the evaluators.

## 2. What recent work has shown

Eleven evaluator systems appeared in about ten months: [WorldEval](https://arxiv.org/abs/2505.19017),
[WorldGym](https://arxiv.org/abs/2506.00613), [Ctrl-World](https://arxiv.org/abs/2510.10125), [RobotArena∞](https://arxiv.org/abs/2510.23571), [PolaRiS](https://arxiv.org/abs/2512.16881),
[SC3-Eval](https://arxiv.org/abs/2606.18610), [RoboWorld](https://arxiv.org/abs/2607.01060), [dWorldEval](https://arxiv.org/abs/2604.22152), [PiL-World](https://arxiv.org/abs/2606.05773),
[GigaWorld-1](https://arxiv.org/abs/2607.02642), and [RoboReward](https://arxiv.org/abs/2601.00675). Each group tests its own system on a
small set of policies. SC3-Eval reports "Pearson correlation of 0.929" on
**seven** policies. RoboWorld similarly reports "Pearson's r = 0.989." Our
area-wide search in Aug 2026 found **no independent audit of any of them**.

Three facts show exactly what is missing:

1. A position paper ([arXiv 2606.15032](https://arxiv.org/abs/2606.15032)) names the needed checks:
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
   lowering a threshold to get the number we expected. The system builders
   cannot give a fully independent audit of themselves. We can run one without
   using a robot.

## 3. What is new in our study

Apply **one plan fixed in advance to several evaluators, then compare them with
human ground truth.** We changed one novelty claim on 2026-08-02, before lock.
The final search found **[2606.01036](https://arxiv.org/abs/2606.01036) (ICML 2026, Tian/Wu/Bajcsy)**. That paper
already uses RoboArena's human pair labels as ground truth for three reward
models. Agreement on rollout pairs falls from 0.72–0.77 on easy tasks to
0.52–0.62 on Tool Use. We therefore **give up** the claim that we are the first
to use this data as evaluator ground truth. Instead, cite it as evidence that
motivates our study.

The following parts remain unanswered. Their paper has no policy leaderboard,
no RoboReward or VLM-judge family, no blind floor, no confidence intervals
(CIs), no bad-episode injection, and no judge swap. RoboArena's 68 citing papers
and RoboReward's 37 citing papers contain no independent audits.

- **First audit of whether robot evaluators recover the policy-level ranking.**
  Test whether the evaluator's leaderboard—not just agreement on one
  pair—matches the human preference ranking. Show honest uncertainty for the
  number of policies normally used.
- **The blind floor:** measure how much agreement is possible for an evaluator
  that never sees the image pixels and uses only language or episode duration.
  Nobody has measured this for an embodied evaluator.
- **Sensitivity to the number of policies:** repeatedly sample n=7 policies
  from our n=8-with-600-episodes set to calculate the CI behind claims like
  r≈0.93. One figure updates how every evaluator paper in §2 should be read.
- **Method part:** change RoboReward's episode scores so they match the ranking
  goal on held-out policies. Also release an "evaluator report card" with
  rank-flip rate, blind floor, and minimum n as a common standard.

## 4. Exact experiment plan

**Fixed data:** the RoboArena dump. The main set contains policies with ≥600
episodes: exactly **7** on 2026-08-02, as corrected in §2. Before running any
evaluator, compute the human ranking once from pairwise preferences using a
Bradley–Terry model. Count a tie as half a win for each side. Use a session-level
bootstrap with 10,000 draws, then freeze the ranking.

The ranking was **frozen on 2026-08-02**, then rebuilt after review with correct
flip meanings. The θ values are bit-for-bit identical. Files are in robojudge
`runs/ranking_freeze/2026-08-02/`. Top-1 `pi0_fast_droid` stays top in **94.0%**
of draws. The bottom policy stays bottom in 100%. Neighbor flip probabilities
are 2↔3 = 0.38, 3↔4 = 0.44, and 4↔5 = 0.22. This means the middle of the human
ranking cannot be statistically separated.

**Evaluator arms:**

- A1: RoboReward-4B and RoboReward-8B released as `teetone/*`.
- A2: ready-made VLM judges Qwen2.5-VL-7B/72B and InternVL3-8B, plus a
  SigLIP2 similarity score from our cached-feature tools. Use a fixed prompt,
  temperature 0, and 3 sampled frames per view unless the model reads video
  directly.
- A3, controls for hidden factors: language only with no pixels, first frame
  only, shuffled frames, swapped instruction, and a duration-only regression.
- A4, obviously bad episodes built offline from the dump: pairs where the
  instruction does not match the video, and no-progress pieces cut from failed
  episodes but shown as complete episodes. A good evaluator should put these at
  the bottom.
- A5, a [DreamGen](https://arxiv.org/abs/2505.12705) component: recreate DreamGen Bench videos for the 4
  published video models and score them with 5 judges, the A2 set plus
  Cosmos-Reason. Keep the published vector of later policy success fixed. Test
  whether the paper's "strong correlation" remains when the judge changes.

**Main measurements:** Kendall τ and Spearman ρ compared with the human ranking,
with bootstrap CIs. Also report MMRV for comparison with earlier work. MMRV was
introduced by [SIMPLER (2405.05941)](https://arxiv.org/abs/2405.05941) and used by SC3-Eval; this credit was
corrected on 2026-08-02. Measure top-1 flip rate when the evaluator changes.
Define the blind floor as τ(A3-language-only)/τ(A1).

**Predictions, fixed before the run:**

- **H1:** at least one evaluator in A1–A2 changes which policy is top. In other
  words, top-1 is NOT fixed across evaluators.
- **H2:** blind floor ≥ 0.5. Predict TRUE because prior clues in language carry
  most of the agreement.
- **H3:** the n=7 bootstrap CI for Pearson r contains 0.5. Predict TRUE.
- **H4:** at least one evaluator scores at least one obviously bad episode type
  above the middle real episode. Predict TRUE.
- **H5:** accuracy on [RoboRewardBench](https://crfm.stanford.edu/helm/robo-reward-bench) does not order the evaluators by
  ranking faithfulness τ. Predict TRUE because benchmark accuracy and ranking
  validity are different.
- **H6, DreamGen:** the ordering of video models changes with the judge, and the
  later-policy correlations across judges cover a range at least 0.4 wide.

**Decision rules:** judge each H with its named statistic and a 95% bootstrap
CI. Use Holm correction across H1–H6. **If every predicted failure is absent,
that full null is still the positive publishable result:** it would be the first
independent evidence that these evaluators recover the ranking.

**Claims we will not make:** claims about evaluators or policies not tested;
claims about running policies in a closed control loop; or claims that one
tested factor *causes* a failure beyond what the A3 comparisons show.

## 5. What each possible result means

- **Expected from H1–H4:** rankings partly depend on the evaluator, the blind
  floor is large, n=7 CIs are wide, and at least one obviously bad episode fools
  an evaluator. Main message: "current robot-evaluator agreement numbers are
  largely driven by prior clues and have too little statistical power." Release
  the report card and the adjusted evaluator that fixes the measurable part.
- **No predicted failures:** evaluators reliably recover the human ranking.
  This becomes the first independent validation. The report card still ships,
  and the sample-size figure still improves reporting practice.
- In either case, release the `RoboJudgeAudit` tools, report card, and
  Bradley–Terry human-ranking reference for future evaluator papers.

## 6. Resources and schedule

Cost: **250–500 GPU-h, inference only.** A5 adds 60–120 GPU-h for generating
video. Use OrangeGrid's 2×A100/L40S, which is free and has no time limits. Use
Anvil A100 for the 72B judge if needed. The RoboArena download size is
**unverified**. Checking it is part of week 1. Store it on `$SCRATCH` and keep
manifests as described in [[Data-Transfer-Between-Clusters]].

Schedule: week 1 check and ranking freeze → weeks 2–3 A1/A2 → week 4 A3/A4 →
week 5 A5 → week 6 analysis and packaging → week 7 writing. The Sep 18 abstract
needs only the frozen plan and the H3 sample-size figure, which can be made from
the dump in week 1.

## 7. Risks and competing work to watch

- If the RoboArena dump is too large or has too few labels for Bradley–Terry,
  use the 2025-08 dump with 7,513 files, or use binary success per episode.
- Students near the evaluator authors Levine, Finn, Pertsch, and Liang could run
  their own audit. Search again every 6 weeks. Watch citations of [2606.15032](https://arxiv.org/abs/2606.15032)
  and the tool-use audit.
- The original search did not cover enough work published only in venue
  proceedings. Run one clean confirmation search before lock, as required in
  §8.

## 8. Week-1 go/no-go checklist

1. ~~Check download size and hydration, and parse one full session.~~ **DONE
   2026-08-02.** Size is 18.5 GB, checked through the API before download. All
   3,284/3,284 sessions parse, with 0 failures. Preferences are exactly
   {A:1404, B:1401, TIE:479}.
2. ~~Freeze the human ranking with bootstrap CIs and make the H3 figure.~~
   **DONE 2026-08-02.** Files are in robojudge
   `runs/ranking_freeze/2026-08-02/` and `runs/h3_figure/2026-08-02/`. The H3
   figure was rebuilt after review using unit-variance normal scores. At n=7,
   the raw-strength band is **[0.51, 0.96]** for r*=0.929. It does NOT contain
   0.5. The earlier [0.40, 0.95] used scores with too little spread. The copula
   band is [0.86, 0.99]. Use this abstract wording: "a true r*=0.929 yields
   measured r anywhere in [0.51, 0.96] at n=7."
3. **DECIDE BEFORE LOCK which Pearson scale H3 uses:** raw strength, which is
   reduced by the binning-droid outlier and keeps the pre-registered prediction,
   or copula/normal scores, which is unbiased and reverses the prediction. Both
   are saved in `h3_draws.npz`. Decide with the professor and record the choice
   and reason here.
4. **DECIDE BEFORE LOCK whether to keep `paligemma_binning_droid` in the main
   set.** It passes ≥600 but is a nearly useless outlier: 17W/511L and θ=−2.47.
   It controls most of the strength range. Default: keep it because that follows
   the fixed rule.
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

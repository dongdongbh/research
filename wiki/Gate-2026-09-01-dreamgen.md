# Method Gate — Ground-truth-anchored judge card for generated-video judges

Date: 2026-09-01. Direction rank 3 in
[[Direction-Audit-2026-09-01]]. Source evidence: `intact-b.md` item 2 and
`robojudge-feeds.md` item 1, both in this folder.

**Verdict: SURVIVES, but weaker than the audit rated it. ★★★, conditional.**
It is a **section of the RoboJudge ICLR submission, not a standalone paper.**

---

## Words used on this page

Read this list once. Every word below is used the same way everywhere.

- **Judge** — a program that watches a video and scores it, so a person does
  not have to watch it.
- **Generated video** — video made by a model, not filmed by a camera.
- **Real episode** — one recorded attempt by a real robot at one task.
- **binary_success** — a 0-or-1 label already recorded by the robot stack that
  says whether the attempt worked. We did not create it and we do not pay for
  it.
- **Pearson r** — a number from −1 to +1 that says how closely two lists of
  numbers rise and fall together.
- **Fisher-z interval** — the standard way to put a range of plausible values
  around a Pearson r. A short calculation, no data needed beyond r and the
  number of points.
- **d′** (say "d-prime") — how well a judge tells good cases apart from bad
  ones. 0 means it cannot tell at all. Bigger is better.
- **Criterion** — how soft or harsh a judge is. A soft judge says "success"
  too often. This is separate from d′.
- **Blind arm** — a run where the judge never sees the video. It shows how much
  agreement is possible with no picture at all.
- **Paired bootstrap** — re-sample the same episodes many times, recompute the
  answer each time, and read off the range. It gives a confidence interval.
- **Anchor** — a set of videos where the right answer is already recorded, used
  to check a judge that normally has nothing to check against.
- **Arm** — one version of the experiment, run to answer one question.

---

## 1. The method filter (the owner's, applied first)

**As written in the scan, "audit the DreamGen judge" FAILS.** An audit is
diagnosis. Diagnosis alone is not a method.

**Reframed, it passes on route 3: an old method applied to a new problem.**
The old method is our RoboJudge validity protocol, built for judges of real
robot episodes. The new problem is judges of **generated** robot video, where
no recorded outcome exists to check the judge against.

The transfer is not obvious, and that is the point. The whole reason a
generated-video judge is hard to check is that nobody knows the right answer
for a video a model invented. **Our answer is the anchor trick: run the
generated-video judge, completely unchanged, on real robot episodes whose
success was already recorded.** Per-episode accuracy then costs no new human
labelling at all.

**The shipped harness (named now, as the standing rule requires).** We extend
`RoboJudgeAudit`, the harness the locked RoboJudge prereg already promises to
release, with a **video-judge card**: for any video judge, it reports d′ with a
paired bootstrap interval, the criterion, the blind-arm score, the
prompt-disagreement rate, and a Fisher-z interval on any aggregated correlation
the judge's authors reported. One harness serves both the audit and the method.

**The method paper it unlocks:** the RoboJudge ICLR submission. DreamGen Bench
is one of that project's four named audit targets, so this is not a new
project. It is a new section of a paper we are already writing.

---

## 2. Second scoop pass — Level 3, down from the first pass's Level 4

A second, independent pass ran today with the `paper-search` tool across arXiv,
Semantic Scholar, OpenAlex, DBLP and Crossref, including a forced sweep of the
most recent ten weeks. **It found three papers the first pass missed. All three
narrow the claim. None kills it.** I re-verified every one of them myself from
the paper text, not from the search summary.

### The three narrowings, loudest first

**Loud: the name "Judge Card" is already taken, and so is the idea of a
standard reporting card that audits a judge.**
[Beyond Accuracy: Policy Invariance as a Reliability Test for LLM Safety
Judges](https://arxiv.org/abs/2605.06161) (2026-05-07) says in its own
abstract: "we contribute the Policy Invariance Score and the Judge Card
reporting protocol". It also already measures how often a judge flips its
answer when the prompt is reworded, and it already uses bootstrap intervals.
It is text only, about software-agent safety logs, with no video and no
anchor. **We must stop calling "a judge card" the new thing.** Rename ours,
cite theirs, and claim only the video-and-anchor part.

**Loud: running a video judge on real footage as a control is already
published.** [Physion-Eval](https://arxiv.org/abs/2603.19607) (2026-03-20)
shows untrained viewers a blinded 1:1 mixture of real and generated clips, then
runs ten automatic judges on the same mixture and scores each with Youden's J.
I confirmed this in the paper's own section 4.1: it benchmarks GPT-5.2, Gemini
3.0 Pro, Claude Opus 4.5, Qwen-3-VL and Cosmos Reason against human viewers.
We are **not** first to run a generated-video judge on real video.

> **Correction to `intact-b.md`.** That record says Physion-Eval "does **not**
> validate any automatic judge". That is wrong, and the sentence it quotes does
> not appear in the paper's abstract. Fix this before anyone quotes it.

**Loud: the complaint that these benchmarks aggregate instead of diagnosing is
already in print.** [RoboGaze](https://arxiv.org/abs/2606.28385) (2026-06-22)
writes: "existing frameworks rank generative models via aggregated scores
rather than diagnose individual videos". That is our complaint, published in
June. RoboGaze also already measures the "cry-wolf" failure, where a judge
invents a fault in a clean clip, and lifts clean-clip accuracy from under 25%
to over 80%. It builds a **new** judge and needs fresh human labels on 382
clips, so it does not audit anyone else's judge. But the observation is no
longer ours.

### The two named threats, deep-read, both survive as non-threats

| Paper | What it does | Where it stops short of us |
|---|---|---|
| [RoboTrustBench 2606.01600](https://arxiv.org/abs/2606.01600) (2026-06-01) | 1,207 real DROID scenes, 13 scoring rules, judge scores compared to human scores by rank correlation | Its 10,000-resample bootstrap says which **generator** ranks first, not how good the **judge** is. It never runs a judge against a recorded success label. The word "DreamGen" does not appear in it. |
| [Physion-Eval 2603.19607](https://arxiv.org/abs/2603.19607) (2026-03-20) | Ten judges scored on a real/generated mix by Youden's J | Its "right answer" on the real clips is an **assumption** (real footage obeys physics), not a **recorded** outcome. No confidence interval on any judge number, no d′-versus-criterion split, no blind arm. |

### The other checks

- **Nobody has audited DreamGen Bench.** All 144 papers citing
  [2505.12705](https://arxiv.org/abs/2505.12705) were pulled from Semantic
  Scholar and scanned for audit, validity, reliability, agreement, correlation,
  judge and calibration words. Nothing audits its judge. An exact-phrase search
  for "DreamGen Bench" returns only DreamGen itself and one paper that uses the
  benchmark.
- **VideoCon-Physics, the physics judge DreamGen leans on, was itself only
  weakly validated.** Its own paper,
  [VideoPhy](https://arxiv.org/abs/2406.03520), reports ROC-AUC 73 for physical
  commonsense, with no interval, on general text-to-video clips. DreamGen then
  uses it on multi-view robot video and admits in its Appendix H.2 that the
  model "has not been trained on multiview videos (RoboCasa)".
- **Adjacent but different:** [Physics-IQ
  Verified](https://arxiv.org/abs/2606.18943) audits a benchmark's prompts and
  scoring weights, not a judge. [RoboProcessBench
  2606.13040](https://arxiv.org/abs/2606.13040) tests whether VLMs understand
  robot processes; it is a capability benchmark, not a judge transfer.

### What is still ours, in one sentence

**An audit of one named, widely used, still-unchecked claim, settled with a
per-episode anchor whose labels were recorded by the robot itself rather than
assumed or freshly annotated, plus the first matched same-model prompt
comparison for a video judge.**

**Level 3.** Above our Level-2 bar, so it passes. Materially below the Level 4
the first pass assigned.

### What could not be checked

The NeurIPS workshop ["Who Verifies the
Agents?"](https://verify-agents-workshop.github.io/) notifies authors on
2026-09-29 and publishes nothing before then. A competing submission cannot be
ruled out. The search tool has no working OpenReview module, which is exactly
where such a paper would sit.

---

## 3. Live verification — every load-bearing fact checked today

### The DreamGen claim, read from the paper's own HTML

The paper says "an average Pearson correlation of > 90%, ensuring that the
model-based evaluation metric is aligned to human judgment".

**Each of those correlations is computed over four points, not sixteen.** Each
row of Tables 6 and 7 holds four fine-tuned models on one dataset. Four
datasets and two judges give eight separate correlations, and no single one
uses more than four points.

I computed the Fisher-z interval for each. **Seven of the eight intervals
include zero.**

| Table | Dataset | Judge | r | 95% interval |
|---|---|---|---|---|
| 6 | RoboCasa | GPT-4o | 0.94 | [−0.218, +0.999] |
| 6 | GR1-Object | GPT-4o | 0.93 | [−0.293, +0.999] |
| 6 | GR1-Behavior | GPT-4o | 0.96 | [−0.014, +0.999] |
| 6 | GR1-Env | GPT-4o | 1.00 | undefined (r is exactly 1) |
| 7 | RoboCasa | Qwen2.5-VL | 0.92 | [−0.355, +0.998] |
| 7 | GR1-Object | Qwen2.5-VL | 0.95 | [−0.128, +0.999] |
| 7 | GR1-Behavior | Qwen2.5-VL | 0.97 | **[+0.132, +0.999]** |
| 7 | GR1-Env | Qwen2.5-VL | 0.96 | [−0.014, +0.999] |

**The sharpest single fact in this gate.** Table 7, RoboCasa: the Qwen2.5-VL
judge scores the four models 8.3, 10.4, 18.8 and 29.2. The humans score the
same four models 81.3, 79.2, 91.7 and 93.8. The judge is about **70 points
below** the humans on every model, and the reported correlation is still 0.92.
Pearson r does not notice a constant offset. A judge that calls almost every
successful video a failure still earns r = 0.92.

Three more facts read from the paper:

- The paper writes "we ... calculate the AUC-ROC between them" in Appendix H.3,
  then reports no AUC-ROC anywhere.
- It never reports how many videos humans watched, how many annotators there
  were, or a single confidence interval.
- Appendix H.3 says humans evaluated "the 3 fine-tuned video world models",
  but Tables 6 and 7 show four columns. The counts do not match.

### The judge is public, cheap, and frozen

[NVIDIA/GR00T-Dreams](https://github.com/NVIDIA/GR00T-Dreams): **605 stars,
Apache-2.0, last pushed 2025-10-24.** The repo has not been touched in ten
months. It ships `dreamgenbench/eval_sr_qwen_whole.py`,
`dreamgenbench/eval_sr_gpt4o_whole.py`, `dreamgenbench/eval_qwen_pa.py` and
`dreamgenbench/utils.py`. I downloaded and read them.

Confirmed by reading the code:

- The backbone is
  [Qwen2.5-VL-7B-Instruct](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct),
  ungated. Frames are sampled with `num_frames=49, scale_factor=0.3` and passed
  as **still images**, not as video.
- **Two different prompts measure the same thing.** The default asks: "Does the
  video follow the instruction to finish the task ... Answer 0 for No or 1 for
  Yes." The `--zeroshot` prompt says: "If you see HUMAN HANDS instead of robot
  arms, IMMEDIATELY ANSWER 0 ... Be extremely strict in your judgment." One is
  neutral, one is harsh. They cannot both be the reported measurement.
- **The instruction the judge checks is parsed out of the video file name**,
  with four different string-splitting rules chosen by which generator made the
  file. A file that does not match its rule silently yields a wrong
  instruction, and nothing warns you.

### The videos and human scores are NOT released

The repo's own instructions point at an internal NVIDIA path,
`/mnt/amlfs-01/home/joelj/human_evals/`. I searched the Hugging Face dataset
API for `dreamgen`, `GR00T-Dreams` and `dreamgenbench`, and checked the
`nvidia` organisation. **There is no DreamGen Bench dataset.** The only public
matches are unrelated language models and community-generated robot data.

**Consequence, and it is a real limit:** we cannot re-score DreamGen's own
videos and we cannot recompute their correlation from raw scores. We can only
(a) recompute intervals from their published numbers, which is done above, and
(b) test their judge on footage we own.

### The repo already admits the weakness, in the authors' own words

From the README, section 5:

> "Our benchmark hopes to be friendly enough to the research community, thus
> only choosing ~50 videos for each dataset and using a relatively small open
> source VLM for major evaluation. Thus, our evaluation protocol might not be
> generalized well to some OOD scenarios like multi-view videos, judging
> physics in a detailed manner, etc."

**Read this twice. It is the single biggest risk to the whole direction.** If
we run their judge on our real DROID footage and it scores badly, NVIDIA can
answer "we told you it does not generalise". A "gotcha" framing dies on that
sentence. The gate's design below exists to survive it.

The paper's own eval sets are tiny, which the README confirms: RoboCasa 48,
GR1-Object 50, GR1-Behavior 47, GR1-Env 30.

### Our anchor works, and it is better than the audit recorded

The frozen index at
`robojudge/runs/judge/2026-08-08/index/episodes.jsonl` holds **5,106 episodes
over 7 policies**, every one carrying a recorded `binary_success` label and an
instruction string.

| | |
|---|---|
| Episodes | 5,106 |
| Policies | 7 (554 to 898 episodes each) |
| Recorded successes | 496 |
| Recorded failures | 4,610 |
| Base rate of success | 9.7% |

> **Correction to `intact-b.md`.** It says seven policies have "1,068–1,431"
> episodes each. The frozen index says 554 to 898. Use the manifest numbers.

**The 9.7% base rate matters and changes the metric.** Plain accuracy is
useless here: a judge that says "failure" every time scores 90.3%. **Report d′
and the criterion, never bare accuracy.** The audit's proposed kill line
"per-episode accuracy ≥ 85%" would have been passed by a judge that answers
nothing but zero. That kill line is replaced below.

The 5,106 episodes give plenty of statistical power for d′. At a realistic hit
rate of 0.5 and false-alarm rate of 0.2, the 95% interval on d′ is about ±0.12
wide on each side. Even at a very harsh hit rate of 0.15 it is about ±0.14.
That is tight enough to separate a working judge from a blind one.

**The videos are on Anvil and reachable:** 28 GB at
`/anvil/projects/x-cis261253/datasets/roboarena/dump/`.

### The control that makes this direction work

**Our locked campaign already ran the exact same model checkpoint on the exact
same episodes.** The judge roster pins `qwen25vl_7b` to
`Qwen/Qwen2.5-VL-7B-Instruct`, revision `cc594898...`, with our own 1-to-5
rubric prompt `generic_progress_v1`, native video at 1 frame per second. Its
5,106 scores sit on disk at
`runs/judge/full-2026-08/qwen25vl_7b/video/shard-*/scores.jsonl`, and
`robojudge-feeds.md` already reports its measured **d′ = 0.840, interval
[0.749, 0.931]**.

So the comparison is fully matched. **Same model, same weights, same episodes,
same recorded labels. Only the prompt and the frame handling change.** The
footage cannot be blamed for a difference, because the footage is identical.
This is what defeats the "out of distribution" defence quoted above, and no
published paper has it. It is the strongest single asset in this gate, and it
costs zero extra compute because half of it is already paid for.

---

## 4. The week-1 decisive step, written exactly

**One sentence:** run DreamGen's own instruction-following judge, unchanged,
on our 5,106 real RoboArena episodes with recorded success, under both released
prompts, and compare it against the same model already run under our prompt.

### The arms

| Arm | What runs | What it isolates |
|---|---|---|
| **A** | `dreamgenbench/eval_sr_qwen_whole.py` unchanged, default prompt, 49 frames at scale 0.3 | The judge exactly as shipped |
| **B** | The same script, `--zeroshot true` (the harsh prompt) | How much the answer depends on which prompt was chosen |
| **C** | DreamGen's default prompt text, run through **our** native-video pipeline at 1 frame per second | Separates the prompt from the frame sampling |
| **D** | Arm A with every frame replaced by one fixed unrelated frame | How much of the score comes from the instruction text alone |
| **E** | **Already on disk, zero cost.** Our `generic_progress_v1` prompt, same checkpoint, same episodes, d′ 0.840 [0.749, 0.931] | The matched baseline |

Arms A to D are four new passes over 5,106 episodes. Feed every arm the
instruction from our frozen index, **not** from the file name, and record that
as a deliberate change. Their file-name parser would produce nonsense on
RoboArena file names, and using it would test their string handling instead of
their judge.

### What gets computed

For every arm: d′, criterion, tie rate, and the full ROC area, each with a
**paired bootstrap interval clustered on session**, the same unit the locked
campaign uses. For each pair of arms: the paired difference in d′ with its own
interval. For arms A and B: the share of episodes where the two prompts give
different answers.

Also computed, at zero cost and already done in section 3: the Fisher-z
interval on each of DreamGen's eight published correlations.

### The kill numbers, written before the run

**The direction dies as a direction if all three hold at once:**

1. Arm A reaches **d′ ≥ 0.60** and its interval's lower end stays **above
   0.40**, and
2. Arms A and B agree on **≥ 90%** of episodes, and
3. Arm D, the blind arm, has a d′ interval that **includes zero**.

That combination means the shipped judge really discriminates, is not at the
mercy of the prompt, and is genuinely looking at the video. The ">90%
agreement" claim would then need no correction beyond the interval note, and
what is left is one paragraph inside the RoboJudge paper, not a section.

**The direction narrows, but survives, if condition 1 holds and either 2 or 3
fails.** The judge works, but its reported number is fragile. We then ship the
harness and the interval-reporting card, and drop every word implying the judge
is broken.

**The direction is strong if arm A's d′ interval's upper end sits below 0.40,
or arms A and B disagree on more than 25% of episodes.** The paired difference
against arm E, on identical footage, then rules out the out-of-distribution
defence, and the audit finding stands on its own.

**One extra guard against fooling ourselves.** If arm C lands far from arm A,
the story is about frame sampling, not the prompt. Say so plainly and rewrite
the claim. Do not report a prompt effect that is a frame-count effect.

### Cost

| Item | Cost |
|---|---|
| Fisher-z recomputation | 0 GPU-h, done, in section 3 |
| Arms A to D, four passes over 5,106 episodes | **10–20 L40S-hours** |
| Arm E | 0, already on disk |
| Full battery, adding VideoCon-Physics and the GPT-4o judge | 20–40 GPU-h plus a small OpenAI bill |
| Wall-clock for the decisive step | about 1 day |

For scale, our whole six-judge locked campaign cost 23.7 L40S-hours. This is a
fraction of one campaign. OrangeGrid is free with no time limit. No robots, no
new model, no new human labels.

---

## 5. Baselines, venue, risks, and what it unlocks

### Systems we must compare against

Every one of these already exists in our own runs, so the comparison table
costs nothing extra:

- **Arm E**, the same checkpoint under our prompt (d′ 0.840) — the matched
  control, and the most important row.
- **RoboReward-4B**, our best judge on these episodes (d′ 1.360 [1.236,
  1.490]) — the practical ceiling.
- **SigLIP2-B/16** (d′ −0.111 [−0.221, −0.008]) — the floor, a judge that
  points backwards.
- **The blind arm and the duration-only floor** from the locked campaign.
- **Physion-Eval's Youden's J numbers** — the closest published way of scoring
  a video judge, and the thing our card must beat on information content.
- **RoboGaze's clean-clip accuracy** — the published version of "does the judge
  cry wolf".

### Venue

**Primary: a section of the RoboJudge ICLR 2027 submission.** DreamGen Bench is
one of that project's four named audit targets, so this is already in scope.
The abstract deadline is **2026-09-18**, which is 17 days away. The decisive
step takes one day of compute, so the timing works.

**The NeurIPS workshop is not available to us.** Submissions to "Who Verifies
the Agents?" closed on **2026-08-29**, three days ago. It is a competitor
clock, not a venue. Treat any assumption that we can ride that wave as wrong.

**Do not plan this as a standalone paper.** After the three narrowings in
section 2, a standalone version is a workshop paper at best.

### Risks, loudest first

**The out-of-distribution defence is pre-written by the authors.** Their README
already says the protocol "might not be generalized well to some OOD scenarios
like multi-view videos". Only the matched arm-E comparison answers it. If we
lose that comparison, the direction loses most of its force.

**The finding may be too small for a section.** The Fisher-z table above is the
headline, and it took no GPU time. If arms A to D come back showing a
reasonable judge, everything left fits in one paragraph.

**The NVIDIA clock.** GR00T-Dreams has not been touched since 2025-10-24, but
NVIDIA shipped [GR00T N1.7](https://huggingface.co/nvidia/GR00T-N1.7-3B)
publicly. A refreshed DreamGen Bench with proper validation would end this
direction overnight. Re-check the repo before the ICLR abstract.

**Naming collision.** [2605.06161](https://arxiv.org/abs/2605.06161) owns
"Judge Card". Pick a different name now, before it appears in a draft.

**A crowded four months.** RoboGaze, RoboTrustBench, RoboProcessBench,
Physion-Eval and Physics-IQ Verified all landed between March and June 2026.
The 48-hour re-gate rule applies to anything adjacent that appears next.

**Two house precedents point the wrong way, and both must be answered.**
[[Unified-Direction-Ranking-2026-08]] Part 3 records that the owner **vetoed**
"seed-noise and variance of agent leaderboards" for three reasons: not
significant, no barrier, and not novel. It also records that "V2A metric
validation" was killed outright. Our direction is a cousin of both: it puts
intervals on someone else's evaluation number. **The answer is that the
deliverable here is a running harness plus a matched same-model contrast, not a
variance complaint, and the anchor removes the labelling barrier that made the
earlier items cheap talk.** If the week-1 arms come back showing the judge
works, that answer stops being true, and the veto reasoning applies to this
direction too. Say so at that point rather than defending it.

**Nothing here is destructive or irreversible.** The decisive step only reads
videos we already hold and writes new score files.

### What it unlocks

- **It closes the specification gap flagged in `robojudge-feeds.md`.** That
  record warns that the locked prereg's promised method, a per-episode
  rescoring, cannot move Kendall τ at all if it never reverses two scores. The
  video-judge card gives the RoboJudge paper a second, independent mechanism
  that does not depend on τ moving.
- **It turns one audit target into a reusable tool.** Any future video judge
  can be run through the same card, on the same anchor, for a few GPU-hours.
- **It makes the anchor an asset.** 5,106 real episodes with recorded outcomes
  is a checking set that costs nothing to reuse, in a field where every
  competitor pays for fresh human labels: 382 clips for RoboGaze, 1,207 pairs
  for RoboTrustBench, 10,990 traces for Physion-Eval.

---

## 6. Rating and conditions

**★★★, conditional. Proceed as a section of the RoboJudge submission.**

Down from the audit's ★★★★. The three reasons are all in section 2: the card
name and concept are published, running a judge on real video is published, and
the aggregation complaint is published.

It stays above three stars because the target claim is genuinely unchecked
across all 144 citing papers, the anchor is free, the matched same-model
control is unique to us, and the headline number is already computed.

**Four conditions, all before the 2026-09-18 abstract:**

1. **Rename it.** "Judge card" belongs to
   [2605.06161](https://arxiv.org/abs/2605.06161). Cite that paper as the text
   version of the same idea.
2. **Fix the two errors in `intact-b.md`** before either is quoted: Physion-Eval
   does validate automatic judges, and the frozen index has 554 to 898 episodes
   per policy, not 1,068 to 1,431.
3. **Replace the accuracy kill line.** At a 9.7% success base rate, "accuracy
   ≥ 85%" is passed by a judge that always answers zero. Use the d′ numbers in
   section 4.
4. **Run arm E's comparison, or do not run this at all.** Without the matched
   same-model contrast, the authors' own out-of-distribution sentence answers
   every finding we could produce.

**Re-gate trigger:** any new paper that audits a generated-video judge against
recorded outcomes, or any refresh of GR00T-Dreams. The workshop notifies on
2026-09-29; re-check the accepted list that day.

---

## Related

[[Direction-Audit-2026-09-01]] · [[Prereg-RoboJudge-Audit]] ·
[[Method-Gates-2026-08]] · [[Unified-Direction-Ranking-2026-08]]

# Live Research Opportunities

Status: **Written 2026-07-26 and partly replaced by later work.** *Updated
2026-08-02 for the general research wiki.* This page followed a correction to
the way we choose ideas. A third survey, which transfers mature theory into a
new area, was later added as Cluster C.

> **Supersession note.** [[Unified-Direction-Ranking-2026-08]] is the current
> source for direction ratings. [[Direction-Reevaluation-2026-08]] is the
> historical August 2 check. That check changed this page in three ways:
>
> - B1 asks whether training data or the training method causes model answers
>   to lose variety. It fell to ★★★ because an April 2026 paper already
>   answered much of the question.
> - Automated heuristic design (AHD), which uses systems such as LLMs to invent
>   problem-solving rules, rose to ★★★★. The important part found here—compare
>   LLM and classical designers at the same cost—was still open at that time.
> - The broad claim that positive method papers are safer than audits was
>   **reversed**.
>
> Dated notes below explain each change. A1–A3 are the three world-model
> evaluation ideas. B2 studies visual attention sinks, B3 studies controls for
> reinforcement learning with verifiable rewards, and Cluster C moves mature
> statistics into LLM research. These sections were not checked again and
> remain as written. [[Top-Researcher-Scan-2026-08]] also turns A1's judge-check
> method into a program across many targets, called convergence V.

## We were using the wrong standard

We stopped 13 ideas because no one had to have worked near them before. The
notes called this **virgin territory**. That has never been the normal standard
for publication. Almost no accepted paper enters a completely untouched area.
This page uses the better question:

> **Is there a clear improvement, extension, or new question that a capable
> reviewer would want to read?**

Nearby papers are normal. They show that the problem matters. The ideas below
use three valid paper shapes: **improve earlier work**, **ask and answer a new
question**, or **move a mature method into a new domain**.

## We also failed to check two highly ranked directions

[[Field-Scouting-Survey]] ranked world-model evaluation first. It was never
checked because the discussion moved to math. That was a separate process
mistake. This page finally checks it, and the direction survives.

---

# Cluster A — construct validity: do world-model tests measure what they claim?

This is called *construct validity*: whether a test truly measures the idea it
claims to measure.

**Checked research area.** There are 759 arXiv CS papers matching
`"world model"` + `benchmark`, and we listed **more than 50 named benchmarks**. A
search for `"video generation"` plus `"meta-evaluation"` returns only **one**
paper, a hallucination detector rather than a study of evaluation quality. **No
video paper compares different benchmarks to see whether they measure the same
thing.** Text research already has the needed tools:
BenchBench ([`2407.13696`](https://arxiv.org/abs/2407.13696)),
Benchmark-squared ([`2601.03986`](https://arxiv.org/abs/2601.03986)), and
*When Benchmarks are Targets* ([`2402.01781`](https://arxiv.org/abs/2402.01781)).

**Three papers say the measurement is broken, but none tests it at scale.**
[`2601.15533`](https://arxiv.org/abs/2601.15533) names visual conflation, and
[`2606.15032`](https://arxiv.org/abs/2606.15032) gives an L0-L7 hierarchy based
on decisions. Both are position papers with no experiments.
[`2607.07196`](https://arxiv.org/abs/2607.07196) adds only an **`n=2` example**:
the model with higher visual quality has worse action-following.

## A1 — "Your judge was never validated" (start here; under 100 GPU-hours)

> **Update 2026-08-02 — rating unchanged, scope enlarged.**
> [[Top-Researcher-Scan-2026-08]] found the same gap at eight separate targets
> and calls it convergence **V**: verification is the main limit, but the
> verifiers themselves have not been checked. Karpathy, Finn, Neubig, Wei,
> D. Zhou, Sutskever, and Bengio all point to it. At least five released judge
> or evaluator systems have **zero independent validity audits**: world-model
> evaluators, [SC3-Eval](https://arxiv.org/abs/2606.18610), the VLM judge in
> [DreamGen Bench](https://arxiv.org/abs/2505.12705), DriveJudge, and VLM reward
> judges. The groups that built them have a built-in reason not to show their
> own metric fails. Use **one registered method across many targets**: test
> hidden factors, easy ways to game the judge, behavior outside the training
> distribution (OOD), and whether changing the judge changes the ranking. This
> is a program, not one paper, and a new top candidate beside T1 and T3. Two
> rules come with it. Release a reusable tool because diagnosis earns trust but
> tools earn citations. Also name, when registering the audit, the **method
> paper** that the diagnosis will make possible.

**A finding that can be checked directly in the paper.**
[VBench-2.0](https://arxiv.org/abs/2503.21755) supports its human-alignment claim
with a **model-level correlation across only four models**: Sora, Kling 1.6,
[HunyuanVideo](https://arxiv.org/abs/2412.03603), and
[CogVideoX-1.5](https://arxiv.org/abs/2408.06072). With `n=4`, a Spearman rank
correlation can only be `{0, ±0.4, ±0.8, ±1.0}`. Even `rho = 1.0` has
`p = 0.083`, so it is **not significant at α=0.05**. Under the natural reading,
not one part of the leading video benchmark proves a statistically meaningful
match to human ratings. The paper's reported values—99.31%, 95.46%, and
81.70%—are **not even in that allowed set**. The paper therefore does not fully
explain how it combined the data, and its result cannot be reproduced from the
description. Either reading gives a real finding.

Other papers show the same pattern. [WorldJen](https://arxiv.org/abs/2605.03475)
reports `rho = 1.000`, which is a warning sign for tiny `n`, not proof of a good
judge. [MSVBench](https://arxiv.org/abs/2602.23969) reports 94.4%,
[MSAVBench](https://arxiv.org/abs/2605.20183) 91.5%, and
[V2V-Bench](https://arxiv.org/abs/2606.05665) 0.905. All use model-level totals
from small groups of models. **[WorldModelBench](https://arxiv.org/abs/2502.20694)
calls its judge "human-aligned" without reporting any agreement number.**
[EWMBench](https://arxiv.org/abs/2505.09694) and
[WorldScore](https://arxiv.org/abs/2504.00983) report none either.

Direct studies say VLM judges are weak at exactly this job. TRAVL
([`2510.07550`](https://arxiv.org/abs/2510.07550)) finds that they need training
on full motion paths to identify impossible physics. Physion-Eval
([`2603.19607`](https://arxiv.org/abs/2603.19607)) finds that expert humans
notice physics errors in **83.3–93.5%** of generated videos where VLMs miss them.

**Why the study is cheap.** [VideoPhy-2](https://arxiv.org/abs/2503.06800)
released **about 102,000 human labels**, automatic scores for each example, and
an **open-weight judge**. A researcher can test the judge without generating
one new video.

**Three steps:**

1. Recalculate agreement from individual examples, not model totals, and show
   how it differs from the published model-level correlations.
2. Run a power analysis. For each published alignment claim, show how large
   `n` must be to reach significance and plot the real uncertainty interval.
3. Keep videos and prompts fixed while changing the judge among
   [Qwen2.5-VL](https://arxiv.org/abs/2502.13923), InternVL, Cosmos-Reason2,
   VideoCon-Physics, and a closed API. Measure how much the leaderboard moves.
   For the closed API, score the same videos again months later to measure
   silent model changes.

Video can reuse methods from machine translation and summarization:
*Guardians of the MT Meta-Evaluation*
([`2408.13831`](https://arxiv.org/abs/2408.13831)), the difference between
system-level and example-level correlation, and *The Autocorrelation Blind
Spot* ([`2604.14414`](https://arxiv.org/abs/2604.14414)), which found that 42%
of turn-level findings disappear after correcting for related observations in
the same group.

**Cost: 1–2 GPUs for a few days, under 100 GPU-h.**

## A2 — do world-model benchmarks agree with one another?

Run 6–8 public benchmarks—
[VBench-2.0](https://arxiv.org/abs/2503.21755),
[WorldScore](https://arxiv.org/abs/2504.00983),
[VideoPhy-2](https://arxiv.org/abs/2503.06800),
[WorldModelBench](https://arxiv.org/abs/2502.20694),
[EWMBench](https://arxiv.org/abs/2505.09694),
[PhyWorldBench](https://arxiv.org/abs/2507.13428), and
[Physics-IQ-Verified](https://arxiv.org/abs/2606.18943)—on **one shared group of
10–15 open models under one fixed generation process**. No leaderboard holds
resolution, video length, random seed, and prompt expansion steady this way.
Report rank correlations for each pair with bootstrap uncertainty intervals.
Separate variation caused by the model, benchmark, prompt, and seed. Calculate
how many models a leaderboard needs before it can detect a real ranking change.

**A useful comparison:** Physics-IQ Verified rechecked *one* benchmark and the
new ranking agreed with the old one at only **Kendall tau = 0.46**. If separate
benchmarks agree around `tau ~ 0.5`, the combined field ranking is mostly noise.

**Cost: 1–3 weeks on 8 GPUs.** Most cost comes from video generation.

**Closest competing work, which can still be improved.** WorldArena
([`2602.08971`](https://arxiv.org/abs/2602.08971)) compares **its own new combined
score** with later usefulness for `n=14` models and **two** robot-manipulation
tasks. It reports `r = 0.360` for action planning without an uncertainty
interval. At `n=14`, `r=0.36` has an approximate 95% interval of
`[-0.21, 0.75]`, so the data cannot tell zero agreement from 0.8 agreement.
The risk is that this well-funded team released version 2.0 only three months
after version 1.0.

## A3 — how much of a closed-loop score comes from the action extractor?

KineBench ([`2607.19876`](https://arxiv.org/abs/2607.19876)) says
inverse-dynamics models (IDMs), which infer actions from state changes, are
"brittle to data outside their training distribution." This makes it unclear
whether a bad score comes from the world model or from the action extractor.
KineBench avoids the issue with IDM-free 6D-pose extraction, but **never
measures how much each part causes the error.** On ManiSkill3 and LIBERO,
compare the same models using IDM extraction, IDM-free extraction, and the real
actions. Show how the extractor's share of the error grows as data moves
farther OOD. The output is a protocol that corrects for extractor error.

## Useful infrastructure

`stable-worldmodel` from `rbalestr-lab` has 2.1k stars, installs with pip, and
supports more than 30 environments plus planning baselines. It is the most
useful single tool here. Open action-conditioned models include
**Cosmos3-Edge** (4B, OpenMDW-1.1, no access gate, 23.9 seconds for one
480p/189-frame clip on one H100),
**[Matrix-Game-2.0](https://arxiv.org/abs/2508.13009)** (1.8B, MIT license,
25 FPS streaming), and
**[V-JEPA 2-AC](https://arxiv.org/abs/2506.09985)** (MIT license). Avoid Wan 2.5,
which is API-only; [Open-Sora-Plan v1.5](https://arxiv.org/abs/2412.00131),
which is NPU-only; and Cosmos-Predict2.5, which is old, gated, and takes 870
seconds per video.

---

# Cluster B — open problems named by the paper authors

This search read **1,046 full papers** and found 1,423 clear open-problem
statements across 656 papers. One method lesson matters: "we lacked compute" is
usually generic and not useful. Better leads name **one experiment the authors
could not run** or **one effect they saw but could not explain**.

## B1 — does post-training data or the method cause diversity collapse?

> **DOWNGRADED 2026-08-02: ★★★★ → ★★★.**
> [[Direction-Reevaluation-2026-08]] found that the paper described below as
> the *opening* also **closes** the opening.
> [`2604.16027`](https://arxiv.org/abs/2604.16027) by Karouzos, Tan, and Aletras,
> posted 2026-04-17, already states the main result: training data writes the
> collapse into model weights; output format does not cause it. The authors
> also named the direct crossover experiment as their own next step, so the
> team best able to run it has publicly claimed it. Apple/CMU
> ([`2605.09995`](https://arxiv.org/abs/2605.09995)) adds the untested claim that
> collapse "worsens with scale." The cost below was too low: a complete
> SFT/DPO/RLVR crossover across model sizes costs **more than 400 GPU-hours**,
> not a weekend with 1–7B models on eight GPUs. The broad area still has many
> openings—about 10 papers per month, no survey, and at least seven separate
> reports in [[Top-Researcher-Scan-2026-08]]. Measurement is established, but
> the cause is open at every level. Still, this **exact** question is closed.
> New work would have to be a contested, careful test between claims.

*Where does output diversity collapse in post-training?*
([`2604.16027`](https://arxiv.org/abs/2604.16027), April 2026) says:
*"No existing study isolates the role of the training method from the training
data, or the generation format from the model weights."* Its own evidence only
observes released [Olmo 3](https://arxiv.org/abs/2512.13961) checkpoints. It did
not directly change one cause at a time.

The original plan looked easy because Olmo 3 releases model weights **and the
post-training data for every stage**. This lets us directly change the hidden
factor. The design was a 2×2 comparison of {narrow data from two teachers vs
broad data from many sources} × {SFT vs DPO}, with the same number of training
tokens and at least three seeds. It would report different reasoning paths
separately from different final answers. The plan called for 1–7B models on
eight GPUs and judged nearby competition low to medium.

## B2 — when and why do visual attention sinks appear during training?

*See What You Are Told*
([`2503.03321`](https://arxiv.org/abs/2503.03321), ICLR 2025, 139 citations)
says that "how visual attention sinks emerge during the training process
remain open questions." About 20 follow-ups use the effect, but **none of the
100 citing papers studies when it appears during training.**

The key test is cheap: during visual instruction tuning, compare a frozen LLM
backbone with an unfrozen one. If sinks appear while the backbone is frozen,
the text model already contained them. If they do not, visual training creates
them. **Either answer can publish.** Use a 7B model on eight GPUs.

## B3 — RLVR controls skipped because they needed more training runs

*Hidden Costs and Measurement Gaps of RLVR*
([`2509.21882`](https://arxiv.org/abs/2509.21882)) says the authors left out
tests of each reward part "because they require additional training runs."
Choose three widely reported gains from reinforcement learning with verifiable
rewards (RLVR). Repeat each with and without a reward for abstaining or being
well calibrated. Keep the training budget equal and use several seeds. Then
separate a sharper answer distribution from a true increase in ability. Use
GRPO at 1.5–7B on 8×H100. Many groups study RLVR methods, but few study these
measurement controls.

## Other named openings

- Speculative decoding with draft models that have large vocabularies
  ([`2502.05202`](https://arxiv.org/abs/2502.05202), ICML 2025). The authors
  clearly name it as future work and release a benchmark harness.
- Compare in-domain and open-domain data for training sparse autoencoders (SAEs)
  ([`2501.06254`](https://arxiv.org/abs/2501.06254), ICLR 2025). Appendix B
  already answers part of it.
- Make test-harness metrics that separate the agent from the surrounding
  scaffold. [`2605.18747`](https://arxiv.org/abs/2605.18747) has a section
  literally called "Open Problems."
- Build long-horizon metrics that allow for self-correction
  ([`2509.09677`](https://arxiv.org/abs/2509.09677)).

---

# Cluster C — move mature statistics into LLM practice

This survey corrected itself twice. First, **queueing theory for LLM serving is
not open.** A real operations-research (OR) literature formed in 2025–26. Dai's
fluid-limit proofs show which systems maximize throughput and find that normal
vLLM is *not* maximally stable. Foster-Lyapunov work proves stability with KV
cache limits and matches real-GPU behavior within 10%. Ye, Jaillet, and
Simchi-Levi also study worst-case competitive performance.

Second, **designing prompt and scaffold experiments is not empty.** Four nearby
papers appeared in ten weeks. One runs a full `2^5` experiment over all
combinations of scaffold parts and computes exact Shapley values
([`2605.05716`](https://arxiv.org/abs/2605.05716)). **CAFE**
([`2607.10380`](https://arxiv.org/abs/2607.10380), **11 July 2026**) releases a
package for experiments over combinations plus mixed-effects models that
assign variation to parts of a compound AI pipeline.

## C1 — use the right error terms for LLM ablations

**Main claim.** Most LLM ablations have the structure of a **split-plot
experiment**, but papers analyze them as if every run were independently and
fully randomized. A split-plot experiment has expensive choices that cannot
change often, called whole-plot factors, and cheap choices that can change
inside each whole plot, called sub-plot factors. Model, checkpoint, and building
a retrieval index are hard-to-change whole-plot choices. Prompt wording,
temperature, and number of examples in the prompt are easy sub-plot choices.
If a split-plot is analyzed as a completely randomized design (CRD), it mixes
two kinds of error. The smaller sub-plot error then dominates. **The test uses
too little error for whole-plot effects and makes them look more certain than
they are.** Those model-level effects are exactly what LLM papers often claim.

**A statistics reviewer will reject the work unless it separates two cases:**

- **Pseudoreplication:** a paper says "our method improves 2% across 3 models,
  `p < 0.001`," but calculates the p-value as if every run were independent.
  The real effective sample size for the model-level claim is 3. The correct
  error for the method is the method × model interaction.
- **A true split-plot design:** whole plots are actually repeated and the data
  has two separate sources of variation.

The analysis also depends on whether models are **fixed**—we care only about
these exact models—or **random**—they are treated as a sample from a larger
population. The design section must state which case it uses.

**Why this avoids the motivation problem that paused
[[KD-Noise-Floor-Stage1]].** The paper does not merely ask people to show more
error bars. It releases a **correct analysis and a practical replacement**
using `lme4` and standard split-plot methods. Researchers have something clear
to do differently.

**The main risk is showing a clear difference from CAFE.** CAFE was only two
weeks old. It uses mixed-effects models to explain **where variation comes
from**, not to choose the correct **error term for a whole-plot claim**. That is
a real but narrow difference. The introduction must explain it in one sentence,
and a reviewer will question it. Paper
[`2605.05716`](https://arxiv.org/abs/2605.05716) reports **183/325 violations of
submodularity**. This is evidence that changing one factor at a time, the
field's normal method, fails. The paper itself does not make that point.

Bundle three nearby open pieces into the paper:

- Lenth's method and half-normal plots for experiments with no exact repeats.
  DBLP gives only **1 result in all of computer science**. LLM experiments are
  expensive and often have no repeats, exactly the setting for which the
  pseudo-standard-error was made.
- **Aliasing**, where one effect cannot be separated from another because of
  the experiment design. No LLM ablation paper discusses it.
- **Fractional designs when `k >= 10`.** The only full-combination study stops
  at `k=5`. Real scaffolds have 10–20 parts, exactly where testing only a chosen
  fraction of combinations becomes necessary.

Follow the shape of Tang, Lin & Sahni, **KDD 2024**
([`2311.14698`](https://arxiv.org/abs/2311.14698)). They used fractional
factorial designs—experiments that test chosen combinations—for DoorDash
policy and gained 5% more profit at 67% lower
cost. The argument can transfer to a new domain.

Background: Wu & Hamada, requiring 2–3 weeks. Compute: less than every other
idea in this survey.

## C2 — treat contamination as differential item functioning

Two agents reached this idea from different starting points, the strongest
independent support in the session. Differential item functioning (DIF) is a
psychometric method for finding questions that act differently for people with
the same ability. That exactly matches the benchmark problem: an item may be
easy for a contaminated model but not for an equally capable clean model.
**This is the best use of the GPUs.**

## C3 — correct agent hazard curves for frailty

This is the fastest idea and has almost no risk. In survival analysis, unseen
differences between subjects are called *frailty*. Because of frailty, the
failure rate for a whole population can move in the **opposite direction** from
the failure rate of every individual. The agent field has no language for this
standard result. The idea reportedly settles a live ICLR 2026 disagreement.

## C4 — predict tail latency when agents fan out and rejoin

Four database searches found this empty. `fork-join` plus LLM gives 3 results;
OpenAlex from 2022 onward gives zero for fork-join/straggler; OpenReview gives
12, none about LLMs. **TraceLab**
([`2606.30560`](https://arxiv.org/abs/2606.30560)), only three weeks old,
contains about 430k tool calls from about 4,300 coding-agent sessions. Tool-call
times have heavy tails. TraceLab directly recommends sending out more work in
parallel, but gives no model for how that changes the slowest finishing time.

The important twist is that classic fork-join limits assume independent jobs.
Sibling agent tasks share a prompt prefix, model, and backend queue, so their
times are probably **positively related**. This breaks the old limits in a
direction no one has measured. The theory already exists but LLM papers do not
cite it: Raaijmakers-Borst-Boxma, *Queueing Systems* 2022, studies fork-join
systems with repeated work under heavy-tailed times. LLM prefix-cache sharing
also makes an extra copy much cheaper than an extra copy in a normal data
center, so the best amount of repeated work changes.

**Do not enter these Cluster C areas:** serving stability and throughput, which
are owned; stopping each query adaptively, which is already crowded and
mathematically strong—[ConSol](https://arxiv.org/abs/2503.17587) is exactly a
Wald sequential probability ratio test (SPRT), while CITE uses e-processes and
has matching best-possible lower bounds; plain IRT; prediction-powered
inference (PPI) for LLM evaluation; and signal-detection theory for calibration.

---

# One program connects five ideas

Four separate searches using four different toolkits found the same problem:
**evaluation and ablation papers often use the wrong structure for their
errors, and that can reverse their conclusions.**

- **DIF:** compare items only after accounting for ability.
- **Split-plot:** keep error from whole plots separate from error inside them.
- **Frailty:** a population's failure rate can move opposite to every person's
  failure rate.
- **Effective sample size:** rows from related checkpoints are not independent
  data points.
- **Interference:** leaderboard entries can affect or resemble one another, so
  they are not independent either.

These are not five unrelated papers. Any two naturally refer to each other.
Together they form a long-term program about better measurement. It provides
new methods, not only criticism, which helps it last.

## Suggested order

**Start C1 and move within weeks.** Its math is cheapest, its compute is
smallest, and it makes a **correctness** claim: a reanalysis can show that
published conclusions fail. It also offers a replacement method instead of
only asking for error bars. The opening is short because CAFE was two weeks old.

**Run A1 at the same time.** It needs under 100 GPU-hours and almost no video
generation. The `n=4` Spearman problem can be checked today from a published
PDF. If A1 works, A2 and A3 naturally form one paper on the validity of
world-model evaluation.

**Run B1 when GPUs become free.** The first version called it a positive
explanation of an effect rather than an audit, so others would not need to
admit an error for the paper to matter.

**Treat B2 as a cheap, uncertain bet.** One frozen-versus-unfrozen test answers
it, and either answer can publish.

**Use C2 as the GPU-heavy follow-up.** It would be the natural second paper in
the error-structure program.

> **Order correction from 2026-08-02.** Remove B1 because earlier work already
> did it; see its own dated note. C1, A1, B2, and C2 remain. B2 partly overlaps
> T1 in [[Method-Opportunities]] because both change whether the model is frozen
> during visual instruction tuning. If T1 runs, the same checkpoints provide
> B2's key measurement almost for free. On 2026-08-02, the top list was T1, T3,
> and the judge/verifier audit program. That historical decision is in
> [[Direction-Reevaluation-2026-08]] and [[Top-Researcher-Scan-2026-08]]. Use
> [[Unified-Direction-Ranking-2026-08]] for current decisions.

## Related

[[Unified-Direction-Ranking-2026-08]] — current direction decisions.
[[Direction-Reevaluation-2026-08]] — historical August 2 ratings; it replaced
the B1 and AHD decisions on this page at that time.
[[Top-Researcher-Scan-2026-08]] — broader judge/verifier audit, plus portfolio
and working rules.
[[Field-Scouting-Survey]] — first ranked world-model evaluation but left it
unchecked.
[[Method-Opportunities]] — method ideas T1–T4 and KV-cache.
[[KD-Noise-Floor-Stage1]] — paused. Its motivation problem explains why B1's
positive framing once mattered and why C1 must release a real alternative
analysis instead of only asking people to do better.

---

# Checked and stopped: comparing weather-extreme claims

**SCOOPED 2026-07-26.** The apparent disagreement had already been
settled in a paper—**by one of the two papers in the proposal.**

Biegert, Allen, Alber & Lerch
([`2606.21170`](https://arxiv.org/abs/2606.21170), KIT/Lerch group; Gneiting is
*not* an author) cite Zhang et al., explain the different results, and give the
cause:

> *"a likely explanation for the discrepancy in our conclusions is that the
> twPCRPS measures the information content of the forecast when predicting
> extreme events, rather than directly assessing the accuracy of the
> deterministic forecasts themselves... due to the forecaster's dilemma, if we
> condition evaluation on an extreme event having occurred, then the AIWP models
> will perform poorly."*

They also separate the uneven ground truth used by Zhang et al.: ERA5 for AI
models but IFS analysis for HRES. They test 101 threshold quantiles, including
`q = 1`, specifically to make a comparison with Zhang et al.

**A theorem breaks the planned compare-all-combinations experiment.** Its main
question was which choice reverses the result. But one scoring-rule choice is
already known to be improper and the other proper. Checking only observations
where an extreme happened creates the forecaster's dilemma. Multiplying a
proper score by a weight that depends on the observed outcome creates an
improper score (Gneiting & Ranjan 2011). Zhang et al. admit this in a Methods
section named *"Forecaster's dilemma."* Our study would have repeated a known
theorem.

Earlier work was also busy and came before both target papers. Olivetti &
Messori (GMD 2024) used a weighted MSE. Loveday & Hertneky
([`2510.25045`](https://arxiv.org/abs/2510.25045)) independently used
threshold-weighted scores. Pasche et al. (AIES 2025) came from Engelke's own
group. **[ExtremeWeatherBench](https://github.com/brightbandtech/ExtremeWeatherBench)**
by McGovern et al., Brightband + NCAR + ECMWF, has 329 cases and 724 commits and
is still being prepared.

**The mistake repeated an earlier pattern.** We called this a live disagreement
without checking whether the later paper discussed the earlier one. This was
the third time in the session that we read a paper's opening story instead of
its method section.

**Two small ideas survive and one person could run them:**

1. Test twPCRPS on data not used to fit it at record-level thresholds.
   [EasyUQ](https://arxiv.org/abs/2212.08376) is fitted **on the same data** with
   `n = 702` at each grid point. At `q = 1`, only a few tail observations matter.
   No one has tested stability or conformal-IDR.
2. Make **ground truth a full experiment factor** with METAR station data from
   the public bucket. This avoids the circular use of ERA5, which is itself an
   IFS product and therefore favors IFS-family systems.

Both need a meteorology collaborator to become more than a workshop paper.

---

# Checked: LLM automatic heuristic design audit — first judged mostly closed

> **UPGRADED 2026-08-02: ★★ → ★★★★. The "mostly closed" title below was wrong.**
> [[Direction-Reevaluation-2026-08]] checked the area by remaining opportunity,
> not by the number of control papers. About 13 control papers in 24 months each
> close only one narrow case. For example,
> [`2605.15221`](https://arxiv.org/abs/2605.15221) studies circle packing only at
> `n=26` and admits it "lacks comparison to domain-specific classical
> optimization methods." At the same time, about 20 new use areas appeared.
> Abstract counts for "automated heuristic design" are 1 / 1 / 5 / **17** for
> 2023 / 2024 / 2025 / January–July 2026. There are about 10–15 new claims for
> each control study.
>
> This section had already found the important survivor: **no one has drawn the
> cost-adjusted LLM-versus-classical performance line**. Three searches found
> no paper. Only [LLaMEA](https://arxiv.org/abs/2405.20132), which proposed the
> method, compared with CMA-ES/DE. Two papers can come from this area:
> **(a)** a cost-crossover curve that puts tokens and CPU in one money unit and
> asks whether problem-landscape roughness predicts the winning tool, using
> CostAda ([`2607.26828`](https://arxiv.org/abs/2607.26828)) for costs and
> [BLADE](https://arxiv.org/abs/2504.20183) as the harness; and **(b)** a
> **novelty audit** that grades rediscovery, recombination, and truly new ideas
> against the 19× worse OOD result from
> [RAISE](https://arxiv.org/abs/2606.31801). **No paper studies (b).** DeepMind
> [`2602.16928`](https://arxiv.org/abs/2602.16928) gives the reason: after
> distillation, "the true driver of generalization lies in a minimal
> algorithmic core." Before starting, remember that van Stein/Bäck and
> DeepMind work nearby, and check GECCO/PPSN because this gate searched only
> arXiv.

**Three control papers appeared from February through July 2026. One was only
six days old.**

- **Gideoni, Risi & Gal**
  ([`2602.16805`](https://arxiv.org/abs/2602.16805), February 2026), *Simple
  Baselines are Competitive with Code Evolution*, test two simple alternatives
  with "no explicit fitness-based selection." They match API dollars, number
  of function calls, **and** wall-clock time. Repeated conditioned sampling
  "matches or exceeds [AlphaEvolve](https://arxiv.org/abs/2506.13131) on 4/9
  problems."
- **Gupta et al.**
  ([`2607.18235`](https://arxiv.org/abs/2607.18235), Berkeley/MIT, **posted six
  days earlier**) run "more than 3.1 million LLM rollouts across 30 harnesses."
  Every comparison uses the same rollout count. They compare with a repeated
  Sequential Best-of-N (BoN) baseline using bootstrap tests and a Holm correction
  for many tests. No harness beats the simple baseline. "More complex
  OpenEvolve-style" settings perform worst.
- **Quan, Sun & López-Ibáñez**
  ([`2509.02297`](https://arxiv.org/abs/2509.02297)) work on 3D packing from
  **inside the AHD line**. They run a real null: replace the score with one
  constant, making selection fully random. [EoH](https://arxiv.org/abs/2401.02051)
  scores `777.3 ± 4.2`; S-GRASP scores `762`, ZHU `691`, and the exact method
  `689`. The LLM heuristic beats First-Fit but loses to every other method.

**What those papers settled.** Variation across seeds and best-of-N inflation
are **already done** at a scale and quality we cannot match. Three papers now
show that selection adds little. Only a strict **non-LLM proposer** comparison
has not run, and that is one section rather than a paper. Transfer tests were
also done twice.

**What remained and carried the direction.** No study uses classical automatic
algorithm configuration—irace, SMAC, or GP-hyper-heuristics—as a **competing
designer at the same wall-clock time and dollar cost**. Gideoni compares only
with LLM-based simple versions. Quan compares with human-made heuristics but
does not match the cost of discovery. RAISE dismisses classical tuners only by
saying "they cannot design new algorithmic logic."

None of the three new papers tests the normal EoH/ReEvo benchmark set of
TSP/BPP/CVRP with a classical tuner. Frame the paper as **"which tool gives the
best result per dollar,"** not as an attack; other papers already made the
attack. The estimated opening was **3–6 months**.

**One separate open question:** evaluation hacking in AHD was measured
**exactly once**. Vesper
([`2605.15221`](https://arxiv.org/abs/2605.15221)) reports an 8.2% hacking rate
from **two runs**, with no non-LLM control and no list of hacking types. It also
makes the striking, never-repeated claim that **stronger models hack more**.
This study would be cheap and uncontested.

## The first broad lesson, and why it was later reversed

The first reading was: three strong control papers appeared in five months,
including one within a week. **The field corrects itself faster than an audit
can be written and reviewed.** That seemed to support work that explains a
cause or supplies a method over work that only criticizes. The first re-ranking
therefore put **B1, diversity collapse, and B2, attention sinks, above A1, the
judge audit**, even though A1 was cheaper and had passed its gate. The claim was
that positive contributions remain open longer.

At that point, the honest status was: A1 and the weather, AHD, and compression
ideas were GATED. C1, B1, B2, and the two remaining AHD ideas were **NOT
GATED**.
The warning was not to treat unchecked ideas as solid.

> **REVERSED 2026-08-02 — the broad claim is false.** Across eight directions,
> [[Direction-Reevaluation-2026-08]] found the opposite in five: **new-method
> areas fill within months, while careful comparisons and diagnosis often stay
> open.** A likely reason is that method groups cannot afford to publish null
> results about their own explanations. Work designed to remain useful when
> the result is null is therefore exactly what stays unclaimed. The first
> ranking also failed its own test: B1, the positive contribution it raised,
> was already done. AHD, which it thought was closing, was adding new claims
> about ten times faster than controls.
>
> One narrower point was still correct. The **specific** audits listed here—
> seed variation, best-of-N inflation, and transfer testing—were taken quickly,
> and a paper framed as an attack becomes old fast. Use these replacement rules:
>
> - Ask **"which tool gives the best result per dollar?"** or carefully decide
>   between competing explanations. Do not frame the paper as an attack.
> - Release a reusable harness, benchmark, or metric with every such study.
>   Diagnosis earns trust; reusable artifacts earn citations. The measured
>   rates in [[Top-Researcher-Scan-2026-08]] are 0.3–0.8 versus 9–12 citations
>   per month.
> - When registering the plan, name the method paper that the diagnosis will
>   make possible.
> - Judge an area by whether new questions appear faster than papers answer
>   them and whether this **exact** question is claimed. Do not judge by crowd
>   size or by whether the paper is an audit or a method.
>
> The warning about unchecked ideas still stands. It gains one more rule:
> **repeat any gate older than about six weeks before spending compute.**

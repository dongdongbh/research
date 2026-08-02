# Live Research Opportunities

Status: **Active 2026-07-26.** Produced after recalibrating the selection bar.
A third survey (mature-theory transfer) is still running and will be appended.

## The bar was wrong, and that was the session's real failure

Thirteen candidates were killed for not being **virgin territory**. That is not
the publication standard and never was — almost no accepted paper clears it. The
correct bar, and the one used here:

> **Is there a specific improvement, extension, or new question that a competent
> reviewer would find worth reading?**

Adjacent work existing is normal. It is evidence the problem matters.

Three legitimate modes, all used below: **improve existing work**, **pose and
answer a new question**, **transfer a mature method to a new domain**.

## Also worth recording: two directions were ranked highly and never gated

World-model evaluation was ranked first in [[Field-Scouting-Survey]] and never
tested, because the conversation pivoted to mathematics. That was a process
failure independent of the bar problem. It is now tested, and it holds up.

---

# Cluster A — construct validity of world-model evaluation

**Verified landscape.** 759 arXiv CS papers match `"world model"` + `benchmark`;
**50+ distinct named benchmarks** enumerated. `"video generation"` +
`"meta-evaluation"` returns **one** paper, and it is a hallucination detector,
not a meta-evaluation. **No cross-benchmark convergent-validity study exists in
video.** The machinery exists on the text side only — BenchBench (`2407.13696`),
Benchmark-squared (`2601.03986`), *When Benchmarks are Targets* (`2402.01781`).

**Three papers now assert the construct is broken; none measures it at scale.**
`2601.15533` (visual conflation) and `2606.15032` (L0-L7 decision-centric
hierarchy) are pure position papers with no experiments. `2607.07196`
(admissibility ladder) adds an **n=2 existence proof** — the model ranking higher
on visual quality ranks lower on action-following.

## A1 — "Your judge was never validated" (start here: under 100 GPU-hours)

**The concrete finding, verifiable from the paper.** VBench-2.0's human-alignment
claim is computed **at the model level over four models** (Sora, Kling 1.6,
HunyuanVideo, CogVideoX-1.5). A Spearman correlation over `n=4` can only take
values in `{0, ±0.4, ±0.8, ±1.0}`, and `rho = 1.0` carries `p = 0.083` — **not
significant at α=0.05**. So on the natural reading, no dimension of the field's
flagship video benchmark demonstrates statistically significant human alignment.
Worse, the reported values (99.31%, 95.46%, 81.70%) are **not in that discrete
set**, so the aggregation is under-specified in the paper and cannot be
reproduced from its description. Either way it is a finding.

The pattern recurs: WorldJen reports `rho = 1.000` (a symptom of tiny `n`, not a
good judge); MSVBench 94.4%, MSAVBench 91.5%, V2V-Bench 0.905, all model-level
aggregates over small model sets; **WorldModelBench calls its judge
"human-aligned" and reports no agreement statistic at all**; EWMBench and
WorldScore report none.

Meanwhile direct evidence says VLM judges are weak at exactly this task: TRAVL
(`2510.07550`) finds they need trajectory-aware fine-tuning to judge physics
implausibility; Physion-Eval (`2603.19607`) finds expert humans catch physics
glitches in **83.3-93.5%** of generated videos where VLMs do not.

**Why it is cheap.** VideoPhy-2 released **~102,000 human annotations**, the
per-sample auto-eval scores, and an **open-weight judge**. You can meta-evaluate
a judge **without generating a single video**.

**Three moves.** (a) Recompute agreement at the *sample* level from the released
labels and report the gap against the published model-level correlations.
(b) Power analysis — the `n` required for each published alignment claim to be
significant, with every claim re-plotted with its real interval. (c) Judge-swap
sensitivity — hold videos and prompts fixed, swap Qwen2.5-VL / InternVL /
Cosmos-Reason2 / VideoCon-Physics / a closed API, and measure leaderboard
movement. Add a closed-API drift check by rescoring the same videos months apart.

**Borrowable machinery, unapplied to video:** MT and summarization
meta-evaluation — *Guardians of the MT Meta-Evaluation* (`2408.13831`),
system-level versus segment-level correlation, and *The Autocorrelation Blind
Spot* (`2604.14414`), which found 42% of turn-level findings fail under
cluster-robust correction.

**Cost: 1-2 GPUs, a few days.**

## A2 — do world-model benchmarks measure the same thing?

Run 6-8 **existing public** benchmarks (VBench-2.0, WorldScore, VideoPhy-2,
WorldModelBench, EWMBench, PhyWorldBench, Physics-IQ-Verified) over **one shared
set of 10-15 open models under a single frozen generation protocol** — the thing
no leaderboard controls, since each uses different resolutions, lengths, seeds
and prompt upsampling. Report pairwise rank correlations with bootstrap
intervals; decompose variance into model / benchmark / prompt / seed; derive how
many models a leaderboard needs before a ranking difference is detectable.

**Calibration point:** Physics-IQ Verified re-audited *one* benchmark and the
ranking moved to **Kendall tau = 0.46**. If independent benchmarks agree at
tau ~ 0.5, the field's aggregate ranking is noise.

**Cost: 1-3 weeks on 8 GPUs**, generation-dominated.

**Closest competitor, and it is beatable.** WorldArena (`2602.08971`) correlates
**its own new composite** against downstream utility at `n=14` models and **two**
manipulation tasks, reporting `r = 0.360` for action planning without confidence
intervals. At `n=14`, `r=0.36` has a 95% interval of roughly `[-0.21, 0.75]` —
indistinguishable from both zero and 0.8. Risk: that team is well-resourced and
shipped 2.0 three months after 1.0.

## A3 — how much of a closed-loop score is the extractor?

KineBench (`2607.19876`) *asserts* that inverse-dynamics models are "brittle to
data outside their training distribution," creating attribution ambiguity
between world-model error and extractor error — then routes around it with
IDM-free 6D-pose extraction. **It never quantifies the attribution.** Evaluate
the same models under IDM extraction, IDM-free extraction, and ground-truth
actions on ManiSkill3/LIBERO; report how the extractor's contribution grows with
OOD distance. Deliverable: an attribution-corrected protocol.

## Infrastructure notes

`stable-worldmodel` (`rbalestr-lab`, 2.1k stars, pip-installable) is an
evaluation platform with 30+ environments and planner baselines — the most
useful single piece of infrastructure here. Open action-conditioned models:
**Cosmos3-Edge** (4B, OpenMDW-1.1, ungated, 23.9 s per 480p/189-frame clip on
one H100), **Matrix-Game-2.0** (1.8B, MIT, 25 FPS streaming), **V-JEPA 2-AC**
(MIT). Avoid Wan 2.5 (API-only), Open-Sora-Plan v1.5 (NPU-only), and
Cosmos-Predict2.5 (deprecated, gated, 870 s per video).

---

# Cluster B — open problems the authors themselves named

From mining **1,046 full papers** for explicit open-problem language (1,423
matched statements across 656 papers). **Methodological note worth keeping:**
"we lacked compute" limitations are mostly boilerplate and low-value. The
productive pattern is authors describing **a specific design they could not
run**, or **a mechanism they observed but could not explain**.

## B1 — isolate whether diversity collapse comes from method or data

*Where does output diversity collapse in post-training?* (`2604.16027`, Apr 2026)
states verbatim: *"No existing study isolates the role of the training method
from the training data, or the generation format from the model weights."* Their
own study was **observational** — read off released Olmo 3 checkpoints. Nobody
has run the interventional version.

**Why it is unusually tractable:** Olmo 3 releases open weights **and open
post-training data at every stage**, so the confound is directly manipulable.
Design: 2x2 over {narrow two-teacher vs broad multi-source data} x {SFT vs DPO},
matched token budgets, >=3 seeds, reporting reasoning-path diversity separately
from final-answer diversity. **1-7B on 8 GPUs.** Low-medium crowding.

## B2 — how do visual attention sinks emerge during training?

*See What You Are Told* (`2503.03321`, ICLR 2025, 139 citations): *"how visual
attention sinks emerge during the training process remain open questions."* The
exploitation side exploded (~20 follow-ups); **no paper among 100 citing works
addresses training-time emergence.**

**The decisive experiment is cheap:** freeze versus unfreeze the LLM backbone
during visual instruction tuning. Sinks appearing with a frozen backbone means
they are inherited from the text model; not appearing means visual tuning creates
them. **Either answer is publishable.** 7B on 8 GPUs.

## B3 — RLVR measurement controls the authors skipped for compute

*Hidden Costs and Measurement Gaps of RLVR* (`2509.21882`) states they excluded
reward-component ablations *"because they require additional training runs"*.
Take three celebrated RLVR gains, re-run with and without an
abstention/calibration reward term under strict budget parity and multiple seeds,
and separate distributional sharpening from capability expansion. GRPO at 1.5-7B
on 8xH100. RLVR is crowded; the **measurement-controls** framing is not.

## Others worth knowing

Speculative decoding with large-vocabulary drafters (`2502.05202`, ICML 2025 —
the authors explicitly propose it as future work, and released a benchmark
harness); in-domain versus open-domain SAE training data (`2501.06254`, ICLR
2025, partially answered in their own Appendix B); harness-level metrics
separating agent from scaffold (`2605.18747` has a literal "Open Problems"
section); self-correction-aware long-horizon metrics (`2509.09677`).

---

---

# Cluster C — mature statistical theory transferred to LLM practice

Two self-corrections from this survey, recorded because they narrow the wedge
honestly: **queueing theory for LLM serving is NOT open** (a real OR literature
formed in 2025-26 — Dai's fluid-limit throughput-optimality proofs finding
vanilla vLLM *not* maximally stable, Foster-Lyapunov stability under KV-cache
constraints validated within 10% on real GPUs, plus the Ye/Jaillet/Simchi-Levi
competitive-analysis line). And **design of experiments for prompt/scaffold
search is NOT empty** — four adjacent papers landed in ten weeks, including a
full `2^5` factorial over scaffold components with exact Shapley values
(`2605.05716`) and **CAFE** (`2607.10380`, **11 July 2026**), which ships
factorial design plus mixed-effects variance attribution for compound-AI
pipelines as a package.

## C1 — the split-plot error structure in LLM ablations

**The claim.** Nearly every LLM ablation is structurally a **split-plot**
experiment analyzed as if it were completely randomized. Model, checkpoint and
retrieval-index build are hard-to-change **whole-plot** factors; prompt wording,
temperature and k-shot are easy-to-change **sub-plot** factors. A split-plot
analyzed as a CRD pools two error terms into one that is dominated by the
sub-plot error — so **whole-plot effects are tested against too small an error,
inflating their significance.** Model-level claims are exactly the class this
field publishes most.

**Statistical nuance that must be handled, or a statistician reviewer kills it.**
The error has two distinguishable forms and the protocol has to say which it is
attacking:

- **Pseudoreplication** — "our method gains +2% across 3 models, p < 0.001,"
  where the p-value treats all runs as independent replicates when the effective
  n at the model level is 3. The correct error term for a method effect is the
  method x model interaction.
- **True split-plot error** — where whole plots are genuinely replicated and two
  variance components exist.

Whether models are **fixed** (you care about these specific models) or **random**
(a sample from a population of models) determines which applies. Get this
explicit in the design section.

**Why it survives the incentive objection that suspended
[[KD-Noise-Floor-Stage1]].** The deliverable is a **correct analysis with a
concrete alternative** (`lme4`, standard split-plot machinery), not an
exhortation to report more variance. It gives people something to *do*. Behaviour
change is plausible in a way "report error bars" is not.

**Differentiation is the risk, and it is tight.** CAFE is two weeks old and uses
mixed-effects models for **variance attribution**, not whole-plot **error
structure**. That distinction is real but narrow; the intro must state it in one
sentence and a reviewer will probe it. `2605.05716`'s own **183/325 submodularity
violations** are evidence that the field's universal one-factor-at-a-time
protocol fails — which the paper never says.

**Adjacent unclaimed pieces to bundle:** Lenth's method and half-normal plots for
unreplicated factorials (DBLP returns **1 hit in all of CS**, and LLM experiments
are expensive and therefore naturally unreplicated — precisely what the
pseudo-standard-error was invented for); **aliasing**, which no LLM ablation
paper discusses; and **fractionation at k >= 10**, where the one existing full
factorial stops at k=5, exactly where fractionation becomes necessary and real
scaffolds have 10-20 components.

**Template to imitate:** Tang, Lin & Sahni, **KDD 2024** (`2311.14698`) —
fractional factorial designs for DoorDash policy, 5% incremental profit at 67%
lower cost. Same argument shape, different domain.

Formalism: Wu & Hamada, 2-3 weeks. Compute: the smallest of anything surveyed.

## C2 — contamination as differential item functioning

Ranked highly by **two agents arriving from opposite directions**, which is the
strongest independent signal in the session. DIF is the psychometric machinery
for detecting items that behave differently for matched-ability subjects — the
exact shape of "this benchmark item is easy for a contaminated model but not for
an equally capable clean one." **Best use of the GPUs.**

## C3 — frailty correction to agent hazard curves

Fastest and near-zero risk. Population-level hazard can move in the **opposite
direction** to every individual hazard under unmodelled heterogeneity — a
standard survival-analysis result the field has no vocabulary for. Reportedly
adjudicates a live ICLR 2026 disagreement.

## C4 — fork-join tail latency for agent fan-out

Verified empty across four indexes (`fork-join` + LLM: 3 hits; OpenAlex 2022+
fork-join/straggler: 0; OpenReview: 12, none LLM). **TraceLab** (`2606.30560`,
three weeks old, ~430k tool calls from ~4,300 coding-agent sessions) reports
heavily-tailed tool calls and **explicitly recommends increasing fan-out
parallelism with no model of what that does to the tail.**

The non-obvious core: classical fork-join bounds assume independence, but agent
siblings share a prompt prefix, a model and a backend queue, so they are almost
certainly **positively correlated** — invalidating those bounds in a direction
nobody has measured. Theory is pre-built and uncited (Raaijmakers-Borst-Boxma,
*Queueing Systems* 2022, fork-join with redundancy under heavy tails). The LLM
twist: prefix-cache sharing makes a replica far cheaper than a classical
datacenter replica, so the optimal redundancy level moves.

**Avoid in this cluster:** stability/throughput queueing for serving (owned);
per-query adaptive stopping (saturated and rigorous — ConSol is a literal Wald
SPRT, CITE uses e-processes with matching minimax lower bounds); plain IRT; PPI
for LLM eval; signal detection theory for calibration.

---

# The convergence — one research program, five entry points

Four independent sweeps using four different toolkits arrived at the same
structural finding: **the field's evaluation and ablation practice has an
error-structure problem that flips published conclusions.**

- **DIF** — you must condition on ability.
- **Split-plot** — you must separate whole-plot from sub-plot error.
- **Frailty** — population hazard can move opposite to every individual hazard.
- **Effective sample size** — checkpoint rows are not independent observations.
- **Interference** — leaderboard units are not independent either.

These are not five unrelated papers. Any two cite each other naturally. If the
goal is a coherent body of work rather than a single submission, that is the
spine — and it is a **positive methodological program**, not an audit, which
is what makes it survivable.

## Recommended sequencing

**C1 first, and move within weeks.** Cheapest formalism, smallest compute, and
it is a **correctness** claim rather than an efficiency one — the strongest kind
of result, because published conclusions fail re-analysis. It also survives the
incentive objection better than anything else here, since it ships a concrete
alternative analysis rather than an exhortation. The window is genuinely tight:
CAFE is two weeks old.

**A1 as the parallel track.** Under 100 GPU-hours, almost no generation needed,
and the `n=4` Spearman finding is verifiable today from a published PDF. If it
lands, A2 and A3 extend naturally into one "construct validity of world-model
evaluation" submission.

**B1 when GPUs are free**, because it is a *positive* contribution explaining a
phenomenon rather than an audit. Nobody has to admit error for it to matter.

**B2 as the cheap high-variance bet** — one freeze/unfreeze experiment, either
outcome publishable.

**C2 as the follow-on** that spends the GPUs, and the natural second paper in
the error-structure program.

## Related

[[Field-Scouting-Survey]] — where world-model evaluation was first ranked and
then left ungated.
[[KD-Noise-Floor-Stage1]] — suspended; its incentive objection is why B1's
positive framing matters.

---

# Gated and killed: the weather-extremes adjudication

**SCOOPED 2026-07-26.** My "live contradiction" was already adjudicated in
print — **by one of the two papers I named.**

Biegert, Allen, Alber & Lerch (`2606.21170`, KIT/Lerch group — note Gneiting is
*not* an author) explicitly cite Zhang et al., diagnose the discrepancy, and
publish the mechanism, verbatim:

> *"a likely explanation for the discrepancy in our conclusions is that the
> twPCRPS measures the information content of the forecast when predicting
> extreme events, rather than directly assessing the accuracy of the
> deterministic forecasts themselves... due to the forecaster's dilemma, if we
> condition evaluation on an extreme event having occurred, then the AIWP models
> will perform poorly."*

They also isolate the ground-truth asymmetry Zhang et al. carry (ERA5 for AI
models, IFS analysis for HRES) and sweep 101 threshold quantiles including
`q = 1` specifically "facilitating a comparison with the work of Zhang et al."

**The factorial's premise is broken by theorem.** Its primary outcome was "which
factor flips the sign." But one level of the scoring-rule factor is
**known-improper** and the other **known-proper**: conditioning evaluation on
the observation being extreme is exactly the forecaster's dilemma, and scaling a
proper score by an outcome-dependent weight yields an improper score
(Gneiting & Ranjan 2011). Zhang et al. concede this in a Methods subsection
titled *"Forecaster's dilemma."* The study would rediscover a published theorem.

Prior adjudication is also crowded and predates both: Olivetti & Messori (GMD
2024) already used a weighted MSE; Loveday & Hertneky (`2510.25045`) do
threshold-weighted scoring independently; Pasche et al. (AIES 2025) is Engelke's
*own* group; and **ExtremeWeatherBench** (McGovern et al., Brightband + NCAR +
ECMWF, 329 cases, 724 commits) is in preparation.

**My error, again the same one.** I called it a "live contradiction" without
checking whether the later paper addressed the earlier. Reading from framing
rather than method sections — the third instance this session.

**What survives, workshop-sized and solo-feasible:** (a) out-of-sample validity
of twPCRPS at record thresholds — EasyUQ is fitted **in-sample** with `n = 702`
per grid point, so the tail at `q = 1` rests on a handful of observations, and
nobody has done the stability or conformal-IDR check; (b) **ground truth as a
first-class factor using METAR station observations**, which sit in the public
bucket and sidestep the "ERA5 is itself an IFS product" circularity that
structurally favours the IFS family. Both need a meteorology collaborator to be
credible at more than workshop scale.

---

# Gated: LLM automatic heuristic design audit — mostly closed

**Three control papers landed between February and July 2026, one six days ago.**

- **Gideoni, Risi & Gal** (`2602.16805`, Feb 2026), *Simple Baselines are
  Competitive with Code Evolution*: two nulls with *"no explicit fitness-based
  selection,"* budget-matched on API dollars, function evaluations **and**
  wall-clock. Result: sequential conditioned sampling *"matches or exceeds
  AlphaEvolve on 4/9 problems."*
- **Gupta et al.** (`2607.18235`, Berkeley/MIT, **posted six days ago**):
  *"more than 3.1 million LLM rollouts across 30 harnesses... All comparisons
  are budget matched by rollout counts,"* with bootstrap tests against a
  repeated Sequential BoN baseline and Holm correction. No harness beats the
  simple baseline; *"more complex OpenEvolve-style"* configurations do worst.
- **Quan, Sun & López-Ibáñez** (`2509.02297`), 3D packing — **inside the AHD
  lineage**, with a true null: *"the scoring function is replaced by a constant
  value, making the selection purely random."* EoH scores `777.3 ± 4.2` against
  S-GRASP `762`, ZHU `691`, exact `689`. The LLM heuristic beats First-Fit and
  loses to everything else.

**Verdicts.** Seed variance and best-of-N inflation: **scooped**, at a scale and
statistical standard not matchable. Harness ablation: *"selection adds little"*
is now published three times; only the strict **non-LLM proposer** arm is unrun,
and that is a section. Transfer evaluation: **scooped** twice over.

**What survives, and it is the only load-bearing item.** Nobody runs classical
automated algorithm configuration — irace, SMAC, GP-hyper-heuristics — as a
**rival designer at matched wall-clock and matched dollars**. Gideoni
budget-matches against *LLM* nulls; Quan compares to human heuristics but not at
matched discovery budget. RAISE dismisses classical configurators by argument
alone: *"they cannot design new algorithmic logic."*

Real remaining gap: **none of the three new papers touches the standard
EoH/ReEvo TSP/BPP/CVRP benchmark set with a classical configurator.** Frame as
*"which tool per dollar,"* not as a takedown — the takedown is done. **Window:
3-6 months.**

**One genuinely open alternative:** evaluation hacking in AHD has been measured
**exactly once** — Vesper (`2605.15221`) reports an 8.2% hack rate on **two
runs**, with no non-LLM control and no taxonomy, plus the striking and wholly
unreplicated claim that **more capable models hack more**. Cheap, uncontested.

## The structural lesson this makes unavoidable

Three strong control papers in five months, one within the week. **The field is
self-correcting faster than an audit can be written and reviewed.** That is a
fact about audit-shaped work in 2026, and it argues systematically for
**positive contributions** — explain a mechanism, supply a method — over
critiques, however well-gated the critique is.

Re-ranking accordingly: **B1 (diversity collapse) and B2 (attention sinks) rise
above A1 (judge audit)**, despite A1 being cheaper and already gated. Positive
contributions are not on the clock the same way.

**Honest status: A1 and the weather/AHD/compression items are gated. C1, B1, B2
and the two surviving AHD items are NOT.** Do not treat the ungated ones as
solid — that pattern has cost this session repeatedly.

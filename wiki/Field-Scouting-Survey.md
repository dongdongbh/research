# Field Scouting Survey — Beyond Compositional VLMs

Status: **All survey parts finished on 2026-07-25.** This page covers
scientific machine learning and agent/world-model research. Separate work on
representation geometry, new topics, and math venues is in
[[Math-Grounded-Direction-Survey]].

**Updated 2026-08-02 for the general research wiki.** Later checks changed
several decisions below. Each affected section has a dated correction. The old
text remains so that the research record is complete. For current decisions,
use [[Unified-Direction-Ranking-2026-08]].
[[Direction-Reevaluation-2026-08]] is the historical August 2 ranking, and
[[Top-Researcher-Scan-2026-08]] records supporting researcher-level evidence.

This survey followed [[Calibration-Prior-Art-Gate]], which ended the last idea
in compositional VLMs. This page does not repeat that project's probes, stages,
or paper numbers; the full record is in the svib repo wiki. Every idea here had
to fit these limits: **one researcher, 4–16 L40S/A100/H100/H200 GPUs, no budget
to pretrain a model, suitable for registering the plan before running it, and
well matched to our strength in controls and attempts to disprove claims.**

## The main pattern

Every area in this survey had the same kind of promising paper: it compared
methods at the same compute cost, or held other possible causes steady, and it
reported how much results changed across runs. In 2026, models and benchmarks
are common. Good control experiments are rare. Our group has spent six months
building exactly that skill.

One sign of an opening is the **method-to-critique ratio**: the number of papers
that propose methods compared with the number that carefully test those
methods. When the ratio is 100:1 or worse, the test side may be empty because
few people work on it, not because the work has little value.

| Area | Method papers (2025) | Critique or audit papers |
|---|---:|---:|
| Physics-informed neural networks | 6,237 | ~10 |
| Neural operators | 5,691 | ~10 |
| World-model benchmarks using "we introduce" | 295 | **1** paper testing whether the metric is valid |
| Neural combinatorial optimization | 524 | ~10, and now well covered |
| LLM automatic heuristic design | 82+ | **1**, a workshop suite |

> **Partly replaced on 2026-08-02.** This ratio can point to a broad research
> area. It cannot tell us whether someone already answered the **exact**
> question, or how quickly the area is filling. Both errors happened within
> weeks: the weather and AHD critique ideas below had already been done even
> though their ratios still looked good. The rule that survived is: ask whether
> new questions appear faster than papers answer them, **and whether this exact
> question is already claimed**. See meta-lesson 1 in
> [[Direction-Reevaluation-2026-08]]. Also follow the portfolio rule from
> [[Top-Researcher-Scan-2026-08]]: when registering an audit, name the method
> paper it will make possible, and release a reusable tool or dataset. A simple
> audit must move quickly. A careful test between explanations, or a study of a
> cause, usually has more time.

## Best opportunities found in this survey

### 1. World-model evaluation — the highest-leverage opening

The survey's clearest number is **295** arXiv abstracts matching
`"world model" AND "benchmark" AND "we introduce"`. Only **1** abstract
matches world model with meta-evaluation or metric validity. About 300 groups
built world-model benchmarks, but almost nobody checked whether those
benchmarks agree with one another or predict useful later behavior.

Companies fully control frontier models: NVIDIA Cosmos,
DeepMind [Genie](https://arxiv.org/abs/2402.15391), Tencent, Skywork, and many
2026 systems. **That is a reason the measurement problem is open.** A company
with a frontier model has little reason to publish that the field's metrics do
not work. This study needs only inference on released models, so 4–16 GPUs are
enough.

Two position papers—[`2601.15533`](https://arxiv.org/abs/2601.15533), which
names "visual conflation," and
[`2606.15032`](https://arxiv.org/abs/2606.15032), which names a
"claim/evidence mismatch"—say there is a problem but do not measure it. Turning
those statements into evidence would be a clear contribution others can cite.

Possible registered studies:

- Test whether benchmarks meant to measure the same thing actually agree. The
  registered guess is that the median pairwise Kendall rank correlation will
  be below 0.5.
- Test whether physical-plausibility metrics survive changes that should not
  change the meaning of a video, such as bitrate, frames per second, and
  resolution.
- Test whether a benchmark score predicts usefulness for later planning.

The main risk is **finding the right venue, not another group publishing
first**. CVPR and ICCV favor new benchmarks over studies of existing ones.
Possible homes are World Model workshops, TMLR, or Data & Benchmarks tracks.

> **Update 2026-08-02 — the direction survives, becomes broader, and is no
> longer the single top priority.** A new search found the same problem in many
> places besides world models. At least five public judge or evaluator systems
> have never had an independent validity audit, and eight senior researchers
> name verification as the main problem. The right unit is therefore **one
> registered audit method used across many targets**—a research program, not
> one paper. It is Tier 1 in [[Top-Researcher-Scan-2026-08]]. By itself, the
> world-model judge audit is ★★★ in [[Status-And-Survivors]]: it is the cheapest
> real result, passed its gate, and still matters, but another group could run
> the audit quickly. Name the method paper it enables before starting.

### 2. Weather and climate ML — an apparent disagreement, almost no compute

> **SUPERSEDED 2026-07-26 — the disagreement had already been explained.**
> [`2606.21170`](https://arxiv.org/abs/2606.21170) cites the Science Advances
> paper and explains why they differ. twPCRPS measures how much information a
> forecast gives, not the accuracy of one fixed prediction. Also, checking only
> cases where an extreme event happened creates the *forecaster's dilemma* and
> is not a proper comparison. A published theorem already answers the planned
> experiment's main question, "which choice reverses the result?" Two smaller
> ideas survive: test twPCRPS on new data at record-level thresholds, and treat
> ground truth as an experiment choice by using METAR station observations.
> Both need a meteorology collaborator and rate ★ in
> [[Status-And-Survivors]]. Full gate record:
> [[Live-Research-Opportunities]], "Gated and killed: the weather-extremes
> adjudication." The original proposal remains below as history.

Two 2026 papers use **the same [WeatherBench 2](https://arxiv.org/abs/2308.15560)
data** and seem to reach opposite results:

- *Physics-based models outperform AI weather forecasts of record-breaking
  extremes* (**Science Advances 2026**,
  [`2508.15724`](https://arxiv.org/abs/2508.15724)): HRES beats
  [GraphCast](https://arxiv.org/abs/2212.12794),
  [Pangu](https://arxiv.org/abs/2211.02556), and
  [FuXi](https://arxiv.org/abs/2306.12873) on record-breaking events at nearly
  every forecast distance.
- *Towards Fair Comparisons... via the Weighted Potential CRPS*
  ([`2606.21170`](https://arxiv.org/abs/2606.21170), Gneiting group): after
  isotonic-distributional-regression post-processing and weighted CRPS,
  **FuXi gives the most informative forecasts for extreme events**.

The original proposal said both summaries could not be right. It blamed a
method choice: raw fixed predictions versus calibrated probability scores. It
proposed an experiment comparing every combination of {scoring rule} ×
{post-processing} × {reference} × {event definition} × {model}. The registered
outcome would be which choice reverses the conclusion.

**The compute would have been almost zero.** WeatherBench 2 forecasts are
already stored as public Zarr data on Google Cloud. The study would have tested
claims that cost 10^5–10^6 GPU-hours to create. It had the best ratio between
verification cost and original-claim cost in the survey.

Two related ideas were: test for contamination in the thing used as truth,
because every model is scored against ERA5 even though ERA5 is itself a model
product and [`2601.04701`](https://arxiv.org/abs/2601.04701) shows GraphCast can
find ERA5's own errors; and make initial conditions fair, because AI systems
start from ERA5 while operational numerical weather prediction (NWP) starts
from real-time analysis. Independent station observations could address the
first issue.

The risk was competition from Gneiting's group and ECMWF, both fast and strong.
The plan allowed about three weeks to learn proper scoring rules and the
double-penalty problem.

### 3. LLM automatic heuristic design — few broad critiques

> **SUPERSEDED TWICE — read both corrections before using the old proposal.**
>
> **(a) 2026-07-26: the named controls were already done.** Three control
> papers appeared from February through July 2026, one within a week of our
> gate. Seed variation and best-of-N score inflation had been tested at a scale
> we cannot match. Three papers found that the selection harness adds little.
> Only a strict non-LLM proposer comparison remains, and that is one paper
> section rather than a whole paper. Two papers already tested transfer. One
> idea remained: compare classical tuners such as irace and
> [SMAC](https://arxiv.org/abs/2109.09831) with an LLM as **rival designers at
> the same wall-clock time and dollar cost**, using the normal
> [EoH](https://arxiv.org/abs/2401.02051) /
> [ReEvo](https://arxiv.org/abs/2402.01145) TSP/BPP/CVRP tasks. Frame it as
> "which tool gives the best result per dollar," not as an attack. Evaluation
> hacking is also still open and has been measured only once. Details:
> [[Live-Research-Opportunities]].
>
> **(b) 2026-08-02: the broad direction rose from ★★ to ★★★★.** The old audit
> was not revived. Instead, the direction became two questions: (i) a
> cost-adjusted line showing when LLMs or classical methods win, which does not
> exist, and (ii) a novelty audit that labels results as rediscovery,
> recombination, or truly new, which has zero papers. New uses are appearing
> about 10–15 times faster than control studies. Abstract counts for
> "automated heuristic design" are 1/1/5/**17** in
> 2023/24/25/January–July 2026. The search covered arXiv only, so check
> GECCO and PPSN before starting. See [[Direction-Reevaluation-2026-08]].

The FunSearch / EoH / ReEvo /
[AlphaEvolve](https://arxiv.org/abs/2506.13131) line says LLMs discover
better-than-human heuristics. Focused searches returned only **1** critique, a
GECCO workshop suite. The query
`"evolutionary" AND "LLM" AND "heuristic" AND "seed" AND "variance"`
returned **0**.

The claims were huge, but an independent check would need only CPU-hours and a
local open-weight model. The original proposed controls, then believed unrun,
were:

- Measure variation across random seeds and best-of-N inflation. Is the
  published score above the 90th percentile of the seed distribution?
- Replace the LLM proposer with a random program mutator that uses the same
  program skeleton. If the gain remains, the harness did the work, not "LLM
  discovery."
- Give the same time and money to `irace` or SMAC for tuning LKH-3 or
  [HGS](https://arxiv.org/abs/2012.10384), then compare results.

**Locking one open-weight proposer version and hashing its source is itself a
useful method here.** Current AHD papers cannot be exactly reproduced when a
vendor silently changes its model.

### 4. PDE foundation models — a missing control experiment

The search `abs:"pretraining" AND abs:"physics" AND abs:"compute-matched"`
returns **exactly 0** arXiv papers. [Poseidon](https://arxiv.org/abs/2405.19101)
(NeurIPS 2024) and later work say that pretraining on many physics problems
greatly reduces the number of examples needed across 15 later tasks. Nobody has
run the controls that would test why: random starting weights at the same total
FLOPs, pretraining on physics data with its meaning scrambled, or pretraining
on Gaussian random fields with matching frequency patterns.

**Break-even cost accounting is also open.** Surrogate-model papers report fast
inference, but almost never include the solver cost of making training data, the
training cost, or a solver comparison at the same accuracy. The search
`abs:"training data generation" AND abs:"cost" AND abs:"amortiz"` returns
**0**.

The field already says this kind of work matters. *Common Task Framework for a
Critical Evaluation of Scientific ML* (**NeurIPS D&B 2025**,
[`2510.23166`](https://arxiv.org/abs/2510.23166)) names weak baselines and
uneven evaluation as the problem. Cite it as support for the method.

### 5. MLIP and materials — excellent cost ratio, but needs subject help

The search `abs:"Matbench Discovery" AND (abs:"leakage" OR abs:"compliance")`
returns **0**, even though
[MLIP Arena](https://arxiv.org/abs/2509.20630) calls leakage a major unsolved
problem. Modern universal machine-learned interatomic potentials (MLIPs) train
on structures that overlap the WBM test set in shape, not only in chemical
formula. Grouping results by this overlap needs only inference on 257k
structures. It would take hours on a few GPUs while testing models that cost
10^5 GPU-hours to train.

Any claim that needs density-functional theory (DFT) or X-ray diffraction
(XRD) to verify a discovery needs a chemistry collaborator. A solo researcher
can still group results by leakage and test energy-conservation behavior.

## Areas not to enter

| Area | Reason |
|---|---|
| **Agent evaluation** | The problem is real, but the area is packed: 370 agent-benchmark abstracts, more than 10 workshops in one cycle, a 27-author Princeton-led group running 21,730 tests, two competing surveys of test-harness engineering, and even a paper that audits the auditors. SWE-Bench+ was withdrawn from ICLR 2026 and rejected from ICLR 2025. |
| **AI for math and theorem proving** | Already studied heavily (`abs:"miniF2F"` gives 107). More GPUs do not help much because formal checking is limited by token generation. The obvious audit was just published: *[Faults in Our Formal Benchmarking](https://arxiv.org/abs/2606.29493)*, ICML 2026, with 4,833 findings. |
| **Chip placement** | Repeatedly argued in public and dangerous to reputation; Nature published an Addendum in 2024. |
| **PINNs vs FEM** | Grossmann et al. 2024 already settled it. |
| **Neural CO vs classical solvers** | [MaxCut-Bench](https://arxiv.org/abs/2406.11897) and [FrontierCO](https://arxiv.org/abs/2505.16952) (ICLR 2026) already settled it. |
| **Multimodal benchmark blind-solvability** | The main result dates to 2024 and is now expected. Only the test of robustness to irrelevant changes, opened ~7 months ago, remains. |

## Venues that accept this kind of work

**Tier A.** TMLR clearly welcomes reproduction studies, judges evidence quality
rather than whether a result is exciting, accepts papers all year, and offers a
Reproducibility Certification. Other choices are the NeurIPS Data & Benchmarks
Track and the ICML/ICLR main tracks. Those main tracks do accept critique papers
and sometimes give them oral presentations: *[The dark side of the
forces](https://arxiv.org/abs/2412.11569)*, ICML 2025 Oral;
[FrontierCO](https://arxiv.org/abs/2505.16952), ICLR 2026; and
*[Faults in Our Formal Benchmarking](https://arxiv.org/abs/2606.29493)*, ICML
2026. A new method is not always required. The ICML Position track fits papers
whose main contribution is a well-supported argument.

**Tier B: subject-area venues that may give scientific claims more trust.**
Nature Machine Intelligence published
[McGreivy](https://arxiv.org/abs/2407.07218) & Hakim and
[Matbench Discovery](https://arxiv.org/abs/2308.14920). It also has *Matters
Arising* for direct replies. Other choices are Machine Learning: Science and
Technology, which clearly welcomes negative results; npj Computational
Materials; PRX Energy; npj Climate and Atmospheric Science; AIES from AMS; and
Science Advances.

**Tier C.** ICBINB, "I Can't Believe It's Not Better," at NeurIPS is built for
negative results. Other choices are ML4PS and AI4Mat.

## Survey work is closed

On 2026-07-25, [[Math-Grounded-Direction-Survey]] finished the remaining work
on representation geometry, including GeoLAN, emerging 2025–26 topics from
best-paper and new-workshop lists, and recent math results that might apply to
ML. [[Direction-Gate-Results]] then checked the ideas for prior work. This page
will not be extended further.

## Related

[[Unified-Direction-Ranking-2026-08]] — current direction decisions.
[[Direction-Reevaluation-2026-08]] — historical August 2 ranking.
[[Top-Researcher-Scan-2026-08]] — supporting researcher-level evidence.
[[Math-Grounded-Direction-Survey]] — theory parts of the same survey.
[[Live-Research-Opportunities]] — full records for the weather and AHD gates.
[[Calibration-Prior-Art-Gate]] — ended the last compositional-VLM idea.
[[Next-Direction-Literature-Survey]] — the earlier, narrower survey.
[[Status-And-Survivors]] — priority table for all surviving directions.

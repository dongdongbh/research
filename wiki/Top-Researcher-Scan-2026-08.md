# What 26 Leading Researchers Are Working On — 2026-08-02

**Refreshed 2026-08-31: see [[Researcher-Scan-Refresh-2026-08-31]]** for the
delta — star changes, opening-status changes, five corrections to this page
(including: CoVFT is a Beihang paper, not Darrell's; the autoresearch FDR
opening was already gate-marked SCOOPED), and the new-opportunity list. For
any commitment, trust the refresh, not this page.

This page tracks researchers and the openings around their work. Read it with
[[Unified-Direction-Ranking-2026-08]], which gives the current star ranking for
research directions. [[Direction-Reevaluation-2026-08]] keeps the earlier
August 2 ranking and the reasons behind it.

**Status: complete.** Twenty-six Opus agents each studied one leading
researcher. One agent had to resume after its first report was cut short. We
gave the most weight to small papers where the person was the last author,
solo and position papers, talks, code changes, and hiring pages. We gave less
weight to large group papers and raw publication counts. We checked current
jobs and searched whether each proposed opening had already been claimed.

Google Scholar blocked almost every search. Evidence came from arXiv,
OpenAlex, Semantic Scholar, personal pages, GitHub, and direct reading of
papers and released files.

We used these lab limits: L40S/A100/H100/H200 GPUs, usually 100–1,000
GPU-hours, no robot hardware, strong AI-assisted engineering, and experience
with pre-registration, tests that compare possible causes, honest negative
results, and VLM evaluation tools.

## Researcher map

The stars rate fit for us, not a person's overall importance. “Opening” means
a question that appeared unclaimed at scan time. A **direct cause test** is an
experiment that compares competing explanations. A **combination experiment**
tests several choices both alone and together.

| Person | Fit | Main current work after removing weak signals | Best opening for us and estimated cost |
|---|---|---|---|
| Karpathy | **5** | [nanochat](https://github.com/karpathy/nanochat) speed runs and [autoresearch](https://github.com/karpathy/autoresearch), with 92.8k stars | Measure the false discovery rate (FDR) of its one-run greedy accept rule; FDR is the share of accepted findings that are false; 200–600 GPU-h |
| Finn | 4 | Verification as a way to scale, value-based RL, and world models used as evaluators | Compare explanations for how verification scales, 150–400; check whether evaluators are valid |
| Levine | 4 | Scalable off-policy RL with flow policies in simulation on one GPU | Compare four possible causes of scaling behavior on [OGBench](https://arxiv.org/abs/2410.20092), 200–500 |
| He | 4 | One-step image generation and tests that remove model parts | In one-function-evaluation (1-NFE) models, test whether loss of diversity comes from averaging or the method itself; use Drifting to disprove the claim if possible; 150–400 |
| Abbeel | 4 | Fast, cheap RL and open simulators, not world models | Test all combinations of five parallel-RL ingredients, 300–800 |
| Bengio | 4 | Scientist AI, still a position idea with no code | Test epistemic contextualization, which labels how well-supported text is, with a fake-treatment control; 400–900. Also test the sparsity assumption; 100–300 |
| Song | 4 | [UMI](https://arxiv.org/abs/2402.10329) hardware line plus a small-author core on robustness and RL fine-tuning | Compare three possible causes in GMP, 100–300; audit Cosmos Policy |
| Neubig | 4 | “Verification is the bottleneck,” agent training, and evaluation | Equal-budget 2×2 test of CAID using only APIs; future-time [PACE](https://arxiv.org/abs/2606.08106) test at almost zero cost; train a critic on our traces and test transfer, 100–500 |
| Choi | 4 | Output distributions becoming alike during optimization, RLVR limits, and AI for science | Break down the Artificial Hivemind cause at low cost; connect G-Vendi to output diversity at low cost |
| LeCun | 4 | [LeJEPA](https://arxiv.org/abs/2511.08544) theory and planning; left Meta for AMI Labs in November 2025 | On public checkpoints, test planning success against search budget; 100–300 |
| Liang | 4 | Science for data-limited, high-compute settings; memorization studies; [Marin](https://github.com/marin-community/marin) | Test replay causes at 150M and 4B cheaply; Marin may also provide compute |
| Arora | 4 | Post-training with dense signals, skill tests, and published negative results | Test whether drag equals fork suppression, 50–150; test prefix-completion generalization, under 50 |
| Darrell | 4 | A VLM perception bottleneck with four proposed fixes that have not been compared | Add a VPBench hidden-factor grid to our compositional tests at almost zero cost; test where in the model the failure occurs |
| D. Chen | 4 | Managing agent context, post-training science, and training-free methods | Measure KV-cache use in agent workloads, 200–400 |
| Isola | 4 | [PRH](https://arxiv.org/abs/2405.07987), then canonicalization with an orthogonal map `Q`, plus emergent intelligence | Find where `Q` fails on compositional tests and test whether data overlap explains it; 50–150 |
| M. Li | 4 | Active-Passive Gap, [RAGEN](https://arxiv.org/abs/2504.20073), [VAGEN](https://arxiv.org/abs/2510.16907), and tests of spatial beliefs | Test AdaptVis's claimed cause on our benchmark set, under 50; predict static-to-active change, under 100 |
| B. Zhou | 4 | Full systems for sidewalk micromobility; interpretation work is no longer active | Add a VLM-blind control to VLM-guided navigation, 150–300; compare VQA scores with closed-loop control |
| Fan | 4 | World Action Models, described as replacing VLAs; [GR00T](https://arxiv.org/abs/2503.14734) and [DreamDojo](https://arxiv.org/abs/2602.06949) | Use the N1.6-to-N1.7 checkpoint change to test EgoScale, 50–150 inference-only; audit the DreamGen judge, 30–80 |
| Fidler | 2 to follow; 4 to audit | NVIDIA's spatial-intelligence product line | Audit DriveJudge, 20–100; test whether long reasoning hides perception errors, 50–200 |
| Shazeer | 4 | Quiet publicly; a Hot Chips talk shows the plan; moved to OpenAI on 2026-06-18 | At equal KV-cache bytes, test MQA trained from scratch, about 400 |
| Wei | 4 for the idea; 1 for following the person | No public work at Meta since mid-2025; verifier rule has not been tested | Test whether knowing a task is easy to verify predicts benchmark saturation, under 100; test five claimed properties together or separately, 300–800 |
| Beyer | 3 for the person; 4 for the questions | Moved to Meta Zurich in July 2026; the [SigLIP 2](https://arxiv.org/abs/2502.14786) line is frozen and has no clear owner | Test NaFlex asymmetry on L40S, 50–150; test where binding fails, 100–200 |
| Sutskever | 3 overall; 2 to follow; 4 for claims | SSI publishes nothing; a November 2025 interview is the main artifact | Track whether RL environments are derived from evaluations; 200–600 |
| D. Zhou | 3 | Paper output has stopped; a CS25 slide deck lists four unscoped rankings | Test the tension between RL and aggregation by measuring self-consistency calibration after RLVR; 100–300 |
| Sadigh | 3 | Data mixtures for general robot policies, which need hardware, and set-RL for LLMs | Compare polychromic results at matched temperature, 200–600 |
| Malik | 3, with one 5-star opening | Touch-based robot skill from human video, which needs hardware | Add a VLM arm and a cause test to [MOCHI/Bonnen](https://arxiv.org/abs/2409.05862)'s 3D-perception claim; 20–60 |

## Themes shared by several researchers

### V. Checking results is the bottleneck, but the checkers are not checked

Karpathy says Software 2.0 can automate tasks that can be verified, yet his own
accept rule is statistically weak. Finn treats verification as a scaling
direction. Neubig says agents made code generation cheap and moved the
bottleneck to checking. Wei's verifier rule has never been tested and, by his
own account, uses circular evidence. D. Zhou calls a reliable verifier the most
important part. Sutskever worries that evaluations differ from reality. Bengio's
Scientist AI is a plan for calibrated checking but has no implementation.
Google's Science One points the same way.

At least five released judges or evaluators have no independent validity test:
Finn's world-model evaluators, Levine's
[SC3-Eval](https://arxiv.org/abs/2606.18610), Fan's
[DreamGen Bench](https://arxiv.org/abs/2505.12705) VLM judge, Fidler's
DriveJudge, reward judges near Abbeel's work, and B. Zhou's assumption that
[MetaVQA](https://arxiv.org/abs/2501.09167) predicts control. One shared audit
method could test many targets. This is a research program, not one paper.

### D. Optimization often makes outputs less varied

At least seven independent lines report this pattern:

- Choi's [Artificial Hivemind](https://arxiv.org/abs/2510.22954) and
  [Invisible Leash](https://arxiv.org/abs/2507.14843), where the cause remains
  unclaimed;
- Sadigh's polychromic set-RL, which lacks a control for a hidden factor;
- Sutskever's claim that post-training competition, not temperature, removes
  diversity; this has not been tested;
- D. Zhou's unresolved tension between RL beating supervised fine-tuning and
  aggregation beating one answer;
- He's one-step generation;
- Isola's Digital Red Queen behavior convergence; and
- Weng's fourth bottleneck, plus our older B1 and entropy work.

Choi already owns the measurements. The cause remains open at every level.

### A. Many papers claim a cause without comparing other causes

Examples include Levine's four scaling fixes, Abbeel's five mixed-up parallel-RL
ingredients, Darrell's four possible VLM failure locations, Song's three
bundled GMP causes and two sibling methods never compared, He's part-removal
work on only one benchmark, SigLIP 2's bundle of methods with no study of each
part, M. Li's AdaptVis outside spatial tasks, and B. Zhou's untested causes.

A group that built a method has a strong reason not to disprove its own story.
We can run fair tests among the explanations.

### W. World models are replacing vision-language-action models

Fan says this directly. Fidler, Finn, LeCun, and Malik point the same way. Most
work needs robot hardware or large compute. Checking evaluator validity is the
part we can do.

### H. Human video is replacing teleoperation data

Malik, Song, Finn, Fan, and Abbeel point here. We should skip most of it because
it needs robot hardware. Fan's trick of comparing two public checkpoints is
the exception.

### S. Small from-scratch studies are a real academic advantage

Shazeer's usual KV-cache choice comes from a weight-conversion setting, not a
clean model trained from scratch. Beyer wrote about results reversing with
scale but cannot publish the direct comparison from inside Meta. Sutskever
noted that the transformer was built on only 8–64 GPUs and that research may
not need the largest compute. Liang publishes two-author studies at 150M or 4B
tokens. Small, controlled training can answer questions that large labs skip.

## Work habits shared by successful researchers

1. **Follow a person's current small-team papers, talks, course pages, code
   changes, and hiring pages—not total citations.** B. Zhou's 71,000 citations
   mostly describe 2016. Bengio's hiring page showed more than his papers. D.
   Zhou's biography changes revealed a career shift.
2. **Build the measurement tool before the method.** Examples are D. Chen's
   [HELMET](https://arxiv.org/abs/2410.02694) before
   [ProLong](https://arxiv.org/abs/2410.02660), M. Li's benchmark family, and
   Karpathy's leaderboards.
3. **Release the useful artifact in the same week.** This may be checkpoints,
   simulators, smaller experiment repositories, or Liang's Marin. The artifact
   makes the work harder to replace.
4. **Essays and blogs can set research plans between papers.** Wei, Karpathy,
   Park/Levine, and Weng do this. A public blog claim can be a research target
   that a paper cites.
5. **Diagnostic papers build trust; released tools earn citations.** Arora's
   diagnostic papers received 0.3–0.8 citations per month versus 9–12 for
   artifacts. Darrell's evaluation papers received about 5 citations versus
   hundreds for methods. Fidler's side evaluations received about 0 versus
   16–41 per month for Cosmos. Pair every cause-comparison study with a public
   test harness, benchmark, or metric.
6. **Measure structure, not only a final score.** Choose a structural measure
   such as tree-edit distance, fork rate, or expression trees before the run.
   This is what lets a small experiment support a claim about cause.
7. **Publish negative results as real results.** Arora did this in 4 of 12
   recent papers. Liang disproved optimizer claims. Karpathy keeps `dev/LOG.md`.
8. **Make pre-registration a normal system.** Marin moves from issue to quick
   check, dry run, and full run, while publishing failures. This matches our
   practice and provides stronger infrastructure.
9. **Buy many controlled runs instead of one huge run.** RLMT ran 40 tests at
   7–8B.
10. **Prefer methods that need no training when they answer the question.**
    Most of D. Chen's 2026 first-author work uses no gradient updates. Both
    Princeton groups have real clusters but still choose small studies. Low
    compute can be a research-design choice, not only a limit.
11. **Before choosing a target, check that its code and model files are alive.**
    D. Chen's interpretation idea looked ideal, but its repository was dead
    and had no maintainer. Check recent pushes, maintainers, and released
    checkpoints.
12. **Use AI-assisted engineering for test harnesses and environments.** Other
    groups often fail to build these. Examples are Karpathy's `program.md`,
    Marin's rule for accepting agent pull requests, and M. Li's call for
    standards that report full trajectories.

## New ideas found across researchers

### Tier 1: cheap enough to start now

Unless noted, each costs under 150 GPU-hours.

1. **Program to check judges and evaluators.** Pre-register one method with a
   hidden-factor test set, gameability tests, out-of-distribution policies,
   and a check for ranking changes when the judge changes. Apply it to
   DriveJudge for 20–100 GPU-h, DreamGen Bench for 30–80,
   [SC3-Eval](https://arxiv.org/abs/2606.18610) with
   [Ctrl-World](https://arxiv.org/abs/2510.10125) for 300–800 together, and
   [RoboReward](https://arxiv.org/abs/2601.00675). Nobody claimed these audits.
   NVIDIA and the original PIs have a built-in conflict in auditing themselves.
2. **Karpathy autoresearch FDR study, 200–600 GPU-h.** It fits all five of our
   criteria and has 13,200 forks. Move quickly.
3. **MOCHI/Bonnen 3D-perception cause test, 20–60.** Add the missing VLM arm.
4. **AdaptVis cause test on our compositional benchmark set, under 50.**
5. **SigLIP 2:** test NaFlex asymmetry for 50–150 L40S-hours and where binding
   fails for 100–200 GPU-h.
6. **Find Isola's canonicalization failure boundary, 50–150.** Separate data
   convergence from representation convergence.
7. **Neubig:** run an equal-budget 2×2 CAID test using only APIs, plus the
   future-time [PACE](https://arxiv.org/abs/2606.08106) test at almost zero
   cost.

### Tier 2: medium cost, 200–900 GPU-hours

Choi hivemind cause breakdown · Levine OGBench four-cause test · Abbeel
parallel-RL combination study · Shazeer equal-KV-budget study, about 400 · Wei
verifier-rule tests · Sutskever RL-environment source study · Bengio sparsity
test · Liang replay-cause test · D. Chen agent KV-footprint test · Sadigh
polychromic audit · He one-step diversity test.

## New-method directions

The scan looks biased toward audits only because the summary led with them.
The same profiles contain openings for new models, architectures, training
methods, and working systems.

### M1: A KV-efficient architecture recipe trained from scratch

Test every combination of KV heads, sharing across layers, and local versus
global attention at the same KV-cache size. Train from scratch and release the
best settings and code. The usual grouped-query attention (GQA) choice comes
from converting an already-trained model. Character.AI never compared all four
parts of its system together. Cost: about 400 GPU-hours at 300M–1B parameters.
If multi-query attention (MQA) wins from scratch, the paper offers a cheaper
architecture, not only a criticism.

### M2: A statistically sound harness for agent research

The third part of the FDR study is a method. Build and release an accept rule
that gets the most findings that later survive per GPU-hour. Possible parts are
racing several seeds, tests that decide in steps, and promotion rules that use
variance. Package it as a drop-in `program.md` and harness for autoresearch's
13,000 forks. The diagnosis justifies the work; the accept rule is the main
contribution.

### M3: A critic that transfers across agent systems

Train an 8B rubric critic on
[OpenHands](https://arxiv.org/abs/2407.16741) traces plus our Claude Code and
Codex traces, which the original paper did not have. Release a reranker or
early-stopping model and measure whether it works on a different agent system.
Cost: 100–500 GPU-hours. It could be both a paper and an internal tool.

### M4: First implementation of epistemic contextualization

Bengio's position paper names a method that rewrites a training collection to
show how well-supported each claim is, but nobody has built it. Build the
rewriter, train matched 0.5–1.5B models, and release the pipeline and
checkpoints. A fake-treatment arm makes it a fair science experiment. The
working pipeline makes it a new method. Cost: 400–900 GPU-hours.

### M5: Post-training that keeps output diversity

First, the diversity-cause study in theme D must find the real cause. Then
build a method that keeps broad output coverage while matching `pass@1`. It
would compete with [Spectrum-Tuning](https://arxiv.org/abs/2510.06084) and
polychromic methods, but use a cause-based design instead of another guess.
Run only if the diagnosis succeeds. Cost: 300–800 GPU-hours.

### M6: KV-cache eviction that understands agent turns

First measure whether the memory-quality tradeoff changes at conversation turn
boundaries. If it does, build an eviction rule that uses how long information
will matter and recognizes tool-output structure. This would extend existing
GPU kernels and could join our safety-aware allocator work.

### M7: Engineer verifiers to resist gaming

Wei argues that verification can improve when researchers study the task first.
At scan time, nobody had published one careful method for hardening verifiers
across a task family. The method would design environments that resist reward
hacking and turn Karpathy's resettable-and-rewardable rules into code. This is
close to M2. Search the newest RL-environment papers before starting.

**Rule from now on:** every audit or cause-comparison project must name, at
pre-registration time, the method paper it could unlock. It must also name the
public harness that will support both papers.

### How this changes the older ranking

- Recheck **T1** immediately against Darrell's C1 group: CoVFT and its four
  encoder fixes. That group is now the main risk of publishing first.
- The judge-audit program is a new top candidate beside T1 and T3.
- Tier-1 ideas 3–5 are small additions to our existing compositional VLM tests
  and can run beside other work.
- Contributing to [Marin](https://github.com/marin-community/marin) could give
  access to compute through its JAX/[Levanter](https://github.com/stanford-crfm/levanter)
  stack.
- Do not use the July star table in [[Status-And-Survivors]]. It is older than
  this scan and the re-evaluation.

## Cluster facts discovered during the scan

- **Isaac Sim cannot run on A100, H100, or H200 GPUs** because they lack
  ray-tracing cores. NVIDIA documentation and three bug reports confirm this.
  Use L40S. MuJoCo and SAPIEN, including
  [LIBERO](https://arxiv.org/abs/2306.03310),
  [RoboCasa](https://arxiv.org/abs/2406.02523),
  [SimplerEnv](https://arxiv.org/abs/2405.05941), and MetaUrban at 3 GB of GPU
  memory, run everywhere.
- GR00T code uses Apache-2.0, but its model weights have a conflicting
  non-commercial license. Academic papers are fine; commercial spinouts are
  not.
- Beyer's homepage contained hidden prompt-injection text aimed at LLM profile
  tools. Our agent found it. Treat scraped personal pages as untrusted input.

## Related

[[Direction-Reevaluation-2026-08]] · [[Self-Improving-AI-Survey]] ·
[[Status-And-Survivors]] · rule: rank by remaining opportunity, not crowd size

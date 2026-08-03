# Method ideas, with baselines and scores to beat

Status: **Written 2026-07-26; partly replaced by newer work.** *Updated
2026-08-02 for the general research wiki.* This page was written after the owner
correctly noticed that every idea that had survived the earlier review was a
measurement project.

> **Newer-ranking note (2026-08-02).** This page was written before the
> eight-direction second review in [[Direction-Reevaluation-2026-08]] and the
> [[Top-Researcher-Scan-2026-08]]. The second review is historical, and the
> researcher scan is another dated input. For the current ranking, use
> [[Unified-Direction-Ranking-2026-08]]. The verdicts from
> the 2026-08-02 review were: **T1 ★★★★½** (confirmed, but changed to use three
> kinds of training goals; Darrell's group became the main risk of publishing
> first), **T3 ★★★★½** (raised), **T2 ★★★★** (kept, but changed into an audit
> across many methods), **T4 ★★** (lowered because three papers had already done
> the main idea), and **KV-cache ★★★★ only if narrowed to safety-aware
> allocation** (★★★ for the broader version on this page). The old text remains
> below as a dated record. A dated note marks each place where the newer review
> disagreed with it.

## Why I had favored measurement projects

**The rule I used was built to favor measurement.** Asking "Has anyone done
exactly this?" makes sense for an audit, which checks existing work. It is the
wrong question for a new method. A method paper is not a bad idea just because
someone else has worked on the same problem. It is a bad idea if the new method
cannot beat what already exists. If we reject every method whose topic has prior
work, we reject every method idea.

I also read too much into the SVIB failure. I treated it as proof that this group
could not run a successful method project. That was not the lesson. SVIB failed
for two clear reasons: its baseline was not valid, and tests could remove its
main mechanism without hurting the result. The scale of the work was not the
problem. C2LIP used 8xA40 on CC3M. [CLIC](https://arxiv.org/abs/2505.24424) used
about ~1M samples and trained only the text encoder. Both are real method papers
done with this amount of computing power.

**For a method, ask: where is the current best method WEAK, and what change
could fix that weakness?** Each idea below gives a baseline and a score to beat.

---

# Group 1 — scoring how well a VLM combines ideas *(closed; shortened 2026-08-02)*

**This section is shortened for the general wiki.** A VLM is a vision-language
model. Every test in this group used SVIB's frozen dual encoder, a model with one
encoder for images and one for text whose weights do not change. This group is
now closed.

The plan had four tests. P1 adjusted scores without learning from the test set.
P2 used exclusive OT assignment, which means each item could be matched only as
allowed by an optimal-transport matching rule. P3 would have learned a fast
version of structured inference. P4 would have represented image regions using
given boxes. We ran P1 and P2. Both failed. When the protocol chose its settings
only from validation data, it selected `alpha = 1.0` for both. That is just the
original global score. P3 and P4 were never run. We withdrew P4 because it works
against a mechanism that earlier work had already shown. For all baselines,
changes on each benchmark, confidence intervals (CIs), limits set by earlier
work, and locked protocol rules, see the **svib repo wiki** pages
*Cluster-1-Compositional-Scoring* and *Method-Opportunities*.

## Lessons from this group

- **Do not compare a method that cannot use test-set information with numbers
  from a method that can.** The first kind is called inductive. The second is
  called transductive. Test-Time Matching reports a frozen-SigLIP jump on
  [Winoground](https://arxiv.org/abs/2204.03162) from `10.25 → 67.00`. But it
  needs the test-set group split. It also changes what counts as success. Four
  separate inequalities with `16.7%` random chance become one shared assignment
  with `50%` random chance. This is not a valid baseline for an inductive method.
  It is also not real "available headroom." Catch a hidden metric change before
  running an experiment.
- **Check whether a headline score came from the no-training version.** The
  `73.0 → 86.3` [SugarCrepe](https://arxiv.org/abs/2306.14610) result that led to
  this group came from a *trained* cross-modal module with 13.3M parameters. The
  no-training method that was given credit for the result was never tested on
  SugarCrepe, [BiVLC](https://arxiv.org/abs/2406.09952), ARO, or Winoground. We
  planned a whole group of work after reading an abstract instead of the table
  of evaluation results.
- **Do the math before paying for computing time.** Separate bias corrections
  for each image and each caption cancel *exactly* in the margin for a shared
  assignment. That made P1's main target impossible by algebra. We could have
  proved this on paper in an hour, before trying to explain any experimental
  result.
- **A score near random chance across the whole field is a warning sign, not
  automatically an opening.** Every end-to-end method scores `1.2–1.9` on a
  controlled-swap binding test where random chance is `16.7`. Pipelines that
  crop image regions solve color (`95.7`) but ruin scale (`20.2` compared with
  `16.7` chance) because of how they are built. A proposed change that fights a
  known mechanism is likely to fail. That is why we dropped P4 instead of
  running it.
- **A huge cost increase can itself be worth studying.** Structured inference
  over image regions costs 133–210× as much as one pass through the base
  encoder. Its gains appeared only for T2I, meaning text-to-image matching. It
  made I2T, or image-to-text matching, worse everywhere, and no one explained
  why. A faster version could use ROI-Align from one dense encoder pass. That
  faster method, together with the speed table that papers do not publish, is
  still a fair and unclaimed engineering contribution for someone working in
  this area.

---

# Group 2 — give KV-cache bits to the right goal

> **Newer note (2026-08-02).** The historical review in
> [[Direction-Reevaluation-2026-08]] gave this group **★★★★ only if narrowed to
> safety-aware allocation**, and **★★★ as scoped below**. The current combined
> ranking is in [[Unified-Direction-Ranking-2026-08]]. M1 below, about long
> rollouts, is no longer open. CONF-KV
> ([`2605.24786`](https://arxiv.org/abs/2605.24786)) now covers it. Three papers
> already offer fixes for errors that grow over time: [SQuat](https://arxiv.org/abs/2503.24358),
> KVarN, and [VeriCache](https://arxiv.org/abs/2605.17613). **M2** still survives.
> Six focused searches found no allocator with a safety goal. KVFundaBench v2
> removed safety from its abstract. CAQ
> ([`2511.07842`](https://arxiv.org/abs/2511.07842)) shows that a paper about
> choosing the wrong goal can be published for weight PTQ, which means
> post-training quantization. The single test that can clearly prove the idea
> wrong is to compare the best per-layer bit allocation for safety with the best
> one for perplexity. Perplexity measures how surprised a language model is by
> text. If the two allocations match, the idea dies cheaply. If they differ, we
> get a new map of the problem and a new allocator. Before the test, name
> [`2605.18053`](https://arxiv.org/abs/2605.18053), where protection beats
> scoring, as the control. The historical review also disproved a common claim
> that this work needs custom CUDA kernels. [KVTuner](https://arxiv.org/abs/2502.04420)
> (ICML'25), [EvolKV](https://arxiv.org/abs/2509.08315) (EMNLP), and
> [SCBench](https://arxiv.org/abs/2412.10319) (ICLR'25) were published with zero
> kernel work. The real requirement is a level of detail that works in a
> serving system, plus speed tests on existing kernels. A related idea is M6 in
> [[Top-Researcher-Scan-2026-08]]: remove KV entries based on the current turn
> when an agent has a conversation. That idea joins the safety-aware allocator.

**Someone published my first idea, but the replacement is better.** Giving
different bit counts to each KV-cache head or layer is not unused. **RateQuant**
([`2605.06675`](https://arxiv.org/abs/2605.06675)) uses a formula called reverse
waterfilling. At 2.5 bits on Qwen3-8B, it improves [KIVI](https://arxiv.org/abs/2402.02750)
from `49.3 → 14.9` PPL. PPL means perplexity. Lower is better. **RDKV**
([`2605.08317`](https://arxiv.org/abs/2605.08317)) uses waterfilling across
tokens and channels. Other existing systems are KVTuner,
[KVmix](https://arxiv.org/abs/2506.08018),
[SpectrumKV](https://arxiv.org/abs/2606.08635),
[MixKVQ](https://arxiv.org/abs/2512.19206), and
[MoQAE](https://arxiv.org/abs/2506.07533). A direct copy of the WaterSIC idea
would fail before it starts.

**However, all of these methods optimize a fixed stand-in score for damage at
one step.** Two separate groups of papers show why that is the weak point. They
do not connect the problem back to the bit allocator:

- **Errors grow during a rollout.** A rollout is a sequence of model steps.
  KVarN ([`2606.03458`](https://arxiv.org/abs/2606.03458)) says that
  *"quantization errors accumulate across timesteps."* Quantization stores
  values with fewer bits. KVarN fixes how the values are represented, using
  Hadamard plus variance normalization. It does **not** fix how bits are given
  to different parts of the cache.
- **A small ability-carrying part of the model can collapse.**
  [`2606.09864`](https://arxiv.org/abs/2606.09864) finds that Mistral-7B loses
  **15.2% of refusals at 1.03x perplexity**. Safety information sits in a
  subspace, meaning a smaller set of directions inside the full model space,
  that is **10^2-10^3x more vulnerable** than the full space. The paper says its
  method *"succeeds where attention-based allocation approaches fail."*

This is published evidence that the current allocation goal is broken. It comes
from researchers who did not repair the allocator itself.

**M1 — allocation that accounts for a rollout.** Keep reverse waterfilling.
Replace its damage model with expected KL drift over an H-step rollout. KL drift
measures how much the model's output distribution changes. Estimate it by
quantizing one head and then running the rollout. Because errors build up, a
flat rate-versus-damage curve becomes weighted by model depth. **Score to beat:**
RateQuant at `14.9` PPL with 2.5 bits, and KVarN on MATH500/AIME24 with 2 bits.
The win rule is to match PPL and improve MATH500 by >3pp. This is plausible
because PPL is the exact measure that hides the problem. Cost: **2 GPUs.**

**M2 — allocation that protects an ability.** Use pairs of opposite prompts and
their difference in average behavior to estimate a low-rank capability
subspace. Low-rank means that only a small number of directions matter. Run
waterfilling on damage **projected onto that subspace**, so it pays attention to
those directions. One Lagrangian setting, which controls a trade-off, gives a
full trade-off frontier instead of one result. **Score to beat:** PCR's best
post-hoc repair setting, but without the extra repair pass. Cost: **4 GPUs.**

**M3 — an end-to-end goal split into Kronecker factors.** Move the sketch from
[YAQA](https://arxiv.org/abs/2505.22988) to the KV cache. A Kronecker factor is a
way to split one large calculation into smaller structured parts. The cache has
a head x channel structure where this split is *more* natural than it is for
weights. The cleanest ablation, or controlled removal test, changes only the
damage measure while keeping the allocator fixed.

**M1 + M2 + M3 can form one paper:** *KV-cache bit allocation has been solved
for the wrong goal.*

---

# Old recommendation

> **Replaced on 2026-08-02.** Group 1 is closed because every local branch was
> rejected. Group 2 survives only when narrowed to safety-aware allocation. The
> historical reasoning is in [[Direction-Reevaluation-2026-08]]. The current
> ranking is in [[Unified-Direction-Ranking-2026-08]]. The old recommendation
> below remains as a record.

The old first choice was **Group 1**. Its code and data system was already built
and checked. A separate ICLR paper showed room to improve, and P1 would take
only hours. The old second choice was **Group 2** if the goal was to leave VLM
research. It had a clearer "beat this number" story, but more researchers and
large companies were already working there.

Neither idea had yet passed a search of earlier work. The record shows that such
a search must happen before we commit. For a **method**, though, the test should
be "is there a change that can fix a known weakness?" It should not use the
**audit** test, "has no one ever done this?"

**The lasting rule is simple.** For a *method*, ask "where is the current best
weak, and what change could fix it?" Do not ask only "has anyone worked on
this?" Rejecting every method with related past work rejects all method ideas.
For an *audit*, checking whether the exact audit already exists is correct. Use
the rule that fits the kind of paper.

## Related

[[Live-Research-Opportunities]] — ideas focused on measurement.
[[Unified-Direction-Ranking-2026-08]] — the current combined star ranking.
[[Direction-Reevaluation-2026-08]] — a historical second review that replaced
the first ranking on this page.
[[Top-Researcher-Scan-2026-08]] — method ideas M1–M7 found by studying leading
researchers.

---

# Finding for the whole program: stop adding local branches *(shortened 2026-08-02)*

**This is a short version of the SVIB evidence.** We tested six different ways
to add local evidence to a frozen dual encoder: SVIB graph+VIB, patch-grid nodes,
claim-level caption decomposition, conformal dispersion, P1 marginal
calibration, and P2 exclusive OT assignment. Each one used a different
mechanism. When the operating point was chosen only with validation data, all
six lost to the plain global score. For every test's exact numbers, intervals,
and provenance hashes that prove where the data came from, see the **svib repo
wiki** pages *Cluster-1-Compositional-Scoring* and
*Post-Rebuttal-Measurement-Sprint*.

**This section stays because of three broad lessons:**

1. **Name the mechanism measure before the experiment, not just the final
   score.** The strongest test predicted the right direction: the expected
   collisions happened. But the gap caused by the mechanism was only `0.00015`,
   while the pre-registered pass mark was `0.02`. The mechanism was right, but
   its size was wrong by two orders of magnitude. That tells us much more than
   a result that only says "no effect." We know this only because the protocol
   named the mechanism measure before the run. Every pre-registration should do
   this.
2. **Choosing settings only on validation data can itself be a contribution.**
   Papers in this method family often report gains after choosing settings on
   the test set or separately for each benchmark. The same methods can be
   rejected completely when the setting is locked on a development split: the
   mixing weight moves to the trivial value. Showing this with our own runs
   makes the point without accusing or auditing anyone. A reviewer does not
   need to be told that they were wrong.
3. **When N meaningfully different branches all reject one representation,
   stop testing that representation and change areas.** A set of pre-registered
   negative results on one question has real value. TMLR clearly allows this
   kind of paper, and so does ICBINB. Turning those results into a paper mainly
   needs writing, not more computing. But the *next* project should move to a
   new area instead of adding a seventh branch. This is especially true when
   the last possible change fights a mechanism that past work already showed.

> **Note (2026-08-02).** The ratings for both suggestions to "move areas" later
> changed. Group 2, on the KV cache, is ★★★★ only when narrowed to safety-aware
> allocation. B1, which would isolate diversity collapse, was **lowered to ★★★ —
> scooped in April 2026** by
> [`2604.16027`](https://arxiv.org/abs/2604.16027). That paper follows all three
> Olmo 3 model lineages. B2, about the appearance of visual attention sinks, did
> not receive a second review. See the historical review in
> [[Direction-Reevaluation-2026-08]], the current ranking in
> [[Unified-Direction-Ranking-2026-08]], and
> [[Live-Research-Opportunities]].

---

# Methods that train a model (2026-07-26) — fixing a second mistaken limit

**My mistake.** I treated "we cannot afford full pretraining" as "we cannot
train anything." Those are very different claims. The difference is about one
order of magnitude. These are confirmed example costs on the same type of
hardware:

| System | Cost | Outcome |
|---|---|---|
| **Prismatic VLM, full 7B run** | **8x A100, under 9 hours** | ICML 2024 |
| C2LIP | 8x A40, CC3M, 5 epochs, batch 768 | CVPR 2026 |
| [Open-Qwen2VL](https://arxiv.org/abs/2504.00595) | 220 A100-40G GPU-h pretrain + 48 SFT | beats Qwen2-VL-2B |
| [DIVA](https://arxiv.org/abs/2407.20171) | 66.4 GPU-hours total | — |
| Perception-R1 | 16x A100, ~16h (~256 GPU-h) | — |
| [DINORankCLIP](https://arxiv.org/abs/2605.06592) | 8x H100, 72h for a *full ablation* | May 2026 |

A 1B model trained on 6.25B tokens takes about ~80 H100-hours. That is ten hours
on eight GPUs. We can afford full fine-tuning of a 7-8B model, contrastive
training at CC3M scale, VLM instruction tuning, and small pretraining runs. All
of these are **in scope**.

## T1 — test freezing x training goal x training stage *(top pick of the session)*

> **Update 2026-08-02 — confirmed ★★★★½, with a changed design.** The historical
> second review in [[Direction-Reevaluation-2026-08]] checked T1 again and kept
> it at HIGH density. For the current combined ranking, see
> [[Unified-Direction-Ranking-2026-08]]. CoVFT
> ([`2603.21077`](https://arxiv.org/abs/2603.21077)) says the choice between
> freezing and fine-tuning "remains unresolved." Its own benchmark uses only
> SFT, or supervised fine-tuning, and VFT wins on 6/12 tests. No survey combines
> the evidence about VLM training recipes. No one owns the 7B model-size band.
> **Design change:** use three training goals: **SFT / RL / SFT + perceptual
> auxiliary (VIRAL-style)**. A perceptual auxiliary is an extra training goal
> that rewards better visual features. This change is needed because PIVOT
> already covers {unfrozen} × {SFT, DPO}. The more exact question is: *does
> freezing help in one direction but hurt under a goal that is not language
> generation?* **The risk of being scooped is now clear.** The CoVFT group could
> add an RL arm, and we should use their public test system. According to
> [[Top-Researcher-Scan-2026-08]], **Darrell's group is the main risk**. Before
> starting T1, immediately check it against his C1 group of work: CoVFT plus the
> four places where encoder fixes have not yet been compared fairly.

**There is an active three-way disagreement about a basic VLM training choice.**

**Prismatic** ([`2402.07865`](https://arxiv.org/abs/2402.07865), ICML 2024) makes
two direct claims. First, *"including the explicit projector pretraining stage
is unnecessary, with single-stage training improving aggregate performance"*.
Skipping that stage saves 20-25% of the cost. Second, *"finetuning the visual
backbone significantly degrades performance, especially on tasks requiring
fine-grained spatial reasoning"*. Its scores fall from VQAv2 `77.09 -> 73.53`,
TextVQA `44.45 -> 38.33`, and GQA `62.57 -> 59.65`.

**Other papers disagree in both directions:**

| Claim | Supports | Contradicts |
|---|---|---|
| Alignment stage unnecessary | Prismatic, [Molmo](https://arxiv.org/abs/2409.17146) | [Eagle](https://arxiv.org/abs/2408.15998) (pre-align helps *atop* unfreezing, `662.5 -> 672.3`), [MM1.5](https://arxiv.org/abs/2409.20566), [LLaVA-OV-1.5](https://arxiv.org/abs/2509.23661) |
| Unfreeze the ViT | [Cambrian-1](https://arxiv.org/abs/2406.16860) (*"benefits performance across all benchmarks"*), Eagle (`616.5 -> 674.2`), [InternVL3](https://arxiv.org/abs/2504.10479) (*"trains every layer jointly"*) | Prismatic, [NVLM-1.0](https://arxiv.org/abs/2409.11402) (InternViT-6B frozen through *every* stage at 72B) |

**Prismatic also suggests a reason for the failure. No one has tested it:**

> *"The degraded performance from full finetuning could be for a number of
> reasons ranging from the scale and diversity of the vision-language data we
> train on to **language generation as a learning objective (vs. objectives that
> encourage learning fine-grained perceptual features)**."*

**PIVOT** ([`2510.16333`](https://arxiv.org/abs/2510.16333)) gives exactly the
missing comparison. Its RL training *"produces stronger and precisely localized
visual representations"* while costing under 1% of vision-pretraining cost.

**The original experiment:** compare every combination in `{frozen, unfrozen} x
{SFT, RL/preference} x {one-stage, two-stage}` on a 7B model. Use the same data
and token budget for all groups. Cost: **~300-600 GPU-hours.** This exact design
is kept as history. The 2026-08-02 update above replaces its training-goal part
with SFT / RL / SFT + perceptual auxiliary.

This was the best idea in the survey for five reasons. It settles a **cited,
active, three-way disagreement**. It directly tests the reason named by the
original authors. It **includes and replaces** the earlier B2 proposal about
freezing. The reference run takes **nine hours**. Finally, any result would be
useful enough to publish.

## T2 — RL may improve answers without improving vision

> **Update 2026-08-02 — kept at ★★★★, but changed into a vision audit across
> methods.** Part of the first idea was already published.
> [`2602.12395`](https://arxiv.org/abs/2602.12395) supported Perception-R1's null
> result by studying the mechanism. [`2603.01301`](https://arxiv.org/abs/2603.01301)
> separated sharpening effects in medical models. On 2026-07-30, another paper
> showed that the PSR estimator was broken:
> [`2607.28336`](https://arxiv.org/abs/2607.28336) says it "conflates perceptual
> insufficiency with reasoning difficulty". **The open part is still large and
> unclaimed.** About 50 RL methods say they target vision. Three tests from 2026
> show that their gains remain even when the image is masked or damaged
> ([`2605.09266`](https://arxiv.org/abs/2605.09266),
> [`2604.03179`](https://arxiv.org/abs/2604.03179)). Yet *no one has run these
> controls on the methods that claim to solve the problem*. No group owns this
> diagnostic area. Those papers have 3 / 0 / ~0 citations. Most tests need only
> inference on public model weights. The control groups need small 3B–7B GRPO
> runs. The cost is ~256 GPU-h, and the work can be ready for ICLR-2027. See the
> historical analysis in [[Direction-Reevaluation-2026-08]] and the current
> ranking in [[Unified-Direction-Ranking-2026-08]].

**Perception-R1** ([`2506.07218`](https://arxiv.org/abs/2506.07218)) uses
McNemar's statistical test. It finds **no statistically significant improvement
in visual perception** from normal RLVR (`p = 0.22-0.69`), even though the main
accuracy score rises. RL makes answers that were already hidden in the model
more likely. It does not repair vision. **PAPO**
([`2507.06448`](https://arxiv.org/abs/2507.06448)) supports the same story from
the other side. Under normal GRPO, **67% of errors are failures to see the image
correctly**. PAPO's fix cuts those failures by 30.5%.

Other RL failures also have measured numbers. In
[VLM-R1](https://arxiv.org/abs/2504.07615), models game the mAP reward by
printing many repeated boxes. [MM-Eureka](https://arxiv.org/abs/2503.07365)
reports a *"sudden training collapse"* at 32B, where the reward reaches zero. It
also finds that RL makes it *"difficult for the model to acquire new knowledge —
improvements come from increasing the probability that the model generates
correct answers."*

The leading papers published the null result. The tests use statistics, and this
program has spent months building exactly that kind of careful process. Cost:
**~256 GPU-h.**

## T3 — mix training data while keeping compute fixed (largest measured effects)

> **Update 2026-08-02 — raised from ★★★ → ★★★★½.** The area only looked full
> because most earlier work studied text alone. DataComp-VLM cites 8 mixture
> methods. Of these, 7 use text only, and the 8th changes data during SFT.
> DataComp-VLM directly says that "there exists no systematic study on filtering
> and mixing strategies in the VLM setting". Its 347 references contain zero
> surveys. It also has 13 LaTeX labels that were never resolved, including one
> for the promised appendix about mixtures across several axes. That analysis
> was not written. **There is a live disagreement to settle.** Shukor
> [`2507.09404`](https://arxiv.org/abs/2507.09404) says mixture scaling laws can
> predict larger runs. DataComp-VLM measures a reversal in the ranking: a
> caption-heavy mix wins at 1B×6.25B, but an instruction-heavy mix wins at
> 2B/4B×25B+. Neither paper cites the other for this issue. Public checkpoints
> at four sizes reduce the cost from about ~25,000 H100-h to about ~500.
> **The clearer question is:** can we *predict* when the best mixture changes
> between a small and large model without paying for all large runs? If not,
> mixture searches that use small models as stand-ins are not valid. Watch for
> the consortium to turn this into a competition. See the historical review in
> [[Direction-Reevaluation-2026-08]] and the current ranking in
> [[Unified-Direction-Ranking-2026-08]].

**Filtering looks dead, but mixing is alive.** DataComp-VLM
([`2606.28551`](https://arxiv.org/abs/2606.28551)) says: *"no quality filter we
tested produces a robust and significant improvement"*. Its best filter adds
only `+0.8pp`. But a **mixture** with 70% instruction-tuning data and 10%
caption data gives **`+5.4pp`**. It also finds that *"a 4B model trained for 100B
tokens beats an 8B model trained on
[FineVision](https://arxiv.org/abs/2510.17269) for 200B tokens."*

**20/20 VLM** ([`2605.11405`](https://arxiv.org/abs/2605.11405)) changes only
which data it uses while keeping compute fixed. With 25B tokens, 2B parameters,
and one training stage, curation gives **`+11.7pp`** across 20 benchmarks and
**`+57.1pp` on grounding**. Grounding connects words to image locations. The
result matches [InternVL3.5-2B](https://arxiv.org/abs/2508.18265) while using
about ~17x less compute.

Other null results support the warning. [MM1.5](https://arxiv.org/abs/2409.20566)
*"did not find conclusive evidence that high-quality synthetic captions improved
performance over the arguably simpler OCR data"*. Also,
[`2405.11850`](https://arxiv.org/abs/2405.11850) reports that
[SEED-Bench](https://arxiv.org/abs/2307.16125) **drops 3.3 points** when
pretraining data grows from 20M to 100M. A fixed-compute mixture study at 2-4B
would cost **~400 GPU-h.**

## Other findings

**The connector design usually has almost no effect.** A connector joins the
vision encoder to the language model. MM1 says *"the vision-language connector
design is of comparatively negligible importance"*. [Eagle](https://arxiv.org/abs/2408.15998)
finds that simply joining channels (`690.4`) beats deformable attention
(`674.3`). There is one strong exception. [Cambrian-1](https://arxiv.org/abs/2406.16860)'s
Perceiver resampler collapses on OCR&Chart, scoring `27.1` instead of `55.5`.
**However, none of these removal tests measured transfer of combined ideas.**
C2LIP's encoder gain falls from `+6.8` to `+0.4` after passing through the
connector. [CLIC](https://arxiv.org/abs/2505.24424) says *"a detailed study of
this is left for future work"*. That question remains open.

**Fact that still needs checking:** Prismatic's p-values are in figure captions
that do not appear in the arXiv HTML. Two separate readings found `0.00381` and
`0.00407`. **Check the PDF before quoting either number.**

**Other groups may publish first:** [DINORankCLIP](https://arxiv.org/abs/2605.06592)
from May 2026 uses the same class of hardware and studies training goals.
[Bottleneck Tokens](https://arxiv.org/abs/2604.11095) and
[MoCa](https://arxiv.org/abs/2506.23115) work on embeddings, where the leading
method changes about every two months. [CABS](https://arxiv.org/abs/2511.20643),
a CVPR 2026 paper, controls the data with concept labels.

## T4 — choose data for the learning-rate decay window *(strongest paper; three searches agreed)*

> **LOWERED ON 2026-08-02: ★★★★★ → ★★. Do not start this project.** The text
> below stays as a record. Its main claim, "there is no method paper for
> anneal-window data selection," was already **false when it was written**. The
> historical second review in [[Direction-Reevaluation-2026-08]] found three
> papers that had already taken the idea. For the current combined ranking, see
> [[Unified-Direction-Ranking-2026-08]]. **DiReCT**
> ([`2605.31175`](https://arxiv.org/abs/2605.31175), 29 May 2026) contains almost
> our exact reason for the project: "effectively selecting training data during
> this phase remains a key challenge... lack a principled grounding". It tests
> Llama-3-8B/300B and includes both theory and code. **QAFSL**
> ([`2605.25698`](https://arxiv.org/abs/2605.25698)) already owns the claim that
> "decay reduces update intensity exactly when high-quality data becomes
> available". It beats WSD by +1.70 at 15B-MoE. **MIRA**
> ([`2605.30288`](https://arxiv.org/abs/2605.30288)) already owns the idea that
> choosing data during middle training is a separate problem. DiReCT had been
> public for ~8 weeks when the July search called the area empty.
>
> There are more problems. A new paper can use our small-scale design as
> evidence *against* us. [`2606.07597`](https://arxiv.org/abs/2606.07597) finds
> that predicting from forked decay runs "frequently fails" when high-quality
> data repeats. That is our exact protocol. The object being studied may even
> disappear: WSM, WSO, and [`2604.13627`](https://arxiv.org/abs/2604.13627)
> separately reach the answer that training should use less decay or no decay.
> The small field that links data choices to the schedule has **zero top-venue
> acceptances**. Finally,
> [Compute-Constrained Data Selection](https://arxiv.org/abs/2410.16208)
> (ICLR'25) shows that selectors based on gradients, which are T4's main tool,
> are almost never the best use of compute. Any version that survives must show
> a baseline curve that counts the selector's own cost.
>
> **One cheap, unclaimed, mechanism-focused question remains.** Does the ranking
> of each document's value *change order* between the stable learning rate and
> the decay learning rate? Measure rank correlation after one shared training
> trunk. This separates the "good data is being wasted" story from the "decay
> creates a sharper solution" story. A null result would weaken all three papers
> that published the main idea first.
>
> **The process lesson, recorded in the historical
> [[Direction-Reevaluation-2026-08]], is this:** before treating an empty area as
> a good sign, explain *why* it is empty. Also repeat any earlier-work search
> older than ~6 weeks before spending compute.

**The gap we thought we found.** Searches on arXiv for `"annealing data"`,
`"decay phase"`, `"cooldown phase"`, and `"annealing phase"` in cs.CL during
2025-2026 found **almost nothing** except the system report for
[OLMo 2](https://arxiv.org/abs/2501.00656). Papers studied **the optimizer side
of cooldown** ([`2508.01483`](https://arxiv.org/abs/2508.01483), WSM
[`2507.17634`](https://arxiv.org/abs/2507.17634), and
[`2603.16127`](https://arxiv.org/abs/2603.16127)). We incorrectly concluded that
**the data side had not been studied** and that no method paper existed for
choosing data during the anneal window.

The old conclusion was that a common rule used by leading labs **had no method
behind it**. MiniCPM, OLMo 2, and [Llama 3](https://arxiv.org/abs/2407.21783) all
put large amounts of high-quality data into the decay phase. The downgrade above
shows why the conclusion was wrong.

**The measured effect was the largest in either survey.**
[PRISM](https://arxiv.org/abs/2603.17074) finds that data chosen during middle
training later gives **`+17` to `+28` on GPQA-Diamond during RL**. Changing the
RL data mixture gives **under 2 points**. Middle training changes over 90% of
the weights, while RL changes about 5%. Researchers spend much more effort on
RL recipes even though the data entering the anneal window appears to matter
about one order of magnitude more.

**The proposed change was exact.** Estimate the added value of each document
**at the decay-phase learning rate**, not at the highest learning rate, or LR.
The LR enters the influence estimate in a straight linear way, but current work
ignores it.

**Scores to beat:** use the Qwen2.5-1.5B architecture, 30B
[DCLM-Baseline](https://arxiv.org/abs/2406.11794) data, and the average over the
main MMLU/ARC/CSQA scores from
[`2511.18903`](https://arxiv.org/abs/2511.18903). The baselines are WSD+uniform
`46.21`, WSD+ascending `45.45`, EMA+ascending `46.95`, and ConstLR+ascending
`47.02`. Notice that this work **changed the schedule to fit the data**. Its best
ending LR was `1e-3` for a curriculum but `1e-5` for uniform data. Our idea was
to **change the data to fit the schedule**. The paper's own future-work section
asks for exactly this: *"a more systematic recipe for the strategy
combinations."*

**The experiment could also have settled an active disagreement.**
[`2603.16127`](https://arxiv.org/abs/2603.16127) studies Warmup-Stable-Only at 1B
and 8B. It finds that **no-decay always beats decay after SFT**, even when decay
has better pretraining loss. Its explanation is that decay creates sharper
minima, meaning solutions that get worse quickly after a small change. This
directly conflicts with [`2511.18903`](https://arxiv.org/abs/2511.18903).
Controlling *which data enters the window* could settle the disagreement.

**Timing cannot be ignored, and a paper has measured it.**
[`2510.14865`](https://arxiv.org/abs/2510.14865) studies
[Pythia](https://arxiv.org/abs/2304.01373) models from 70M-1B using 128B C4
tokens. Giving code **80% of the data weight at 12B tokens works well. Giving the
same 80% at 105B tokens hurts more than giving only 10%**. If data first appears
after the model's plasticity window, or time when it can easily change, adding
more of that data later cannot make up for the lost chance.

**The design would keep the cost reasonable and the statistics fair.** WSD lets
all groups share one trunk. First, **train the stable phase once**. At 1B with
20B tokens, this costs about 256 H100-hours. Then **split into K separate decay
phases**, each with 5B tokens and about 64 hours. Twelve experiment groups cost
about **1,020 H100-hours**. Because every group starts from the same trunk,
**every comparison is paired.** This design may be worth more than the method
itself. RegMix-D openly lists **single-seed target runs** as a weakness.
[PolyPythias](https://arxiv.org/abs/2503.09543), with 45 runs, 9 seeds x 5 model
sizes, shows that random-seed effects are real.

## How crowded each nearby area looked in the same search

> **Use these as evidence about how much of each problem is solved, not just as
> paper counts (2026-08-02).** The historical
> [[Direction-Reevaluation-2026-08]] corrected the rule behind this list. A hot
> topic should not be removed only because many people study it. Remove it only
> if little useful work remains. The current combined ranking is in
> [[Unified-Direction-Ranking-2026-08]]. The details below are still evidence:
> [Aioli](https://arxiv.org/abs/2411.05735)'s null result, the 22 mixture methods,
> and the gap about when repeated data appears. But labels like "SATURATED" and
> "cleanest hole" are not trustworthy if they come only from paper counts. The
> anneal/decay label was especially wrong. See the T4 downgrade above.

- **Optimizing data mixtures: SATURATED, so do not attack it directly.** There
  were at least 22 methods from Jan 2025 through Jul 2026, **plus two surveys**.
  In Nov 2024, Aioli showed that **no existing method always beats stratified
  sampling**. Some methods make test perplexity up to `6.9` *worse*. The field
  answered this null result by making even more mixture methods.
- **Curriculum and ordering: only a few papers, with no agreement.** Three
  serious pretraining papers study it, and they disagree.
- **Anneal or decay window: appeared to be the cleanest gap.** The search found
  one nearby paper from one lab that was eight months old. The downgrade note
  explains why this claim was false.
- **Repetition or replay: not crowded.** In particular, **no one studies *when*
  repeated data should appear**. [`2605.12715`](https://arxiv.org/abs/2605.12715)
  runs more than 2,000 experiments. It finds that mixture training can handle
  **15-20 repeats** of a rare dataset. That is much higher than the rule of 4
  epochs when using one data source.
- **Trap:** do not build a 1B project around predicting several tokens at once.
  Meta reports `+12%` on HumanEval at **13B** and directly says the method is
  *"increasingly useful for larger model sizes."*

## Two rules for every training study on this page

1. **Plan and pay for two model sizes in every study.** The order of methods can
   reverse as size changes. This is now documented for algorithms used after
   training, optimizers, **and** data mixtures.
   [DataDecide](https://arxiv.org/abs/2504.11393) trained 1,050 models across 25
   datasets x 14 sizes x 3 random seeds. The ranking at 150M picks the best 1B
   dataset in only **~80%** of pairwise comparisons. Also, **none of eight
   scaling-law baselines beats the simple guess from one small model.** Apple's
   combined mixture law proves why. Its cross-derivative is not zero, so the
   mathematically best mixture weights must change with model size.
2. **Never compare 1B models using only one random seed.**

## The old choice between T1 and T4

> **No longer a real choice after 2026-08-02 — T4 is dead (★★, scooped 3×).** We
> keep this comparison because its *reasoning* is still a good way to choose
> between two training studies. It also clearly shows how careful reasoning can
> still fail when it starts from an old, false claim that an area is empty. The
> historical second review changed the live comparison to T1 versus **T3**, both
> ★★★★½; see [[Direction-Reevaluation-2026-08]]. For the current ranking, use
> [[Unified-Direction-Ranking-2026-08]].

**T1 (VLM freezing x training goal x stage)** costs less, about ~300-600 GPU-h.
It is in an area where the needed system already exists, and it settles a
three-way disagreement. **T4 (the anneal window)** seemed to have the larger
effect, a truly empty area supported by three separate searches, and a cleaner
experiment. But it costs about ~1,000 H100-h and is farther from the group's
past work. The downgrade above shows that the "empty area" claim was wrong.

**The old conclusion was: T1 fits the group better, but T4 would make the
stronger paper.**

> **What the second review proved about this choice.** Saying that three separate
> searches confirmed the claim gave no real safety. All three searches used the
> same words. All three missed a paper that had been public for two months.
> Separate *searches* do not give separate *evidence* when they use the same
> method. Paper counts did not predict success either. Every July downgrade
> based on a crowded topic was later reversed: T3, AHD, self-improvement, and KV.
> Both ideas that received credit because their areas looked empty, T4 and B1,
> fell.

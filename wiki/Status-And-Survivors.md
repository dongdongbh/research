# What Happened to SVIB and Which Ideas Were Still Alive

**Purpose:** easy starting point, written on 2026-07-27. On 2026-08-02, Part
1 was shortened to keep only lessons that transfer to other projects.

> **The star ranking in Part 3 is old.** It records what we believed in July,
> not what to do now. An August 2 check changed six ideas. T4, about data used
> while the learning rate falls, dropped to ★★. B1, about why model answers
> lose variety, dropped to ★★★. T3, about multimodal data mixtures, rose to
> ★★★★½. T1, about which model parts to train and with what goal, moved to
> ★★★★½. Automatic heuristic design (AHD) rose to ★★★★. The KV-cache idea
> rose only after becoming more specific. That check also reversed the “do not
> enter self-improving AI” decision. It is recorded in
> [[Direction-Reevaluation-2026-08]]. Use
> [[Unified-Direction-Ranking-2026-08]] for the current ranking and
> [[Top-Researcher-Scan-2026-08]] for newer researcher-level ideas.

## Part 1: What happened to SVIB

The `svib` repository wiki holds the exact numbers, tests, and records of where
the data came from. This page keeps only the lessons we can reuse.

The submitted paper's main result came from a code error. The frozen baseline
applied the text projection twice. Fixing that bug reduced the claimed gain to
about one point. We found the error ourselves because the corrected baseline
matched published results, while the submitted one never did.

With equal training, none of the special parts—object proposals, sparse
one-way routing, or the variational bottleneck—caused the gain. A fixed grid of
image crops plus one normal self-attention layer matched or beat the full
system with far less extra work. The hand-written rule that chose when to use
the detailed system mostly turned on when the global scorer was already right.
Even a perfect choice rule had little room to help. Six more tests also lost
to the raw global score when all settings were chosen on validation data.

### Lessons to reuse

1. **Test the simple baseline before trusting the clever method.** Simple often
   wins. There is also an important opposite lesson: object proposals were not
   the costly part. Encoding every region again at full resolution was. Cheap
   one-pass replacements lost.
2. **At `alpha = 1`, a fused system must exactly match its backbone alone.**
   This easy check catches evaluator bugs. Also match QuickGELU with the right
   reference model. Using standard GELU can silently make a comparison wrong.
3. **A hand-written choice rule may turn on at the wrong times.** First measure
   the **oracle ceiling**, which is the best possible gain if the system always
   chose the right expert.
4. **A second valid caption can reverse benchmark conclusions.** Check whether
   the model ranking stays stable before trusting it.
5. **Choosing settings on the test set makes gains look too large.** When we
   chose only with validation data, the complex methods fell back to the
   simplest setting. The careful selection rule may be more useful than the
   failed method.

**Decision:** do not present SVIB as a new method. Write a controls-first study
of what actually causes compositional gains in frozen two-encoder VLMs. Target
TMLR or ICBINB and release the evaluator tools. The study checks our own work,
so it does not create a reviewer conflict. The experiments already exist; only
writing remains.

Full details are in `Post-Rebuttal-Measurement-Sprint` and
`Cluster-1-Compositional-Scoring` in the `svib` repository wiki.

## Part 2: Ideas that were still alive in July

### Strong ideas

**T1: Should the vision encoder stay frozen, and does the training goal change
the answer?**

Three respected papers disagree about a basic VLM training choice. One says
that updating the vision tower badly hurts detailed tasks. Two say updating it
helps. The first paper suggests a cause: perhaps next-word training is the
wrong goal for learning visual detail. Nobody had tested that guess. The plan
was to compare every combination of frozen versus trainable vision, language
loss versus reinforcement learning, and one training stage versus two. The
reference run takes nine hours.

**T4: Which data should a model see while its learning rate falls?**

Large labs often save their best data for the final “cooldown” part of
pretraining. At the time, we thought this was only tradition. Data used in this
window changed hard-reasoning scores by **17–28 points**, while changing the
later reinforcement-learning recipe changed them by **under 2 points**. A
cheap design could train one shared main run, then branch into several short
cooldown runs. This makes each comparison use the same starting point.

> **Outdated on 2026-08-02:** DiReCT had already been public for about eight
> weeks, so “no method paper” was false. Three papers covered T4, and one gave
> a small-scale version of our branch design that reviewers could cite against
> us. T4 fell to ★★. Only a small leftover question remains. See
> [[Direction-Reevaluation-2026-08]].

**SVIB write-up:** described in Part 1.

**T2: Does reinforcement learning truly improve what a model sees?**

Probably not. One paper used a proper statistical test and found no meaningful
gain in visual perception even when final scores rose. RL mainly made the model
more likely to give answers it already partly knew. Another paper found that
**67% of remaining errors came from perception**. The null result is already
published, but a follow-up would be cheap.

**B1: Does post-training reduce variety because of the method or the data?**

Fine-tuned models give less varied answers. The cause could be the training
method or the narrow training data. The authors said they had not separated
these causes. OLMo 3 releases weights and data at every stage, making a clean
test possible.

> **Outdated on 2026-08-02:** an April 2026 paper answered the exact question.
> B1 fell to ★★★. A smaller cause-and-effect crossover remains, but it costs
> more than we first estimated. See [[Direction-Reevaluation-2026-08]].

### Medium-strength ideas

**T3: Changing the data mix helps more than filtering data.**

A large study found that no quality filter helped reliably; the best gain was
`+0.8`. Changing the share of instruction data versus captions gave `+5.4`.
Careful data selection at equal compute gave `+11.7` overall and `+57.1` on
grounding. These are large effects. We first thought the field was too busy, so
the paper would need to focus on equal-compute comparisons instead of adding
another mixer.

> **Outdated on 2026-08-02:** the busy field was text-only. Multimodal data
> mixing was nearly empty. T3 rose to ★★★★½, and public checkpoints cut the
> expected cost by about 50×. See [[Direction-Reevaluation-2026-08]].

**A1: Were video-model benchmarks checked against people?**

Mostly no. One leading benchmark based its human-agreement claim on only four
models, too few for its test ever to find significance. Another calls its
judge human-aligned but gives no agreement score. A third benchmark released
102,000 human labels and its judge, so we could check it without making any new
videos.

**KV-cache allocation that protects the result we care about.**

Current methods decide how many bits to give each part of the attention cache
by minimizing one-step error. Two papers show different real harms: errors grow
through long generations, and safety behavior lives in a fragile direction
that loses 15% of refusals while perplexity barely changes. Neither paper
improves the allocator. The study needs 2–4 GPUs and no training.

**C1: Most LLM part-removal tests use the wrong statistics.**

Suppose a study tests three models on many prompts. The prompts are easy to
change, but the models are expensive, hard-to-change units. Treating every run
as independent makes model-level effects look more certain than they are.
Industrial statistics already has a standard fix, and we could release working
code for it.

**C2: Find test-data contamination with educational-test statistics.**

Psychometrics, the science of educational tests, can find questions that act
strangely for people with the same skill. A benchmark item learned during model
training has the same shape: it becomes unusually easy for a contaminated
model. Two separate searches rated this idea highly.

### Weak ideas

- **P3 and P4:** more work on the local branch of frozen two-encoder models.
  Six tests already failed there, and the official code release is still an
  empty placeholder. Do not continue.
- **A2 and A3:** follow-ups to A1. Run them only if A1 produces a result.
- **C3 and C4:** narrow uses of survival analysis for agent failures and
  queueing theory for agent fan-out. Both are real but small.
- **AHD ideas:** compare classical optimizers with LLM algorithm discovery at
  the same cost. The gap was real, but three control papers appeared in five
  months, leaving a 3–6 month window.

  > **Outdated on 2026-08-02:** claims were appearing about ten times faster
  > than controls, and nobody had measured the cost-matched boundary. AHD rose
  > to ★★★★. A second, still-open idea checks novelty claims. See
  > [[Direction-Reevaluation-2026-08]].

- **Weather:** credible work needs a meteorology collaborator.

## Part 3: Historical July ranking

> **This table is a record, not current advice.** It rewarded quiet research
> areas and punished busy ones. We now rank by how much useful work remains.
> Every row lowered only for being busy later rose, while both rows raised for
> being empty later fell. [[Direction-Reevaluation-2026-08]] records that
> August 2 change. Use [[Unified-Direction-Ranking-2026-08]] for current stars
> and [[Top-Researcher-Scan-2026-08]] for newer ideas.

The “Priority” column was the overall recommendation after considering fit,
cost, how many groups worked nearby, and the chance of producing a paper.

| # | Direction | Type | Nearby work | Compute | Good points | Bad points | July priority |
|---|---|---|---|---|---|---|---|
| 1 | **T1:** frozen parts × training goal × stage | Method | Medium | 300–600 GPU-h | Settles a live three-way disagreement; 9-hour reference run; tests a cause named by the original authors; every result is useful | Feels like settling a dispute; needs an exact equal-budget design | ★★★★★ |
| 2 | **T4:** data in the cooldown window | Method | **Very low** | about 1,000 H100-h | Largest effect found, +17–28; three searches seemed to find an empty area; shared-start design gives paired comparisons | Highest compute; furthest from our experience; needs two model sizes | ★★★★★ |
| 3 | **SVIB write-up** | Both | Low | **about 0** | Experiments are done; our own data avoids reviewer conflict; releases a reusable tool | Not a top venue; negative-result framing | ★★★★☆ |
| 4 | **T2:** separate RL gains from perception | Method | Medium | about 256 GPU-h | Builds on a published null; statistical and cheap | Extends another group's result; RL changes quickly | ★★★★ |
| 5 | **B1:** loss of diversity | Method | Low-medium | about 200–400 GPU-h | Could give a positive fix; authors named the missing design; OLMo 3 makes the cause testable | Had not been fully checked; a 2026 paper meant someone else might be doing it | ★★★★ |
| 6 | **T3:** data mixture at equal compute | Method | **High** | about 400 GPU-h | Large measured gains, +5.4 to +11.7 | We thought data-mixture work already had 22+ methods and 2 surveys | ★★★ |
| 7 | **KV allocation, M1/M2** | Method | High | 2–4 GPUs | Clear number to beat; two known failure types; no training | Strong industry groups; may need custom GPU code | ★★★ |
| 8 | **A1:** world-model judge check | Evaluation | **Very low** | under 100 GPU-h | Cheapest real result; one finding can be checked from a PDF now | An audit, and audits may become old before publication | ★★★ |
| 9 | **C1:** split-plot statistics for model part tests | Evaluation | Low | Very low | Correctness result; releases a method, not only a complaint | Short window because a competitor published two weeks earlier; still mainly measurement | ★★★ |
| 10 | **C2:** contamination through DIF | Evaluation | Low-medium | Medium | Two searches agreed; strongest evaluation use of GPUs | Had not been fully checked; nearby privacy work moves quickly | ★★★ |
| 11 | **A2/A3:** world-model follow-ups | Evaluation | Low | 1–3 weeks on 8 GPUs | Natural next steps after A1 | Make sense only if A1 works | ★★ |
| 12 | **AHD:** classical-optimizer comparison | Evaluation | Medium-high | Low | Real missing comparison on the standard benchmarks | Three control papers in five months; only a 3–6 month window | ★★ |
| 13 | **C3/C4:** frailty and fork-join statistics | Evaluation | Low | Low | Nobody else clearly doing them | Narrow and small audience | ★★ |
| 14 | **P3/P4:** more local-branch work | Method | Medium | Medium | — | Six failed probes; code release is a placeholder | ★ |
| 15 | **Weather:** twPCRPS/METAR | Evaluation | Medium | Very low | Almost no compute | Needs a domain expert; main claim already published | ★ |

## What the July page told us to do

The July choice was to write SVIB because it needed no new compute, and to
start T1 because its reference run took nine hours and addressed a real
disagreement. If T1 worked, T4 was supposed to be the larger next project.

> **Updated 2026-08-02:** writing SVIB and checking T1 still make sense, but T4
> is no longer the large next bet; it fell to ★★. T3 and the revived
> self-training-collapse question are now stronger second choices. T1 also
> needs a new check against Darrell's encoder-fix group. See
> [[Direction-Reevaluation-2026-08]] and
> [[Top-Researcher-Scan-2026-08]].

## Related

- [[Unified-Direction-Ranking-2026-08]] — current star ranking.
- [[Direction-Reevaluation-2026-08]] — historical ranking that first replaced
  Part 3.
- [[Top-Researcher-Scan-2026-08]] — current researcher-level openings.
- [[Method-Opportunities]] — full methods, baselines, and scores to beat.
- [[Live-Research-Opportunities]] — evaluation ideas.
- `Post-Rebuttal-Measurement-Sprint` and `Cluster-1-Compositional-Scoring` in
  the `svib` repository wiki — full SVIB evidence.

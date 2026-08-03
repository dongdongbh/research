# Self-Improving AI: What Works and What Is Still Open

> **The final choice on this page is outdated as of 2026-08-02.** This page
> originally said not to enter the field because many groups were already in
> it. The PI later corrected that rule: judge an idea by how much useful work
> remains, not simply by how busy the area is. A new check raised the
> self-training-collapse question in O1 from ★★ to ★★★★. The reason is in
> [[Direction-Reevaluation-2026-08]]. The current overall ranking is
> [[Unified-Direction-Ranking-2026-08]]. The map of the field and its failure
> modes below are still useful. They also explain why the higher rating is
> reasonable.

Updated for the shared research wiki on 2026-08-02.

**Survey date: 2026-07-27.** We searched arXiv, Semantic Scholar, OpenAlex,
DBLP, Crossref, and OpenReview for work from 2024–2026. Four broad searches
found **689 different papers**. A fifth, narrower search found **83 papers** on
loss of randomness and diversity. We read 15 main papers directly.

**Search limit:** OpenReview allowed only three requests per time window, so
our ICLR 2026 coverage is only a sample. Web search was unavailable because the
session had used its budget.

## Short answer

Self-improving AI is real, but “improvement” can mean several things:

- Updating a model's **context or memory** works.
- Updating its **weights**, the learned numbers inside the model, is still
  debated.
- Updating its own **code** can work, but costs a great deal.

This field is also heavily studied. Two large surveys appeared within twelve
months. The newer one was 97 pages long and only two weeks old. Each basic idea
we checked already had 5–14 papers.

The original recommendation was “do not enter.” That choice is now outdated.
The important exception is O1: the cause of sudden collapse during
self-training with a true code grader. The rest of the page records what the
survey found and why the first decision changed.

## Part 1: What “self-improving” means

Most systems use the same three-step loop:

1. Create possible answers or behaviors.
2. Score them with a **verifier**, which checks how good they are.
3. Save some kind of update.

The main difference is what gets saved. Two recent surveys give nearly this
same map. Ren et al. describe a system that creates and saves its own updates
to model weights or agent parts
([arXiv 2607.13104](https://arxiv.org/abs/2607.13104)). Fang et al. divide the
field into system inputs, the agent, its environment, and the optimizer
([arXiv 2508.07407](https://arxiv.org/abs/2508.07407)).

| Type of loop | What changes | Example work | Cost | Does it work? |
|---|---|---|---|---|
| **L1: Context** | prompt or playbook | ACE, [Combee](https://arxiv.org/abs/2604.04247), SIPDO, [GEPA](https://arxiv.org/abs/2507.19457) | almost free | **Yes** |
| **L2: Memory** | saved experiences | Evo-Memory, [EvolveR](https://arxiv.org/abs/2510.16079), [Contextual Experience Replay](https://arxiv.org/abs/2506.06698) | cheap | Partly |
| **L3: Weights after collecting data** | model weights learned from its own data | STaR/ReST, [Self-Rewarding LMs](https://arxiv.org/abs/2401.10020), R-Zero, Absolute Zero, SvS | medium | **Debated** |
| **L4: Weights during a test** | weights for one test item | [SEAL](https://arxiv.org/abs/2506.10943), TTT, SLOT | medium | Only in narrow cases |
| **L5: Code** | the agent's own program | [Gödel Agent](https://arxiv.org/abs/2410.04444), [Darwin Gödel Machine](https://arxiv.org/abs/2505.22954), Huxley-Gödel, AlphaEvolve | very high | Yes, at high cost |

### L1: Updating context works and gives the best value

**Agentic Context Engineering** by Zhang et al.
([arXiv 2510.04618](https://arxiv.org/abs/2510.04618)) had 229 citations in
under one year. It treats context like a playbook that can improve. It targets
two problems:

- **brevity bias:** shortening the playbook removes useful expert knowledge;
- **context collapse:** repeated rewriting slowly destroys important detail.

The paper reports gains of 10.6% on agent tasks and 8.6% on finance. On
[AppWorld](https://arxiv.org/abs/2407.18901), a smaller open model matched a
leading production agent without labeled training data.

No model weights change. This is the cheapest clear example of
self-improvement, which is why many groups entered this area quickly.

### L2: Updating memory has a good benchmark but weak methods

**Evo-Memory** ([arXiv 2511.20857](https://arxiv.org/abs/2511.20857)) is the
most useful shared test. It contains 10 datasets, gives models tasks one after
another, and rebuilds more than 10 memory systems. Models need to learn from a
long stream of interactions, but often lose useful lessons from earlier ones.
Older memory tests measured recall from one fixed conversation, which is a
different problem.

### L3: Updating model weights is the main debate

There are two sides.

**The hopeful side:**

- Absolute Zero ([arXiv 2505.03335](https://arxiv.org/abs/2505.03335)) lets one
  model create and solve coding and math tasks. A code runner checks the
  answers. In a setting with no outside data, it reached the best reported
  results and beat models trained on tens of thousands of human-made examples.
- R-Zero ([arXiv 2508.05004](https://arxiv.org/abs/2508.05004)) uses two
  models. A Challenger tries to write tasks near the edge of a Solver's skill.
  The Solver tries to answer them. On Qwen3-4B-Base, it gained 6.49 points on
  math and 7.54 on general tasks.
- [Agent0](https://arxiv.org/abs/2511.16043),
  [SPIRAL](https://arxiv.org/abs/2506.24119),
  [SPELL](https://arxiv.org/abs/2509.23863),
  [Vision-Zero](https://arxiv.org/abs/2509.25541), and
  [Multi-Agent Evolve](https://arxiv.org/abs/2510.23595) extend this pattern.

**The doubtful side:** Yue et al.
([arXiv 2504.13837](https://arxiv.org/abs/2504.13837)) measured `pass@k`, the
chance that at least one of `k` attempts is correct, instead of only `pass@1`.
Models trained with reinforcement learning from verifiable rewards (RLVR) beat
their base models when `k` is small, but lose when `k` is large. This suggests
that training helps select answers the base model could already produce,
rather than creating new reasoning skill. The authors say the base model limits
the result and that distillation, not reinforcement learning, truly expands
reasoning ability.

**The theory:** Song et al.
([arXiv 2412.02674](https://arxiv.org/abs/2412.02674)) focus on the
**generation-verification gap**: the difference between what a model can create
and what it can correctly check. They find that this gap grows steadily with
the compute used before training, so larger models may have more room to
improve themselves. Sun et al.
([arXiv 2507.00075](https://arxiv.org/abs/2507.00075)) fit a math model of the
same solver-versus-verifier gap to experiments. Both reach the same basic idea:
self-improvement uses the gap between creating and checking, then stops when
the gap closes.

### L5: Updating code works, but is expensive

AlphaEvolve ([arXiv 2506.13131](https://arxiv.org/abs/2506.13131)) found a way
to multiply 4×4 complex matrices with 48 multiplications. This was the first
improvement over Strassen's method for this setting in 56 years. It also
improved data-center scheduling and simplified circuits.

[Darwin Gödel Machine](https://arxiv.org/abs/2505.22954) and the
[Huxley-Gödel Machine](https://arxiv.org/abs/2510.21614), an ICLR 2026
submission, let agents change themselves without a fixed end point. This work
is real and impressive, but usually needs industry-level compute. Algorithm
discovery is also crowded. FunSearch,
[LLaMEA](https://arxiv.org/abs/2405.20132), SeaEvo,
[OR-Agent](https://arxiv.org/abs/2602.13769),
[BLADE](https://arxiv.org/abs/2504.20183), Latent Heuristic Search, and a
Zarankiewicz-number result all appeared in the same search.

## Part 2: Four known failure modes

### F1: Training improves, then suddenly collapses

“Self-Improvement Can Self-Regress”
([arXiv 2606.21090](https://arxiv.org/abs/2606.21090), June 2026) studies
REINFORCE training on programming contests. `pass@1` rises, peaks after only
tens of gradient steps, and then falls—sometimes almost to zero.

This general shape is older. “Can Large Reasoning Models Self-Train?”
([arXiv 2505.21444](https://arxiv.org/abs/2505.21444), May 2025) found it
fourteen months earlier with majority-vote feedback. That paper blamed
**reward hacking**: the model learns to raise the training score without truly
improving.

The 2026 result is still different in important ways:

- It is not forgetting between different tasks. The model over-trains its
  policy on one fixed task distribution.
- KL limits and EWC, two methods that try to stop harmful weight changes, do
  not prevent the collapse.
- The best fix is to stop at the highest checkpoint. The 7B model reaches
  22.2% `pass@1`. If early stopping is the best fix, no one has solved the
  cause.
- GRPO improves the worst point but does not remove the sudden drop.
- The paper tests Qwen-2.5-3B/7B, plus a Gemma-3-4B pilot, across 10 campaigns
  of 20 steps and several random seeds. This failure appears at small scale.
- Most importantly, the reward is a binary code grader with a truly right or
  wrong answer. The older majority-vote reward could be hacked; the code
  grader should not be. That unexplained difference is narrow but real.

### F2: Answers lose randomness and variety

This nearby area is much busier. **Entropy** measures how spread out a model's
possible choices are. During RL, entropy tends to fall because the relation
between action probability and logit change stays positive, according to Cui
et al. ([arXiv 2505.22617](https://arxiv.org/abs/2505.22617)). They report
`R = -a·e^H + b`, with a hard upper limit when entropy reaches zero, and offer
Clip-Cov and KL-Cov as token-level fixes.

The Reasoning Boundary Paradox
([arXiv 2510.02230](https://arxiv.org/abs/2510.02230)) adds two possible causes:

- **negative interference:** learning some problems makes the model worse on
  others;
- **winner take all:** RLVR strengthens answers that were already likely and
  hides answers that began as unlikely.

SvS ([arXiv 2508.14029](https://arxiv.org/abs/2508.14029)) creates different
problems that share the same reference answer. It improves Pass@32 by 18.3 and
22.8 points on AIME24 and AIME25 and scales from 3B to 32B models.

### F3: A system loses skills over its lifetime

“Do Self-Evolving Agents Forget?”
([arXiv 2605.09315](https://arxiv.org/abs/2605.09315), May 2026) finds skill
loss when agents change any of four parts: workflow, skill, model, or memory.
It proposes Capability-Preserving Evolution. The paper was only two months old
at survey time, so that group already owns the first clear claim.

### F4: A self-changing agent can become unsafe

Examples include “Your Agent May Misevolve”
([arXiv 2509.26354](https://arxiv.org/abs/2509.26354)), “Zombie Agents:
Persistent Control of Self-Evolving LLM Agents via Self-Reinforcing
Injections” ([arXiv 2602.15654](https://arxiv.org/abs/2602.15654)),
[SAHOO](https://arxiv.org/abs/2603.06333), and two surveys devoted to safety.
This is a real problem, and the work is moving quickly.

## Part 3: How much work already exists

| Starting idea | Papers found in this search | Result at survey time |
|---|---:|---|
| Self-play or a curriculum with no outside data | 20+ | **Already studied heavily** |
| RLVR `pass@k` and reasoning limits | 14 | **Already studied heavily**; a named debate already has a two-stage paper joining the sides |
| Model collapse from training on generated data again and again | 7+ | **Already studied heavily**; Nature paper plus follow-ups |
| Verifier or generation-verification gap | 6+ | **Already studied heavily** |
| Updating context and prompts | 6+ | **Already studied heavily**; ACE leads |
| LLM-based algorithm discovery | 10+ | **Already studied heavily** |
| Updating memory | 6+ | Busy; a benchmark had just appeared |
| Safety of self-changing agents | 5+ | Busy and moving fast |
| Entropy or diversity loss in RLVR | **11+ methods and 83 papers** | **Already studied heavily**; two ACL 2026 papers try to prevent it |

Two broad surveys appeared in twelve months:
[arXiv 2508.07407](https://arxiv.org/abs/2508.07407) in August 2025 and
[arXiv 2607.13104](https://arxiv.org/abs/2607.13104), a 97-page survey that was
two weeks old. This strongly suggests that the easy questions are gone.

The search also found an unusual amount of weak work on SSRN, Zenodo, and
ResearchGate, with names such as “Recursive Psyche Improvement,” “Zero Leap
Theory,” and “The Gödel Dyad.” The phrase “self-improving AI” attracts poor
work, so a paper using it in the title must meet a higher evidence bar.

## Part 4: Ideas we checked

We considered three kinds of contribution: improve an existing method, name a
new problem, or move an idea to a new field. We allowed training, but required
the work to fit academic compute. The owner preferred a new method over a
measurement-only paper.

### O1: Why does one self-training campaign collapse? Originally ★★; now ★★★★

**The old text below first rejected this idea after checking recent work. The
2026-08-02 re-evaluation reversed that choice.**

The new check found that entropy papers do not test any of the three parts that
matter here: a sequence of campaigns, true code graders, or sudden collapse in
about 20 steps. Entropy falls slowly, while this score falls like a phase
change. The two published collapse settings give explanations that cannot both
be right, and nobody has compared them. The remaining question—collapse under
a ground-truth grader—is now a four-cause comparison, not another proposed
entropy fix. See [[Direction-Reevaluation-2026-08]].

The first idea was to combine three results: F1's unexplained collapse, the
entropy-covariance account, and the Reasoning Boundary Paradox. A 3B–7B test
would cost under 300 GPU-hours.

The first search seemed to find two reasons to reject it:

1. **The general rise-and-collapse shape was already known.**
   [2505.21444](https://arxiv.org/abs/2505.21444), published fourteen months
   before F1, saw sudden and complete collapse with majority-vote feedback and
   blamed reward hacking.
2. **Many groups already test entropy fixes.** A focused search found 83
   different 2025–2026 papers and at least 11 direct methods: Clip-Cov/KL-Cov,
   [CURE](https://arxiv.org/abs/2508.11016), Rethinking Entropy Regularization
   in Large Reasoning Models, Rethinking Entropy Interventions in RLVR (ACL,
   45 citations), Understanding and Preventing Entropy Collapse in RLVR with
   On-Policy Entropy Flow Optimization (ACL 2026), HEALing Entropy Collapse
   (ACL 2026), EP-GRPO, PAEC, Flexible Entropy Control in RLVR, Precise Entropy
   Curve Control, and Understanding Diversity Collapse in RLVR via the Lens of
   Overtraining. Nearby work includes Uniform-Correct Policy Optimization,
   [R-Diverse](https://arxiv.org/abs/2602.13103), UpSkill, Pass@K Policy
   Optimization, and [max@k](https://arxiv.org/abs/2510.23393).

Two ACL 2026 papers even use “understanding and preventing entropy collapse”
in their titles. Another generic entropy method has little room.

The narrow difference is the reason O1 survives. Shafayat et al. study a
majority-vote **pseudo-reward**, which a model can game. F1 uses a binary code
grader with a true answer, so reward hacking cannot explain the collapse.
No paper explains that difference. The old page rejected it because eleven
groups worked nearby. The new review correctly asks whether those groups answer
this exact question; they do not.

**Process lesson:** do not accept a paper's own claim that its problem is new.
Search the whole area first. Also, do not reject an exact open question only
because nearby work is busy.

### O2: Self-improving VLMs with a free checker — ★★★☆☆

Most self-improvement work uses text, math, or code because a code runner is a
free and nearly perfect checker. Outside math and code, it is hard to score new
answers cheaply. VLM work is thinner:
[Vision-Zero](https://arxiv.org/abs/2509.25541), Self-Rewarding VLM via
Reasoning Decomposition, Calibrated Self-Rewarding VLMs,
[Pixel Reasoner](https://arxiv.org/abs/2505.15966), and one study of whether
RLVR expands VLM reasoning limits.

The idea is to use **compositional consistency** as a free checker. When a
caption and a hard negative are scored, swapping the roles should reverse their
order. A different caption with the same meaning should not. This rule needs no
human labels. We already have the benchmark set, the `alpha=1` evaluator check,
and paired confidence intervals in the `svib` wiki.

The risk is serious. Six very different tests already failed in the frozen
dual-encoder local-branch family, and more work there rates ★ in
[[Status-And-Survivors]]. This new idea trains a generative VLM, so it is not
the same method. Still, the nearby failures may repeat, and the same benchmarks
may again fail to separate methods. Run a small pilot before committing.

### O3: Keep skills during lifelong change — ★★☆☆☆

This would improve existing work. F3 clearly named the problem and proposed
Capability-Preserving Evolution two months before the survey. The problem is
real, but that group started first and can move faster than us.

### O4: Another survey or audit of self-improvement — ★☆☆☆☆

Do not choose this. Two surveys already exist, including a 97-page survey from
two weeks before our check. A third audit would mainly measure the field, not
add a method.

## Part 5: Old recommendation, kept as a record

> **This whole section became outdated on 2026-08-02.** The self-training
> cause question now rates ★★★★. The fallback advice also used old ratings:
> T4 later fell from ★★★★★ to ★★ after three competing papers.
> [[Direction-Reevaluation-2026-08]] records why those August 2 changes were
> made. Use [[Unified-Direction-Ranking-2026-08]] for current decisions.

The old survey said not to enter this field. It counted 689 broad-search
papers, 83 entropy-search papers, two major surveys in twelve months, and at
least 11 methods on its favorite sub-question. Every idea it first drafted
seemed occupied.

It then suggested returning to T1 and T4 in [[Status-And-Survivors]]. At that
time, both rated ★★★★★. T1 compared frozen parts, training goals, and training
stages for 300–600 GPU-hours. T4 studied when to add data during training and
had the largest measured effect in the search. Those ratings are also outdated.

The one useful fallback was the exact question that later survived: why does
self-training collapse with a true code grader when reward hacking explains
only the pseudo-reward case? The proposed two-week pilot would reproduce the
curve on Qwen-2.5-3B and test whether existing entropy methods move the sudden
drop.

Do not lead a paper title with “self-improving AI.” The search found too much
weak work under that phrase. Describe the exact topic instead, such as training
dynamics or stability during RL post-training.

## Related

- [[Unified-Direction-Ranking-2026-08]] — current direction ranking.
- [[Direction-Reevaluation-2026-08]] — historical August 2 decision that first
  replaced O1 and Part 5 on this page.
- [[Top-Researcher-Scan-2026-08]] — its diversity-collapse theme is the
  researcher-level view of F2.
- [[Status-And-Survivors]] — other ideas that were alive at the time; its star
  table is also outdated.
- [[Method-Opportunities]] · [[Live-Research-Opportunities]]

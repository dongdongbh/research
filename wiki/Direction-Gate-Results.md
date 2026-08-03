# Direction Gate Results

*Updated 2026-08-02 for the general research wiki.*

Later update: [[Direction-Reevaluation-2026-08]] changed the lesson about how
to find research ideas. This page had said to look for areas with few papers.
The later review found that the number of papers predicted nothing. The new
rule is: look at how much useful work is still left. If an area looks empty,
we must also explain why other researchers have stayed away. The gate results
below are still kept as the record of what happened.

Status: **Four gates were run on 2026-07-25.** A gate is a check of whether
recent or older work has already done the proposed research. Three ideas were
stopped, and one was lowered in priority. Two ideas survived. For both
survivors, the main challenge is careful measurement, not new theory. The
gates below are ordered by how much their failures taught us.

## The failure that kept happening

Read this before making more candidates. Four of the five gates so far,
including the earlier GPTQ gate, stopped an idea for one of two reasons:

GPTQ is a method that compresses a trained model by rounding its weights to a
small set of values. Those possible rounded values can be viewed as points on
a mathematical grid called a lattice. An SSM, or state-space model, carries a
fixed-size hidden state from one token to the next. These are the two technical
examples used below.

1. **An expert could fill the gap in one line.** The GPTQ/lattice idea was
   already a future-work sentence in its main paper. The SSM succinctness
   claim follows directly from Prop 1 of its paper. The camera-ready abstract
   states the claim, but the paper gives no proof in the body, because an
   expert could write the proof in an afternoon.
2. **An older field had already solved the problem, using different words.**
   The benchmark unidimensionality idea is Horn's parallel analysis
   (Psychometrika 1965). The random-matrix-theory, or RMT, version was already
   published by Dobriban & Owen (JRSS-B 2019). The distillation U-shape was
   already proved in an ICML 2025 appendix.

A search for method names will miss both problems. Search by the **task, the
benchmark, and the older field's terms** before taking a new idea seriously.

## Gate 1: distillation capacity gap — SCOOPED, and our framing was wrong

**The theorem already exists and has a name.** Busbridge et al.,
*Distillation Scaling Laws* ([`2502.08606`](https://arxiv.org/abs/2502.08606),
Apple, **ICML 2025**), has Appendix C.1.3, titled *"U-shape in the student
error."* It gives Lemma C.1 and Lemma C.2. It proves two cases: error decreases
when `m < n`, increases when `m >= n`, and reaches an inside optimum at
`m ~ n`. The proof ends with a formal QED. The target theorem was proved and
named a year ago. An earlier survey called this paper "fitted, not derived."
**That statement was wrong**, and this page corrects it.

**The contradiction that motivated the idea also does not exist.** Menon et
al. ([`2005.10419`](https://arxiv.org/abs/2005.10419)) does **not** say that a
more capable teacher must always help. It has sections titled *"Why can more
accurate teachers distill worse?"* and *"Trading off bias for variance: model
complexity."* Its synthetic experiment finds an ideal depth of `d = 8`. A
reviewer who had read Menon would reject the claim that our idea contradicts
that paper.

There was already more related work. Harutyunyan et al. §3.1 is titled
*"Trade-off between teacher accuracy, margin, and complexity"* and finds an
inside optimum for temperature. [Ildiz et al.](https://arxiv.org/abs/2410.18837)
Prop 3 already finds an inside optimum for teacher capacity in a high-dimensional
asymptotic model, meaning a model of behavior as the number of dimensions grows.

**The effect may not even be reliable.** In CRD Table 10, the same student is
trained from teachers whose accuracy differs by 4.7 points. The student's
score differs by only `-0.02`, while the run-to-run spread is `sigma = 0.32`.
DKD says the real cause is *"the suppression of NCKD"*: a feature of the
coupled KL training objective. Replacing KL with DKD changes the relationship
between teacher and student ranking from `rho ~ 0.26` to `rho ~ 0.94`.

**Surviving idea:** write the *deflationary* paper. In other words, test whether
the U-shape comes from the coupled objective, not from model capacity, and
whether it disappears with a decoupled loss. The evidence already supports
this skeptical view. The study needs strong control experiments, not
convex-Gaussian min-max theorem (CGMT) theory.

## Gate 2: one-dimensional benchmark ability — SCOOPED TWICE

Here, *unidimensional* means that one hidden ability explains benchmark
performance.

**The statistical method already exists.** This is Horn's parallel analysis
(1965, 6,518 citations). Its modern RMT replacement is already published:
Dobriban & Owen, *Deterministic Parallel Analysis* (**JRSS-B 2019**), and
Dobriban, *Permutation methods for factor analysis and PCA* (**Annals of
Statistics 2020**). Finding the number of hidden factors with the BBP
transition is settled in econometrics through Onatski 2009/2010, Bai & Ng, and
Ahn & Horenstein. A new paper entered this area three weeks ago
([`2607.06908`](https://arxiv.org/abs/2607.06908)).

**The exact data question was also already answered.** *AI Cartography*
([`2605.25272`](https://arxiv.org/abs/2605.25272), May 2026) studies item-level
responses from the Open LLM Leaderboard: more than 4,000 models on six
benchmarks. It uses confirmatory factor analysis and structural equation
modeling (CFA/SEM), compares six possible hidden structures including the
one-factor model, and includes permutation controls. It concludes that both
the one-factor view and the view that each benchmark is independent are not
supported. Prop 4.1 already gives the important decision: benchmark totals are
not enough to describe a model's hidden capability profile.

The planned test also had two technical traps. First, after removing the main
effects from a **binary** matrix, each item's leftover variance is `p(1-p)`.
The correct null is therefore a generalized Marchenko-Pastur, or MP, law with
different variances, not the standard MP law. Second, items are grouped inside
benchmarks. That grouping guarantees a second spike, so finding it would be an
expected result rather than a discovery.

## Gate 3: transformer succinctness — (a) stopped, (b) open but hard, (c) survives

*Succinctness* means how compactly a model can describe a language or behavior.

**The main paper is less new than we first reported.** Its key idea—a short
description whose shortest accepted word is extremely long—comes from
**Meyer & Stockmeyer 1972**. The field has called this "economy of description"
since 1971. A full-text check of the paper found **zero** mentions of Horne,
Hush, Sanford, Telgarsky, Hsu, Holzer, Kutrib, Gelade, Neven, Alon, Indyk, or
Meyer. Three missing references matter:

- **Horne & Hush (NIPS 1993 / Neural Networks 1996)** gave size limits for RNNs
  that implement finite-state machines. Their bounds were `O(sqrt(m))` with no
  restrictions, `O(sqrt(m) log m)` with weights in `{-1,1}`, and `O(m)` with
  fan-in 2. They also gave matching lower bounds. This was 32 years earlier,
  and it measured RNN size directly.
- **[Sanford, Hsu & Telgarsky](https://arxiv.org/abs/2306.02896) (NeurIPS
  2023)** already proved a transformer-versus-RNN **size** separation using
  communication complexity, which is the sharper method.
- **[Gelade & Neven](https://arxiv.org/abs/0802.2869) (STACS 2008)** proved
  tight, matching double-exponential bounds. That result is stronger in form
  than Thm 17 of the transformer paper.

The paper does have original results: the UHAT counter construction, the
exponential UHAT-to-LTL translation that improves an older doubly exponential
translation, and EXPSPACE-**completeness**.

**(a) SSM/[Mamba](https://arxiv.org/abs/2312.00752) succinctness — SCOOPED.
Kill this idea.** An SSM is a state-space model. At fixed numeric precision, it has
a bounded state vector, so it is a deterministic finite automaton (DFA) with
`2^{kD}` states. Thm 17 then applies directly. The camera-ready abstract already
states this result even though the body has no proof. Jelassi et al. (ICML 2024)
also published a transformer-versus-SSM size separation with experiments. A
structure-aware version would first need a definition of "SSM size." No agreed
cost model exists because papers treat the MLP between layers as free.

**(b) Beyond fixed precision — OPEN, hard for a clear reason, and crowded.**
The shortest-accepted-word method stops working when precision is not fixed.
The proof would need fooling sets or communication complexity. This is real,
non-routine research, but Chakrabarti, Pitassi & Alman are now working in this
area. A solo researcher would need 9–18 months to prepare, and would likely
finish second.

**(c) Empirical succinctness — SURVIVES, with stronger reasons than we first
thought.** Researchers in descriptional complexity have named **intrinsic size
measures—parameters and numeric precision as a trade-off surface** as an open
problem. This is the theory version of measuring a fixed bit budget. Because
there is no agreed meaning of "SSM size," measuring the bits actually used
also avoids a definition fight that the theory community has not settled. The
authors admitted this gap to a reviewer: *"unclear if a training algorithm will
always converge to such a succinct transformer."* The award committee also
asked for this work in writing.

The protocol would define `s_exist(n)` and `s_learn(n)`. They are the smallest
bit budgets, measured as parameters × precision, at which a model architecture
*can* recognize a graded family of languages and at which training *does*
learn to recognize it, up to a chosen error limit. Bound `s_exist` from above
with a hand-built construction plus distillation, pruning, and quantization.
Estimate `s_learn` with best-of-K-seed training. **The difference between the
two is the main object to study.** Another group could still publish first:
Cotterell's group studies empirical formal-language learning, and Lin's group
builds C-RASP synthesis with program-size optimization. The likely window is
12–18 months.

## Gate 4: C-RASP learnability — ACTIVE, small, but busy and speeding up

The candidate found a narrow opening, but wrongly described the whole field as
small. Two funded, prize-winning groups work together here. One is
Lin/Zetzsche/Chiang/Yang on formal methods; the other is Hahn/Bhattamishra on
learning. Together they published about **six ICML/ICLR 2026 papers**. Hahn has
an Emmy Noether grant, won the 2026 Heinz Maier-Leibnitz Prize, and is hiring
people to work on this topic.

The claim that "little work investigates learnability" is now outdated.
Chen/Ma/Li (ICML'25), Yang et al. (ICML'26), Izzo et al., Huang et al.
(NeurIPS'25), and two Bhattamishra papers all study it. The part that is still
thin is *sample complexity*: how many examples learning needs. The target paper
([`2607.11760`](https://arxiv.org/abs/2607.11760)) was only 12 days old when
this gate ran. Its four authors include the two people best placed to finish
the work.

The area also moves very fast. In early 2026, researchers clearly named the
problem of computing C-RASP length-generalization bounds. By ICML 2026, the
problem was **fully solved**.

**Least-contested opening:** study length complexity and sample complexity
together. Chen/Ma/Li cover length. Rizvi-Martel cover samples. Nobody covers
the combined question. A researcher with a statistics background could prepare
in 4–8 weeks, and the work needs one GPU. **Important limit:** the easier half
is an *upper* bound from counting parameters. The result that makes a paper is
the *lower* bound. That needs constructions showing what the model can shatter,
and it moves back toward the harder formal-theory side.

The experimental branch is even more crowded. Huang et al. (ICLR 2025)
already tested whether C-RASP predicts the behavior of trained transformers.
An ICML 2026 paper, 104 pages long, already turns transformers back into RASP
programs.

## Related

[[Math-Grounded-Direction-Survey]] — the source of these candidates.
[[Home]] — the standing rule to check whether prior work already did an idea;
this was originally recorded as Stage-E-Prior-Art-Audit in the svib repo wiki.

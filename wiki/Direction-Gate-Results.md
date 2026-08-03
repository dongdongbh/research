# Direction Gate Results

*Updated 2026-08-02 for the general research wiki.*

Later development: the generating-process lesson below (scan for empty areas →
the area is never empty) was revised by [[Direction-Reevaluation-2026-08]],
which found crowd-count filtering predicted nothing and replaced it with
"filter by remaining opportunity, and an empty lane must explain why it is
empty." The gate verdicts on this page stand as recorded.

Status: **Four gates run 2026-07-25.** Three candidates killed, one downgraded.
Two survivors, both of which put measurement rather than theory on the critical
path. Gates ordered by how instructive the failure was.

## The recurring failure mode — read this before generating more candidates

Four of the five gates run so far (including the earlier GPTQ one) killed the
candidate for one of exactly two reasons:

1. **The gap is one line for an insider.** GPTQ/lattice was the anchor paper's
   own stated future-work sentence. SSM succinctness is a one-line corollary of
   the paper's Prop 1 — asserted in the camera-ready abstract with *no proof in
   the body* because an expert writes it in an afternoon.
2. **An older literature already solved it under different vocabulary.**
   Benchmark unidimensionality is Horn's parallel analysis (Psychometrika 1965)
   with the RMT version already published (Dobriban & Owen, JRSS-B 2019). The
   distillation U-shape was proved in an ICML 2025 appendix.

Both failure modes are invisible to method-name searches and only surface when
searching by **task, benchmark, or the older field's vocabulary**. Any future
candidate must be gated that way before it is taken seriously.

## Gate 1: distillation capacity gap — SCOOPED, and my framing was factually wrong

**The theorem exists and is named.** Busbridge et al., *Distillation Scaling
Laws* ([`2502.08606`](https://arxiv.org/abs/2502.08606), Apple, **ICML 2025**), **Appendix C.1.3, titled "U-shape in
the student error."** Lemma C.1, Lemma C.2, two cases (`m < n` decreasing,
`m >= n` increasing), interior optimum at `m ~ n`, closed with a formal QED.
That is the target theorem, proved, named, a year ago. An earlier survey
described this paper as "fitted, not derived" — **that was wrong** and is
corrected here.

**Worse, the motivating contradiction does not exist.** Menon et al.
([`2005.10419`](https://arxiv.org/abs/2005.10419)) does **not** predict monotone improvement in teacher capacity. It
contains a subsection literally titled *"Why can more accurate teachers distill
worse?"*, a subsection *"Trading off bias for variance: model complexity"*, and
a synthetic experiment reporting *"There is an optimal depth d = 8."* The pitch
"this contradicts Menon" would not survive a reviewer who has read Menon.

Also: Harutyunyan et al. §3.1 is titled *"Trade-off between teacher accuracy,
margin, and complexity"* and derives an interior optimum (in temperature), and
[Ildiz et al.](https://arxiv.org/abs/2410.18837) Prop 3 already derives an interior optimum in teacher capacity in
high-dimensional asymptotics.

**The phenomenon may not even be robust.** CRD Table 10: same student, teachers
4.7 points apart in accuracy, student differs by `-0.02` against `sigma = 0.32`.
DKD states verbatim that the real cause is *"the suppression of NCKD"* — a
property of the coupled KL objective — and swapping KL for DKD moves
teacher/student rank correlation from `rho ~ 0.26` to `rho ~ 0.94`.

**Survivor:** the *deflationary* paper — show the U-shape is a property of the
coupled objective rather than of capacity, and that it dissolves under a
decoupled loss. Contrarian, evidence already on its side, and it needs controls
rather than CGMT.

## Gate 2: benchmark unidimensionality — SCOOPED TWICE

**Statistically**, this is Horn's parallel analysis (1965, 6,518 citations), with
the modern RMT replacement already published: Dobriban & Owen, *Deterministic
Parallel Analysis* (**JRSS-B 2019**) and Dobriban, *Permutation methods for
factor analysis and PCA* (**Annals of Statistics 2020**). Factor-number
determination via BBP is settled in econometrics (Onatski 2009/2010, Bai & Ng,
Ahn & Horenstein), with a new entry three weeks ago ([`2607.06908`](https://arxiv.org/abs/2607.06908)).

**Empirically**, the question is already answered on the exact data. *AI
Cartography* ([`2605.25272`](https://arxiv.org/abs/2605.25272), May 2026) runs item-level CFA/SEM on Open LLM
Leaderboard responses — 4,000+ models, six benchmarks — compares six competing
latent structures including unidimensional, uses permutation controls, and
concludes *"unidimensional and independent-benchmark assumptions are
untenable."* Its Prop 4.1 already delivers the changed decision: benchmark
totals are not sufficient statistics for latent capability profiles.

Two technical landmines confirm it was ill-posed anyway: after stripping main
effects from a **binary** matrix the residual variance is `p(1-p)`, so the null
is a generalized MP law with a variance profile, not standard MP; and item
nesting within benchmarks guarantees a second spike, making detection a
foregone conclusion rather than a finding.

## Gate 3: transformer succinctness — (a) killed, (b) hard and crowded, (c) survives

**The anchor paper is less novel than reported.** The core technique — a small
description forcing an astronomically long shortest accepted word — originates
in **Meyer & Stockmeyer 1972**; the field has been called "economy of
description" since 1971. A full-text citation audit found **zero** occurrences
of Horne, Hush, Sanford, Telgarsky, Hsu, Holzer, Kutrib, Gelade, Neven, Alon,
Indyk, or Meyer. Three omissions are damaging:

- **Horne & Hush (NIPS 1993 / Neural Networks 1996)** gave size bounds for RNNs
  implementing finite state machines — `O(sqrt(m))` unrestricted,
  `O(sqrt(m) log m)` with weights in `{-1,1}`, `O(m)` with fan-in 2 — **with
  matching lower bounds**, 32 years earlier, measuring RNN size *intrinsically*.
- **[Sanford, Hsu & Telgarsky](https://arxiv.org/abs/2306.02896) (NeurIPS 2023)** already prove a transformer-vs-RNN
  **size** separation using communication complexity, the sharper technique.
- **[Gelade & Neven](https://arxiv.org/abs/0802.2869) (STACS 2008)** prove tight matched double-exponential bounds,
  a stronger result form than the transformer paper's Thm 17.

What is genuinely the paper's own: the UHAT counter construction, the
exponential UHAT-to-LTL translation (improving a doubly-exponential one), and
EXPSPACE-**completeness**.

**(a) SSM/[Mamba](https://arxiv.org/abs/2312.00752) succinctness — SCOOPED. Kill.** Asserted in the camera-ready
abstract with no body proof, because every fixed-precision SSM is a bounded
state vector, hence a DFA with `2^{kD}` states, hence Thm 17 applies. Jelassi et
al. (ICML 2024) independently published a transformer-vs-SSM size separation
with experiments. A structure-aware version would first require inventing a cost
model for "SSM size," which does not exist because every paper treats the
inter-layer MLP as free.

**(b) Beyond fixed precision — OPEN, hard for a principled reason, crowded.**
The shortest-accepted-word trick *fails* outside fixed precision; you must
import fooling sets or communication complexity. That is real research, not
routine — but Chakrabarti, Pitassi & Alman are now in this lane. Solo ramp
9-18 months, and second place.

**(c) Empirical succinctness — SURVIVES, and is better motivated than I thought.**
The descriptional-complexity community's own flagged open problem is **intrinsic
size measures — parameters and precision as a trade-off surface** — which is the
theoretical counterpart of a bit-budget protocol. And because "SSM size" has no
agreed cost model, measuring bits actually used **sidesteps a definitional fight
the theory community has not settled.** The authors conceded exactly this gap to
a reviewer (*"unclear if a training algorithm will always converge to such a
succinct transformer"*), and the award committee invited it in writing.

Protocol shape: define `s_exist(n)` and `s_learn(n)` as the minimal bit budget
(parameters x precision) at which an architecture *can* versus *does* recognize
a graded language family to tolerance; bound `s_exist` above by construction plus
distillation/pruning/quantization; estimate `s_learn` by best-of-K-seed
training; **report the gap as the object of study.** Scoop risk is real
(Cotterell's group publishes empirical formal-language learnability; Lin's group
ships C-RASP synthesis with program-size optimization). Window 12-18 months.

## Gate 4: C-RASP learnability — ACTIVE, small but crowded and accelerating

The candidate correctly found a thin spot and mislocated it as a thin field.
Two funded, prize-winning, mutually collaborating groups (Lin/Zetzsche/Chiang/
Yang on the formal-methods axis; Hahn/Bhattamishra on the learning axis) placed
roughly **six ICML/ICLR 2026 papers** between them. Hahn holds an Emmy Noether
grant and the 2026 Heinz Maier-Leibnitz Prize and is hiring onto this agenda.

The framing claim is stale: "little work investigates learnability" is
contradicted by Chen/Ma/Li (ICML'25), Yang et al. (ICML'26), Izzo et al., Huang
et al. (NeurIPS'25), and two Bhattamishra papers. What is actually thin is
*sample* complexity specifically. And the target paper ([`2607.11760`](https://arxiv.org/abs/2607.11760)) is 12 days
old, by four insiders including the two best placed to finish it.

Tempo warning: the computability of C-RASP length-generalization bounds was an
explicitly named open problem in early 2026 and was **fully resolved by ICML
2026**.

**Least-contested seam:** joint length x sample complexity — Chen/Ma/Li own
length complexity, Rizvi-Martel own sample complexity, nobody owns the joint
object. Statistics-side ramp is genuinely short (4-8 weeks) and compute is one
GPU. **Honest caveat:** the accessible half is the *upper* bound via parameter
counting; the paper-making half is the *lower* bound, which needs shattering
constructions and drifts back to the formal side.

The empirical branch is *more* crowded, not less — Huang et al. (ICLR 2025)
already tested whether C-RASP predicts trained-transformer behaviour, and an
ICML 2026 paper (104 pages) decompiles transformers to RASP.

## Related

[[Math-Grounded-Direction-Survey]] — where these candidates came from.
[[Home]] — the standing prior-art-gate rule that produced these gates (rule 1);
originally recorded as Stage-E-Prior-Art-Audit (svib repo wiki).

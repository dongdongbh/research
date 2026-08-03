# Math-Grounded Direction Survey

Status: **Complete on 2026-07-25.** This page covers the theory parts of the
scouting work: [GeoLAN](https://arxiv.org/abs/2603.19460), the shape of learned
representations, math and theoretical-computer-science (TCS) results with open
ML uses, and newer topics with few papers. [[Field-Scouting-Survey]] covers the
experiment and measurement parts.

**Updated 2026-08-02 for the general research wiki.** The first version of each
idea remains here. Dated notes explain when a later check changed the result.

**Most ideas below were later checked and did not survive.** When this page was
first written, none had been checked against all earlier work. Its warning was:
*"No prior-art gate has been run on any candidate below. Per the standing rule
(Stage-E-Prior-Art-Audit — svib repo wiki), nothing here is actionable until
gated."* Here, a *prior-art gate* means checking whether recent or older work
already did the idea. Four gates ran on 2026-07-25 and are recorded in
[[Direction-Gate-Results]]. [[Direction-Reevaluation-2026-08]] later changed
the KV-cache idea and is now a historical August 2 record. For current
decisions, use [[Unified-Direction-Ranking-2026-08]]. **Read those results
before acting on this page.** The main
lesson was simple: an idea often fails because an expert can fill the gap in
one line, or because an older field solved it under different terms. Searching
only for a modern method name misses both problems.

## [GeoLAN](https://arxiv.org/abs/2603.19460) uses math mostly as decoration

Pan & Woodard, Findings of ACL 2026. We checked these points by reading the PDF.

GeoLAN is a **training-time regularizer**, meaning an extra training penalty,
not a theory paper. It combines two existing tools—isotropy regularization with
random projections and attention spectral-entropy regularization—and presents
them with Kakeya/Wolff language.

- **No Kakeya theorem appears in any proof.** Wolff, Guth-Zahl, and
  [Wang-Zahl](https://arxiv.org/abs/2502.17655) are only motivation.
- Theorem 1 never uses its K-stickiness assumption.
- The paper defines grains as connected components. Lemma 3 is therefore a
  basic fact from point-set topology for any continuous paths, including an
  untrained model or a model whose features completely collapse.
- For the orthogonality claim, the proof says the components do not overlap, so
  `Vol(empty set) = 0 <= C/K`. The paper itself calls this **"(trivially)"**.
- The Lipschitz bound simply multiplies a constant from each layer. It grows
  exponentially with depth, so it tells us nothing for a large model. The
  paper creates K-dependence only by *naming* the per-layer constants with K.
- The linear-probe result **assumes** the operator-norm limit that it is meant
  to prove.
- The argument has a missing middle step:
  Loss → isotropy/entropy → **???** → Axioms → Theorem 1. **K is never
  measured.**
- Experiments report about 25 Cohen's d values with positive and negative
  signs, no correction for making many comparisons, and `n=4` seeds. The paper
  calls `p=0.078` "significant" and says `p=0.05` "approaches significance."
  MMLU changes of `+0.75/+0.55/+0.15%` are within random-seed noise.
- The estimated training cost is about `1.8e22` FLOPs, or **6,300
  B200-GPU-hours**. The paper never reports that number. It is far beyond a 4–16 GPU
  budget. The advertised code repository is only a stub.

**Do not copy this paper's style.** It is the bad pattern: math used as branding
rather than as a tool that changes the result.

## A simple rule for math papers that get accepted

The survey found three separate examples:

- Xu et al.'s diagonalization proof in *Hallucination is Inevitable* was
  **rejected by TMLR**. [Kalai et al.](https://arxiv.org/abs/2509.04664) instead
  reduced the claim to a classifier that can be measured, and that version was
  accepted.
- *[The Invisible Leash](https://arxiv.org/abs/2507.14843)* was **rejected at
  ICLR 2026**. A fully experimental paper making the same claim received a
  **NeurIPS 2025 Oral**.
- A short, **single-author proof** that GPTQ equals Babai's algorithm received
  an **ICLR 2026 Poster**.

> **A theorem should limit or guide a method people really use. A researcher
> should also be able to read its constant from a real model.**

[GeoLAN](https://arxiv.org/abs/2603.19460) fails both parts. The GPTQ paper
passes both. It shows that one person can publish a small, math-based result in
this area.

## Ranked ideas

### 1. GPTQ and lattice reduction — **GATED AND KILLED 2026-07-25**

The prior-work check fully closed this idea. [[Direction-Gate-Results]] counts
it as the first of five gates and as the clearest example of a gap that was
already a future-work sentence in the main paper. The full reasons remain here
because they are useful warnings.

1. **The idea is the exact future-work sentence in the main paper and a second
   paper.** Chen et al. say their result can "open the door to importing decades
   of progress in lattice algorithms towards the design of future quantization
   algorithms." Birnick's separate proof
   ([`2508.01077`](https://arxiv.org/abs/2508.01077), also ICLR 2026) ends by
   "hinting at the possibility of using lattice basis reduction for improved
   quantization." Two groups publicly claimed this path.
2. **Klein randomized rounding was already used.** OJBKQ
   ([`2602.08376`](https://arxiv.org/abs/2602.08376), February 2026) uses
   box-constrained Babai plus extended Klein and keeps the K best remaining
   errors for LLM post-training quantization.
3. **The OJBKQ authors are from Xiao-Wen Chang's group**, a leading group for
   LLL applied to integer least squares. They knew LLL/BKZ, called reduction the
   standard fix in their related-work section, and still chose Klein. That is a
   knowledgeable negative sign, not something they missed.
4. **The required box makes full reduction impossible.** Reduction applies a
   unimodular matrix `U`. It turns the required box `{0..2^b-1}^n` into a
   slanted parallelepiped. Work on box-constrained integer least squares already
   knows this problem and can use only the column-swapping part of LLL.
5. **Even the legal part of LLL is expected to hurt here.** Wen & Chang prove
   that when noise is fairly large, LLL-P lowers Babai's chance of success.
   Two-bit and three-bit quantization are in that large-noise range.
6. **GPTQ already includes that legal part** under the names `act_order` and
   `desc_act`.
7. **The measured room to improve is about 0.03 perplexity.** Klein K-best is
   a stronger closest-vector-problem (CVP) solver than Babai, but on 4-bit
   Llama3-8B it changes perplexity only by `9.38 -> 9.35`, with no more gain
   after K=5. Most of OJBKQ's improvement came from its objective, not its
   solver.
8. **The deepest problem is that reduction changes the basis, not the lattice.**
   Exact CVP gives the same answer for every basis. Reduction can only shrink
   the difference between Babai and exact CVP. WaterSIC measures the remaining
   error even at exact CVP as `0.255` bits. It says the integer lattice is a
   poor shape for vector quantization. That is a **codebook space-filling**
   problem, which basis reduction cannot fix.

One path technically survives. WaterSIC removes the box restriction by using
unbounded integers and entropy coding. Full unimodular reduction becomes legal,
so objections 4 and 5 go away. But this would compete with
Ordentlich/Polyanskiy on their own topic for at most 0.255 bits, and reduction
cannot recover most of even that. **If this is ever reconsidered, use a two-hour
stop test:** apply LLL to one 128-dimensional Cholesky block and measure the
remaining gap between exact CVP and Babai. If it is below 1%, there is no paper.

### 1b. The original GPTQ idea, kept as history

**The result.** Chen et al.,
[`2507.18553`](https://arxiv.org/abs/2507.18553) (**ICLR 2026**), show that if
GPTQ runs from back to front, it is *exactly the same mathematically* as
Babai's nearest-plane algorithm. The related closest-vector lattice is defined
by the input Hessian of the layer, and GPTQ receives Babai's worst-case error
limit. Separately, [`2603.04956`](https://arxiv.org/abs/2603.04956) proves GPTQ
can be **arbitrarily far** from the best possible information limit and gives a
waterfilling rate assignment within **0.255 bits** of that limit.
[`2602.05790`](https://arxiv.org/abs/2602.05790) shows that using one general
codebook costs at most **0.11 bit**.

**Why we first thought it was open.** Two 2026 papers independently called the
standard method a weak lattice algorithm. We believed **nobody had tried LLL,
BKZ, Klein randomized rounding, or block sieving on the Hessian lattice**.
These are library calls in cryptanalysis, so the work looked easy. It was called
"the cheapest high-value item in the entire survey."

**The first planned test.** For each 128-column block, run BKZ on the
Hessian-based basis with block size 20–40 before Babai's nearest-plane pass.
At 2/3/4 bits, compare the orthogonality defect with perplexity. Cost: CPU
lattice reduction plus one normal GPTQ pass.

**A second opening we named:** waterfilling had been used only for linear
layers. Nobody had assigned different bit counts by head, layer, or position
for the **KV cache**, whose covariance is very uneven across directions.

**Nearby idea:** most systems use E8 or Leech lattices because they are easy to
decode. New records for normalized second moment (NSM) include a glued
12-dimensional lattice that beat K12 after 40 years, and SGD-designed lattices
that are best in every dimension up to 16. No LLM quantizer had used them. Test
whether lower NSM always gives lower perplexity at the same bit rate. If it
does not, that failure is the more interesting paper.

### 2. Distillation capacity gap — the target theorem already exists

> **GATED AND KILLED 2026-07-25; the old framing below was wrong.**
> Busbridge et al., *Distillation Scaling Laws*
> ([`2502.08606`](https://arxiv.org/abs/2502.08606), ICML 2025), already gives
> the named theorem in **Appendix C.1.3, "U-shape in the student error."** Two
> lemmas prove an inside optimum at `m ~ n`, and the proof ends with QED. The
> old statement that the law was only fitted, not derived, is false. Menon et
> al. also has a section called *"Why can more accurate teachers distill
> worse?"* and reports a best depth, so the claimed conflict with Menon is not
> real. A second gate, [[Temperature-Confound-Preregistration]], also stopped
> the follow-up plan: a NeurIPS 2022 paper had already tuned temperature for 18
> teachers at larger scale, and the gap remained. One idea survives: test
> whether the U-shape comes from the coupled KL objective rather than model
> capacity and disappears with a decoupled loss. That needs controls, not new
> theory.

The original observation was that a stronger teacher can make a small student
**worse**. We first called this universal and unexplained.

The old reading was: Menon et al.
([`2005.10419`](https://arxiv.org/abs/2005.10419)) says soft labels reduce label
variation and therefore predicts that every improvement in teacher quality
helps. Harutyunyan et al.
([`2301.12245`](https://arxiv.org/abs/2301.12245)) studies supervision
complexity compared with the student's neural tangent kernel (NTK) and seemed
to give the force in the other direction, but did not state the theorem. Ildiz
et al. ([`2410.18837`](https://arxiv.org/abs/2410.18837)) already finds the best
surrogate in high-dimensional ridgeless regression, a solvable capacity-gap
model that nobody had described in those words.

The old crowding search, `distillation "capacity gap" theoretical`, returned
**7 method papers and zero theory papers**. It incorrectly called Busbridge et
al. ([`2502.08606`](https://arxiv.org/abs/2502.08606), ICML 2025) a law that was
*fitted*, not proved.

The planned compute was a two-layer teacher and student plus CIFAR
ResNet-{8,20,56}. The target theorem was a student risk curve shaped like a U as
teacher capacity changes, with its best point
`p_T* ~ f(p_S, n, SNR)`. The proposed explanation balanced reduced variation
against a harder target.

This had changed the distillation decision in
[[Next-Direction-Literature-Survey]] from "industry owns it" to "methods are
owned, but theory is empty." The gate showed that the theory side was not empty.

### 3. Does one hidden ability explain a benchmark? — already done twice

> **GATED AND KILLED 2026-07-25.** The statistical idea is Horn's parallel
> analysis from 1965. Dobriban & Owen, JRSS-B 2019, and Dobriban, Annals of
> Statistics 2020, already gave the random-matrix-theory (RMT) version. Work in
> econometrics already uses the BBP transition to find the number of hidden
> factors. The exact data question is also answered: *AI Cartography*
> ([`2605.25272`](https://arxiv.org/abs/2605.25272)) runs item-level CFA/SEM on
> more than 4,000 models and six benchmarks, with permutation controls. The
> proposed plan had two more problems. A binary response matrix follows a
> generalized MP law with different variances, not the standard MP law. Also,
> putting items inside separate benchmarks guarantees a second spike. See
> [[Direction-Gate-Results]].

The original concern was that psychometric methods used for benchmark audits,
including item response theory (IRT) and mean scores, assume one hidden ability.
Two major 2026 papers ([`2511.16842`](https://arxiv.org/abs/2511.16842),
[`2605.30504`](https://arxiv.org/abs/2605.30504)) state this assumption but do
not test it.

**Original math idea:** after removing overall model ability and item
difficulty, asking whether a benchmark has one factor becomes a search for a
second eigenvalue spike above the Marchenko-Pastur bulk. The known BBP phase
change would give a best-possible detection limit based on the numbers of
models and items and the signal-to-noise ratio (SNR).

**Compute would be zero.** The study would only reanalyze published
model-by-item response tables.

The registered predictions were: P1, at least 5 of 9 benchmarks would have two
or more eigenvalues above a threshold calibrated by permutation; P2, the most
important claim, items loading on factor 2 would be unusually common among
items independently marked as problematic; and P3, at least 10% of model pairs
would swap order when changing from mean score to factor-1 score.

At the time, the IRT-for-benchmarks area had 108 papers and was adding about 20
in eight months. The estimated window was 6–12 months.

### 4. Adversarially robust streaming × KV-cache eviction

> **Re-scoped 2026-08-02.** The general KV direction rose in value only after
> narrowing it to **safety-aware allocation**: ★★★★ for the narrow version and
> ★★★ for the original. Two openings used by the old idea were taken. CONF-KV
> [`2605.24786`](https://arxiv.org/abs/2605.24786) covers long use over many
> steps. [SQuat](https://arxiv.org/abs/2503.24358),
> [KVarN](https://arxiv.org/abs/2606.03458), and
> [VeriCache](https://arxiv.org/abs/2605.17613) address errors that grow over
> time. The belief that new kernels were required was also wrong:
> [KVTuner](https://arxiv.org/abs/2502.04420),
> [EvolKV](https://arxiv.org/abs/2509.08315), and
> [SCBench](https://arxiv.org/abs/2412.10319) all published with no kernel work.
> Six searches found no allocator designed for safety. The remaining clear
> test compares the safest per-layer allocation with the allocation that gives
> the best perplexity. See [[Direction-Reevaluation-2026-08]]. The original
> adversarial-streaming proposal below never received its own full check.

Autoregressive generation changes as it runs: the cache-eviction choice affects
the next token, and that token becomes the next query. This matches the threat
model in adversarial streaming, where later input can depend on earlier choices.
Yet empirical KV-compression work—
[H2O](https://arxiv.org/abs/2306.14048),
[SnapKV](https://arxiv.org/abs/2404.14469),
[StreamingLLM](https://arxiv.org/abs/2309.17453), and
[PyramidKV](https://arxiv.org/abs/2406.02069)—reports 30–90% compression even
though Haris-Onak gives an unconditional `Theta(nd)` lower bound
([`2502.15955`](https://arxiv.org/abs/2502.15955)). **Nobody had measured what
special structure lets real models avoid the lower bound.**

Two separate searches for `"adversarially robust" AND "KV cache"` returned
**zero results**. ML papers in this area assume future queries do not react to
the algorithm's earlier choices, which is called the oblivious model.

The proposed output was an adversarial KV benchmark, a matching proof that some
performance is impossible, and a safer method with the predicted polylogarithmic
extra cost. A negative answer—"real generation does not behave adversarially
enough"—would also be new and publishable.

### 5. Smaller theory ideas that stand on their own

- **A unique formula from rules for isotropy and effective rank.** Six simple
  rules may uniquely force `(sum lambda_i)^2 / (d * sum lambda_i^2)`. IsoScore
  follows five of the six. The check found that IsoScore
  ([`2108.07344`](https://arxiv.org/abs/2108.07344)) has **zero theorem
  environments**. [Godey et al.](https://arxiv.org/abs/2401.12143),
  *Anisotropy Is Inherent to Self-Attention* (EACL 2024), also has **zero
  theorems** despite making a universal claim in its title.
  [Rudman](https://arxiv.org/abs/2305.19358) & Eickhoff (ICLR 2024) give
  experiments against the common belief that anisotropy is harmful. This has
  the best impact for the amount of background needed among representation
  geometry ideas. It needs one GPU.
- **GL(d)-equivariance of steering directions.** In
  [Park et al.](https://arxiv.org/abs/2311.03658), the causal inner product is
  **Definition 3.1, not a theorem**. The paper says D "is a free parameter with
  d degrees of freedom... We do not have a principle for picking out a unique
  choice." The proposed claim is that mean-difference steering changes under a
  reparameterization that leaves outputs the same, while a whitened rule does
  not. Testing this needs only inference.
- **Every Bit Counts → choose precision for each task.**
  [`2602.02707`](https://arxiv.org/abs/2602.02707) gives a function that can be
  computed with `p` bits but not with `p-1`. The prediction is that exact-match
  and comparison tasks fail suddenly below a bit-width threshold, while
  meaning-based tasks get worse slowly. A one-week study could produce a useful
  rule for choosing precision by task.

## Areas not to enter

**Math used mostly as decoration:** category theory in ML, which has had zero
experimental wins in seven years and no successor to its main position paper;
topological data analysis (TDA) as a general feature extractor, because
[`2507.07156`](https://arxiv.org/abs/2507.07156) gets the same performance from
models trained on **unreduced** persistence diagrams, nearly disproving the
reason for TDA; sheaf neural networks, because
[`2603.05395`](https://arxiv.org/abs/2603.05395) from March 2026 finds that an
identity-sheaf baseline with **no sheaf at all** matches every version; Kakeya,
geometric Langlands, Ramsey bounds, sunflowers, and new sphere-packing bounds;
and information geometry, because **no theorem from 2022–2026 changes what we
would do**. Information geometry also lost to plain *norm* geometry: the search
"Fisher-Rao" plus "language model" gives 6 papers, while Muon gives 292.

**Already studied heavily:** neural collapse and unconstrained feature models
(UFM), with 281 papers and a proof that deep neural collapse is not best when
`K >= 6`; learning-augmented algorithms; Muon and spectral optimizers, with at
least 18 papers and large labs; conformal methods applied to new domains, with
846 papers; LLM-as-judge bias; reward-hacking fixes, with 88 titles; and
nonlinear ICA/causal representation learning, where every named open problem
was claimed within about 12 months.

**Correction to remember:** three separate groups built the SAE-identifiability
connection in 2025–26. Do not call
it untouched. Only the phase-change geometry and computational-hardness parts
remain.

## New top idea: transformer succinctness

> **CHECKED 2026-07-25 — the main claims did not survive, but one branch did.**
> [[Direction-Gate-Results]], gate 3, gives full detail.
>
> 1. The main paper is less new than described below. Its key method goes back
>    to Meyer & Stockmeyer 1972 and the name "economy of description" from
>    1971. A full-text citation check found no Horne & Hush (NIPS 1993, matching
>    RNN size limits), Sanford/Hsu/Telgarsky (NeurIPS 2023, a transformer-vs-RNN
>    size separation), or Gelade & Neven (STACS 2008, matching tight
>    double-exponential limits). Its truly new results are the UHAT counter,
>    the exponential UHAT-to-LTL translation, and EXPSPACE-**completeness**.
> 2. The SSM/Mamba succinctness idea below was **already done and stopped**. It
>    follows in one line from Prop 1 of the paper, appears in the camera-ready
>    abstract without a proof in the body, and Jelassi et al. (ICML 2024)
>    already published a transformer-versus-SSM size difference with tests.
> 3. The **experimental** branch survives for a stronger reason than this page
>    first gave. Descriptional-complexity researchers themselves call intrinsic
>    size—parameters × precision—as an open trade-off. Measuring actual bits
>    avoids the field's unsettled definition of "SSM size." The protocol and
>    12–18 month window are in the gate page. Work beyond fixed precision is
>    open, but hard for a clear reason and now crowded.
>
> *Credit correction:* a later summary said "Lan et al., ACL 2024" was the
> main paper that had already done this. That citation does not appear anywhere
> in this wiki's gate record, so this page does not repeat it. Before citing
> either version, re-verify. The evidence currently recorded is the Meyer &
> Stockmeyer / Gelade & Neven / Jelassi chain above.

This paper was found while closing two acknowledged gaps in the survey. **At
the time, it replaced GPTQ as the top idea.**

*Transformers are Inherently Succinct* by Bergsträßer, Cotterell, and Lin
([`2510.19315`](https://arxiv.org/abs/2510.19315)) was **one of only two ICLR
2026 Outstanding Papers**. *Succinctness* asks how short a description can be
while still defining the same language. The paper presents this as a new kind
of expressive power, separate from the heavily studied question "what can the
model compute?" At fixed number precision, transformers need exponentially
less description than linear temporal logic (LTL) and RNNs, including SSMs,
and doubly exponentially less than finite automata. The paper also gives a
matching upper bound and proves that checking transformers is
EXPSPACE-complete.

**We first judged the follow-up area empty, based on two checks.** Nine months
after posting, Semantic Scholar listed **4 citations total**: one wrong, one
only loosely related, and **both real follow-ups written with members of the
paper's own group** at MPI-SWS/Kaiserslautern
(Lin/Bergsträßer/Zetzsche). A full-text arXiv search for `succinct` plus
`transformers` since October 2025 gave 8 results, all unrelated. The award
committee invited follow-up, saying the work *"may stimulate additional
theoretical **and empirical** investigation into succinctness of concept
representation by transformers and other architectures."* The committee also
mentioned unnamed "critiques," which suggested the fixed-precision assumption
was the weakest point.

**Why it first looked like the best fit.** It needs almost no compute, exactly
matching the lack of a pretraining budget, and one person can do it. The math is
automata theory, temporal logic, and complexity: building explicit pieces and
proving how much larger one description must be. It does not need concentration
inequalities, NTK, or stochastic-differential-equation (SDE) limits. For a
strong researcher who is not a probability specialist, this is much easier to
enter than grokking theory.

The named open questions were: how succinct SSMs/Mamba are compared with
transformers, since the paper mentions SSMs only through an RNN consequence;
succinctness without fixed precision; and the **experimental** question invited
by the committee. The last one best matches this group's proven skill in
measurement.

**The risk was real.** Few outside papers but an active inside group can mean
that the obvious follow-ups are already in the authors' drafts. It would also
be a move from measurement into theory, a different skill. That is another
reason the experimental branch may be the best entry.

## Grokking: the earlier decision was wrong in two ways

- **Theory is ACTIVE and OPEN, not something to avoid.** *To Grok Grokking:
  Provable Grokking in Ridge Regression*
  ([`2601.19791`](https://arxiv.org/abs/2601.19791), Xu/Vardi/Safran) received
  an **ICML 2026 Outstanding Paper Honorable Mention**, one of seven award
  spots. It proves three stages and the first numeric limits on how long
  grokking takes, but **only for linear ridge regression**. About 12 theory
  papers exist, not the 3 first reported. One person could read this small,
  understandable literature. Section 6 gives an unusually helpful open
  problem that the authors tried and could not solve: **prove lazy-to-rich
  grokking**. They say "none provided a rigorous theoretical analysis proving
  it."
- **Experiments are risky because the area is flooding.** arXiv title counts
  rose from 24 in 2022–23, to 26 in 2024, 35 in 2025, and **60 in the first
  seven months of 2026**. At least seven papers in three months entered our
  exact specialty: small-scale, multi-seed experiments that try to measure
  causes. Repeated identical typos across different submissions suggest some
  automated production.
- **The math is HARD.** The open problem needs random-features analysis that
  works across starting conditions and a proof of the change from lazy training
  to rich feature learning. This uses non-asymptotic concentration, NTK and
  feature-learning dynamics, and implicit bias. It would take 6–9 months to
  prepare and would compete with Vardi/Lee/Li, who are already working on it.

## In-context learning: the earlier "stay out" decision was correct

Annual paper counts for `in-context learning` plus `gradient descent` have
stayed near 26 for three years: 16 in 2022–23, 26 in 2024, 26 in 2025, and
about 27 per year based on 2026 so far. The main papers have 300–800 citations.
That pattern suggests the area is mature. It is also disputed now:
[`2512.04268`](https://arxiv.org/abs/2512.04268) says initialization decides
whether in-context learning (ICL) acts like gradient descent at all. Entering
would mean defending an old claim. **Stay out.** Merrill and Sabharwal also
control the TC⁰ expressivity line.

**[C-RASP](https://arxiv.org/abs/2404.04393)** was the nearby active edge: about
10 papers, almost all from 2025–26. A self-described "preliminary" nine-page
paper clearly named the link from what a model can express to what training can
learn as open.

> **Checked 2026-07-25 — a narrow opening, not a small field.** Two funded,
> prize-winning groups that work together produced about six ICML/ICLR 2026
> papers. Five named papers disprove the line "little work investigates
> learnability." Only *sample* complexity is still thin. The least-contested
> question combines length and sample complexity. The easier half is an upper
> bound; the lower bound needed for a paper is harder. One clearly named open
> problem in this area was fully solved within one conference cycle, showing
> how quickly it moves. See [[Direction-Gate-Results]], gate 4.

## Limits and errors in the surveys

1. **Search budgets ran out.** All paper-count numbers are lower bounds from
   arXiv, OpenReview, and DBLP, not complete counts.
2. One agent **withdrew its own dismissal of grokking** because it had not been
   checked. ICML 2026 gave *To Grok Grokking: Provable Grokking in Ridge
   Regression* an Honorable Mention, while provable grokking had been reported
   to have only about 3 papers. This may have been a missed idea.
3. **The survey did not cover the mechanism of in-context learning or
   chain-of-thought expressivity.** Three of four smaller searches never
   returned.
4. Several conference claims were marked as unverified. Check them again before
   using them in a paper.
5. One agent gave made-up findings in an intermediate message and then withdrew
   them. This page includes only the checked content.

## Related

[[Direction-Gate-Results]] — results of the four gates on ideas above; read it
before using any of them.
[[Unified-Direction-Ranking-2026-08]] — current direction decisions.
[[Direction-Reevaluation-2026-08]] — historical August 2 ranking and its
then-current change to the KV-cache idea.
[[Temperature-Confound-Preregistration]] — the failed follow-up gate for the
distillation capacity gap.
[[Field-Scouting-Survey]] — experiment and measurement parts.
Stage-E-Prior-Art-Audit (svib repo wiki) — the rule requiring a gate before any
run; [[Home]] gives the general version as rule 1.

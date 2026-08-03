# Math-Grounded Direction Survey

Status: **Complete 2026-07-25.** Covers the theory legs of the scouting effort:
[GeoLAN](https://arxiv.org/abs/2603.19460), representation geometry, math/TCS results with unclaimed ML surface, and
emerging low-crowding topics. Companion to [[Field-Scouting-Survey]], which
covers the empirical/measurement legs.

**Updated 2026-08-02 for the general research wiki.** Original wording is
preserved; outcome notes were added inline where a later gate changed the
verdict.

**The candidates below have since been gated, and most did not survive.** When
this page was written none had been: *"No prior-art gate has been run on any
candidate below. Per the standing rule (Stage-E-Prior-Art-Audit — svib repo
wiki), nothing here is actionable until gated."* Four gates were then run on
2026-07-25 and are recorded in [[Direction-Gate-Results]]; the KV-cache
candidate was re-scoped by [[Direction-Reevaluation-2026-08]]. **Read the gate
page before acting on anything here.** Its headline lesson: candidates die
either because the gap is one line for an insider, or because an older
literature solved it under different vocabulary — both invisible to
method-name search.

## [GeoLAN](https://arxiv.org/abs/2603.19460) is decorative theory — read this before using it as a model

Pan & Woodard, Findings of ACL 2026. Verified by reading the PDF directly.

It is a **training-time regularizer**, not a theory paper: two pre-existing
techniques (isotropy regularization via random projections, attention
spectral-entropy regularization) wrapped in Kakeya/Wolff branding.

- **No Kakeya result is invoked in any proof.** Wolff, Guth-Zahl, and [Wang-Zahl](https://arxiv.org/abs/2502.17655)
  are motivation only.
- Theorem 1 never uses its K-stickiness hypothesis.
- Grains are *defined* as connected components, so Lemma 3 is point-set topology
  true of any continuous trajectories — including an untrained model and
  including total collapse.
- The orthogonality claim is proved by noting components are disjoint, hence
  `Vol(empty set) = 0 <= C/K`. The paper itself writes "**(trivially)**".
- The Lipschitz bound is the trivial depth-composition product, exponential in
  depth and vacuous at scale; K-dependence is smuggled in by *labelling* the
  per-layer constants.
- The linear-probe result **assumes** the operator-norm bound that is its
  conclusion.
- The chain Loss → isotropy/entropy → **???** → Axioms → Theorem 1 is broken in
  the middle. **K is never measured.**
- Empirics: ~25 Cohen's d values with signs both ways, no multiple-comparison
  correction, n=4 seeds. `p=0.078` is called "significant"; `p=0.05`
  "approaches significance". MMLU deltas of `+0.75/+0.55/+0.15%` are inside
  seed noise.
- Compute: ~1.8e22 FLOPs ≈ **6,300 B200-GPU-hours**, never reported in the
  paper, far outside a 4-16 GPU budget. Advertised code repo is a stub.

**Do not use this as a template.** It is the anti-pattern: math as branding.

## The rule that actually predicts acceptance

Three independent data points from the survey:

- Xu et al.'s diagonalization-based *Hallucination is Inevitable* was **rejected
  by TMLR**; [Kalai et al.](https://arxiv.org/abs/2509.04664)'s reduction-to-a-measurable-classifier version landed.
- *[The Invisible Leash](https://arxiv.org/abs/2507.14843)* was **rejected at ICLR 2026**; the purely empirical
  version of the same claim got a **NeurIPS 2025 Oral**.
- A **single-author short proof** that GPTQ equals Babai's algorithm got an
  **ICLR 2026 Poster**.

> **The theorem must constrain a method someone is actually using, and its
> constant must be readable off a real model.**

[GeoLAN](https://arxiv.org/abs/2603.19460) fails this test on both clauses. The GPTQ result passes on both, and is
the existence proof that a small, single-author, mathematically-grounded
contribution lands in this space.

## Ranked candidates

### 1. GPTQ / lattice reduction — **GATED AND KILLED 2026-07-25**

The prior-art gate closed this comprehensively (see also
[[Direction-Gate-Results]], which counts this as the first of the five gates and
reads it as the archetype of failure mode 1 — the gap was the anchor paper's own
future-work sentence). Recorded in full because the failure modes are
instructive.

1. **It is the literal stated future-work sentence of the anchor paper**, and of
   a second one. Chen et al. write: "open the door to importing decades of
   progress in lattice algorithms towards the design of future quantization
   algorithms." Birnick's independent proof ([`2508.01077`](https://arxiv.org/abs/2508.01077), also ICLR 2026)
   closes by "hinting at the possibility of using lattice basis reduction for
   improved quantization." Two groups have publicly planted the flag.
2. **Klein randomized rounding is already published.** OJBKQ ([`2602.08376`](https://arxiv.org/abs/2602.08376),
   Feb 2026) applies box-constrained Babai plus extended Klein with K-best
   residual selection to LLM post-training quantization.
3. **The OJBKQ authors are Xiao-Wen Chang's group** — the canonical
   LLL-for-integer-least-squares people. They had LLL/BKZ fully in hand, named
   reduction as the textbook remedy in their own related work, and deliberately
   used Klein instead. That is an informed negative signal, not an oversight.
4. **The box constraint structurally forbids full reduction.** Reduction applies
   a unimodular `U`, which maps the required box `{0..2^b-1}^n` to a skewed
   parallelepiped. The box-constrained integer-least-squares literature already
   knows this and can use only the column-permutation part of LLL.
5. **Even the box-legal part is predicted to hurt here.** Wen & Chang prove that
   when noise is relatively large, LLL-P *decreases* Babai's success
   probability. 2-3 bit quantization is squarely the large-noise regime.
6. **GPTQ already ships the box-legal part** as `act_order`/`desc_act`.
7. **Measured headroom is ~0.03 perplexity.** Klein K-best is a strictly
   stronger CVP solver than Babai and buys `9.38 -> 9.35` on 4-bit Llama3-8B,
   saturating by K=5. Most of OJBKQ's gain came from the objective, not the
   solver.
8. **The deepest objection: reduction changes the basis, not the lattice.**
   Exact CVP is basis-independent, so reduction can only close the
   Babai-to-exact-CVP gap. WaterSIC pins the residual gap *at* exact CVP to
   `0.255` bits and attributes it to the suboptimality of the integer lattice
   for vector quantization — a **codebook space-filling** problem that
   reduction structurally cannot touch.

Only surviving door: WaterSIC discards the box constraint (unbounded integers
plus entropy coding), under which full unimodular reduction becomes admissible
and obstructions 4-5 dissolve. But that means competing with
Ordentlich/Polyanskiy on their own turf for at most 0.255 bits, most of which
reduction cannot reach. **Decision rule if ever revisited:** a two-hour check —
LLL-reduce one 128-dim Cholesky block, measure the exact-CVP-versus-Babai
residual gap. If under 1%, there is no paper.

### 1b. Original entry, retained for the record

**The result.** Chen et al., [`2507.18553`](https://arxiv.org/abs/2507.18553) (**ICLR 2026**): run back-to-front,
GPTQ is *mathematically identical* to Babai's nearest-plane algorithm for the
closest-vector problem on the lattice defined by the layer's input Hessian, and
inherits Babai's worst-case error bound. Independently, [`2603.04956`](https://arxiv.org/abs/2603.04956) proves GPTQ
can be **arbitrarily far** from the information-theoretic limit and gives a
waterfilling rate allocation provably within **0.255 bits** of it; [`2602.05790`](https://arxiv.org/abs/2602.05790)
shows the price of a universal codebook is at most **0.11 bit**.

**Why it is open.** Two 2026 results independently say the standard method is a
weak lattice algorithm. **Nobody has run LLL, BKZ, Klein randomized rounding, or
block sieving on the Hessian lattice** — these are library calls in
cryptanalysis. Difficulty: low. Called "the cheapest high-value item in the
entire survey."

**Concrete attack.** BKZ-reduce the Hessian-derived basis per 128-column block
(block size 20-40) before the nearest-plane pass; measure orthogonality defect
against perplexity at 2/3/4-bit. Cost: CPU lattice reduction plus one normal
GPTQ pass.

**Second, equally open target:** waterfilling has been done for linear layers
only. Nobody has done per-head/per-layer/per-position bit allocation for the
**KV cache**, where covariance is wildly anisotropic.

**Adjacent:** everyone uses E8 or Leech because the decoders are nice. Recent
normalized-second-moment records (glued 12-D dethroning K12 after 40 years;
SGD-designed lattices optimal in all dims <= 16) have **never been put in an LLM
quantizer**. Test whether perplexity is monotone in NSM at fixed rate; if it
breaks, that is the better paper.

### 2. Distillation capacity gap — a published theorem contradicts universal observation

> **GATED AND KILLED 2026-07-25; the framing here was factually wrong.** The
> target theorem exists and is named: Busbridge et al., *Distillation Scaling
> Laws* ([`2502.08606`](https://arxiv.org/abs/2502.08606), ICML 2025), **Appendix C.1.3, "U-shape in the student
> error"** — two lemmas, an interior optimum at `m ~ n`, closed with a QED. The
> claim below that it is "*fitted*, not derived" is wrong and is corrected in
> [[Direction-Gate-Results]]. The motivating contradiction also does not exist:
> Menon et al. contains a subsection titled *"Why can more accurate teachers
> distill worse?"* and reports an optimal depth. A second gate,
> [[Temperature-Confound-Preregistration]], separately failed the follow-up
> protocol — per-teacher temperature tuning was already run at larger scale
> (18 teachers, NeurIPS 2022) and does not remove the gap. Survivor: the
> *deflationary* paper — that the U-shape is a property of the coupled KL
> objective rather than of capacity, and dissolves under a decoupled loss.
> That one needs controls, not theory.

A stronger teacher can make a small student **worse**. Universally observed,
no theory.

Menon et al. ([`2005.10419`](https://arxiv.org/abs/2005.10419), soft labels reduce label variance) predicts
**monotone** improvement in teacher quality, flatly contradicting the observed
U-shape. Harutyunyan et al. ([`2301.12245`](https://arxiv.org/abs/2301.12245), supervision complexity w.r.t. the
student's NTK) supplies the opposing force but never states the theorem. Ildiz
et al. ([`2410.18837`](https://arxiv.org/abs/2410.18837)) already characterizes the optimal surrogate in
high-dimensional ridgeless regression — a capacity-gap statement in a solvable
model that nobody framed as one.

**Crowding:** `distillation "capacity gap" theoretical` returns **7 papers, all
methods, zero theory**. Busbridge et al. ([`2502.08606`](https://arxiv.org/abs/2502.08606), ICML 2025) is a
*fitted*, not derived, law.

**Compute:** two-layer teacher-student plus a CIFAR ResNet-{8,20,56} grid.

**Target theorem:** student risk is U-shaped in teacher capacity with minimizer
`p_T* ~ f(p_S, n, SNR)`, from variance-reduction versus target-complexity.

Note this reframes the distillation direction previously rejected in
[[Next-Direction-Literature-Survey]] as industry-dominated: **the method side is
owned, the theory side is empty.** (The gate above shows the theory side was
not empty either.)

### 3. Benchmark unidimensionality as a spiked-covariance / BBP problem

> **GATED AND KILLED 2026-07-25 — scooped twice.** Statistically this is Horn's
> parallel analysis (1965), with the RMT replacement already published
> (Dobriban & Owen, JRSS-B 2019; Dobriban, Ann. Stat. 2020) and factor-number
> determination via BBP settled in econometrics. Empirically it is already
> answered on the same data: *AI Cartography* ([`2605.25272`](https://arxiv.org/abs/2605.25272)) runs item-level
> CFA/SEM over 4,000+ models and six benchmarks with permutation controls. Two
> technical landmines make the pre-registration below ill-posed anyway: a
> binary response matrix gives a generalized MP law with a variance profile,
> not standard MP, and item nesting within benchmarks guarantees a second
> spike. Detail: [[Direction-Gate-Results]].

Psychometric methods now used to *audit* benchmarks (IRT, mean-score summaries)
assume a unidimensional latent construct, and that assumption is nowhere tested.
Two high-profile 2026 papers ([`2511.16842`](https://arxiv.org/abs/2511.16842), [`2605.30504`](https://arxiv.org/abs/2605.30504)) state it explicitly
and do not test it.

**Math core:** remove ability and difficulty main effects and "is this benchmark
unidimensional?" becomes detecting a second spike above the Marchenko-Pastur
bulk — a problem with a **known BBP phase transition** and therefore a
ready-made minimax threshold in (#models, #items, SNR).

**Compute: zero.** Pure re-analysis of published model-by-item response
matrices.

**Pre-registrable:** P1, at least 5 of 9 benchmarks show >= 2 eigenvalues above
a permutation-calibrated threshold. P2 (load-bearing), factor-2-loading items are
enriched among independently flagged problematic items. P3, at least 10% of
model pairs reverse order between mean score and factor-1 score.

**Window:** IRT-for-benchmarks is at 108 papers and accelerating (~20 in 8
months). 6-12 months.

### 4. Adversarially robust streaming x KV-cache eviction

> **Re-scoped 2026-08-02.** The KV direction as a whole moved up — but only
> when narrowed to **safety-aware allocation** (★★★★ narrowed / ★★★ as
> originally scoped). Two of the slots this section relies on have since been
> taken: the long-horizon slot is claimed (CONF-KV [`2605.24786`](https://arxiv.org/abs/2605.24786)) and
> compounding error is remedied three ways ([SQuat](https://arxiv.org/abs/2503.24358)/[KVarN](https://arxiv.org/abs/2606.03458)/[VeriCache](https://arxiv.org/abs/2605.17613)). The kernel
> objection was also refuted — [KVTuner](https://arxiv.org/abs/2502.04420), [EvolKV](https://arxiv.org/abs/2509.08315) and [SCBench](https://arxiv.org/abs/2412.10319) published with zero
> kernel work. What is unclaimed is a safety-objective allocator (six searches,
> zero hits), with one falsifiable sweep: safety-optimal versus
> perplexity-optimal per-layer allocation. See
> [[Direction-Reevaluation-2026-08]]. The adversarial-streaming framing below
> was never gated on its own.

Autoregressive decoding is **definitionally adaptive**: the eviction policy's
own decisions determine the next token, which becomes the next query. That is
exactly the adversarial-streaming threat model. Meanwhile the empirical
KV-compression literature ([H2O](https://arxiv.org/abs/2306.14048), [SnapKV](https://arxiv.org/abs/2404.14469), [StreamingLLM](https://arxiv.org/abs/2309.17453), [PyramidKV](https://arxiv.org/abs/2406.02069)) reports 30-90%
compression against Haris-Onak's unconditional `Theta(nd)` lower bound
([`2502.15955`](https://arxiv.org/abs/2502.15955)), and **nobody has measured which structural exemption is doing
the work**.

Two independent searches for `"adversarially robust" AND "KV cache"` returned
**zero hits**. The ML-side work is entirely in the oblivious model.

Deliverable: an adversarial KV benchmark with a matching impossibility proof,
plus a robustification fix at the predicted polylog overhead. A negative result
("real decoding isn't adversarial enough") is also publishable and currently
unknown.

### 5. Smaller, self-contained theory targets

- **Axiomatic uniqueness for isotropy / effective-rank measures.** Six axioms
  should pin `(sum lambda_i)^2 / (d * sum lambda_i^2)` uniquely, with IsoScore
  satisfying five of six. Verified: IsoScore ([`2108.07344`](https://arxiv.org/abs/2108.07344)) has **zero theorem
  environments**; [Godey et al.](https://arxiv.org/abs/2401.12143)'s *"Anisotropy Is Inherent to Self-Attention"*
  (EACL 2024) has **zero theorems** despite a universal title claim; [Rudman](https://arxiv.org/abs/2305.19358) &
  Eickhoff (ICLR 2024) **empirically refute** the "anisotropy is harmful"
  folklore. Best impact-per-prerequisite in representation geometry. One GPU.
- **GL(d)-equivariance of steering directions.** [Park et al.](https://arxiv.org/abs/2311.03658)'s causal inner
  product is **Definition 3.1, not a theorem**, and the paper states verbatim
  that D "is a free parameter with d degrees of freedom... We do not have a
  principle for picking out a unique choice." Conjecture: mean-difference
  steering is not equivariant under output-preserving reparameterization, while
  the whitened rule is. Validation is inference-only.
- **Every Bit Counts → per-task quantization precision.** [`2602.02707`](https://arxiv.org/abs/2602.02707) gives a
  function computable with `p` bits but not `p-1`. Prediction: exact-match and
  comparison tasks have a sharp bit-width cliff; semantic tasks degrade
  gracefully. One week, yields a deployable per-task precision rule.

## Do not enter

**Decorative:** category theory in ML (zero empirical wins in seven years; the
flagship position paper has no successor); TDA as a general feature extractor
([`2507.07156`](https://arxiv.org/abs/2507.07156) finds models trained on **unreduced** persistence diagrams do as
well — close to a falsification of the premise); sheaf neural networks
([`2603.05395`](https://arxiv.org/abs/2603.05395), Mar 2026: an identity-sheaf baseline with **no sheaf at all**
matches every variant); Kakeya, geometric Langlands, Ramsey bounds, sunflowers,
new sphere-packing bounds; information geometry (**no 2022-2026 theorem changes
what you would do**; note IG lost to *norm* geometry — "Fisher-Rao" plus
"language model" returns 6 papers, Muon returns 292).

**Crowded:** neural collapse / UFM (281 publications, and deep neural collapse
was **proved non-optimal** for K >= 6); learning-augmented algorithms; Muon and
spectral optimizers (18+ papers, well-resourced labs); conformal-for-X (846);
LLM-as-judge bias; reward-hacking mitigation (88 titles); nonlinear ICA / causal
representation learning (every named open problem claimed within ~12 months).

**Correction to carry forward:** the SAE-identifiability bridge was built in
2025-26 by three independent groups. Do not pitch it as untouched; only the
phase-transition geometry and computational-hardness legs remain.

## New top candidate: transformer succinctness

> **GATED 2026-07-25 — headline claims did not survive; one branch did.**
> Full detail in [[Direction-Gate-Results]], gate 3. Three findings:
> **(i)** the anchor paper is less novel than reported here — the core
> technique dates to Meyer & Stockmeyer 1972 ("economy of description," 1971),
> and a full-text citation audit found zero occurrences of Horne & Hush
> (NIPS 1993, matching RNN size bounds), Sanford/Hsu/Telgarsky (NeurIPS 2023,
> a transformer-vs-RNN *size* separation) or Gelade & Neven (STACS 2008, tight
> matched double-exponential bounds). What is genuinely its own: the UHAT
> counter construction, the exponential UHAT-to-LTL translation, and
> EXPSPACE-**completeness**.
> **(ii)** The SSM/Mamba succinctness problem named below is **scooped and
> killed** — it is a one-line corollary of the paper's own Prop 1, asserted in
> the camera-ready abstract with no body proof, and Jelassi et al. (ICML 2024)
> published a transformer-vs-SSM size separation with experiments.
> **(iii)** The **empirical** branch survives, and is better motivated than
> this page argued: the descriptional-complexity community's own flagged open
> problem is intrinsic size measures (parameters × precision as a trade-off
> surface), and measuring bits actually used sidesteps the unsettled
> definition of "SSM size." Protocol shape and the 12–18 month window are on
> the gate page. The "beyond fixed precision" branch is open but hard for a
> principled reason and now crowded.
>
> *Attribution note:* a later summary of this outcome credited the headline
> scoop to "Lan et al., ACL 2024." No such citation appears anywhere in this
> wiki's gate record, and it is not repeated here — the scoop evidence on file
> is the Meyer & Stockmeyer / Gelade & Neven / Jelassi chain above. Re-verify
> before citing either version.

Found while closing the two acknowledged survey holes. **This displaces the
GPTQ candidate.**

*Transformers are Inherently Succinct* (Bergsträßer, Cotterell, Lin,
[`2510.19315`](https://arxiv.org/abs/2510.19315)) is **one of only two ICLR 2026 Outstanding Papers**. It introduces
*succinctness* — how compactly a formalism describes a language — as a new axis
of expressive power, orthogonal to the exhausted "what can it compute" axis.
Fixed-precision transformers are **exponentially more succinct than LTL and
RNNs** (hence SSMs) and **doubly exponentially more succinct than finite
automata**, with a matching upper bound and the consequence that transformer
verification is EXPSPACE-complete.

**The follow-up field is empty, verified two ways.** Nine months after posting,
Semantic Scholar shows **4 citations total**: one spurious, one tangential, and
**both substantive ones co-authored by the paper's own group**
(Lin/Bergsträßer/Zetzsche, MPI-SWS/Kaiserslautern). Cross-checked against a
full-text arXiv sweep for `succinct` + `transformers` since Oct 2025: 8 hits,
all unrelated. The award committee explicitly invited follow-up — *"may
stimulate additional theoretical **and empirical** investigation into
succinctness of concept representation by transformers and other
architectures"* — while noting unspecified "critiques," which points at the
fixed-precision assumption as the attackable soft spot.

**Why it fits better than anything else surveyed.** Near-zero compute, which
matches the no-pretraining-budget constraint exactly. Single-person scale. And
critically, the mathematics is **automata theory, temporal logic, and
complexity** — combinatorial and constructive, building gadgets and proving
blow-up lower bounds. No concentration inequalities, no NTK, no SDE limits.
That is a materially lower barrier for a strong non-probabilist than the
grokking lane.

Named open problems: quantitative succinctness of SSMs/Mamba versus
transformers (the paper reaches SSMs only as a corollary via RNNs); succinctness
beyond fixed precision; and the **empirical** succinctness question the
committee invited, which is the one that plays to this group's measured
strengths rather than against them.

**Risk to weigh honestly.** An empty *external* field with an active *in-group*
means the obvious extensions may already be in the authors' drafts. And this is
a pivot to theory from a track record built on measurement — a genuinely
different skill, which is why the empirical branch may be the right entry.

## Grokking: the earlier verdict was wrong in both directions

- **Theory: ACTIVE and OPEN, not a hazard.** *To Grok Grokking: Provable
  Grokking in Ridge Regression* ([`2601.19791`](https://arxiv.org/abs/2601.19791), Xu/Vardi/Safran) is a confirmed
  **ICML 2026 Outstanding Paper Honorable Mention** — one of 7 award slots. It
  proves three provable stages and the first quantitative grokking-time bounds,
  but **only in linear ridge regression**. Roughly 12 theory papers exist (not
  the 3 previously reported): a small, legible literature one person can read
  in full. Its §6 leaves a rare gift — a named, attempted-and-failed,
  obstruction-identified open problem: **provable lazy-to-rich grokking**, which
  the authors state "none provided a rigorous theoretical analysis proving it."
- **Empirics: a genuine hazard.** arXiv title counts go 24 (2022-23) → 26 (2024)
  → 35 (2025) → **60 in the first seven months of 2026**. The flood is landing
  precisely on this group's niche — small-scale, multi-seed, causal-measurement
  grokking studies, 7+ in three months — with hallmarks of automated production
  (an identical typo recurring across distinct submissions).
- **Math difficulty: HIGH.** The open problem is agnostic random-features
  analysis and lazy-to-rich transitions: non-asymptotic concentration,
  NTK/feature-learning dynamics, implicit bias. Six to nine months of ramp, in
  Vardi/Lee/Li territory, and they are actively on it.

## In-context learning: the earlier dismissal was correct, now with evidence

`in-context learning` + `gradient descent` counts are **flat at ~26/year for
three years** (16 in 2022-23, 26 in 2024, 26 in 2025, ~27 annualized in 2026)
against foundational papers cited 300-800 times. That is the signature of a
worked-out area. It is also now actively contested ([`2512.04268`](https://arxiv.org/abs/2512.04268) argues
initialization determines whether ICL is gradient descent at all), so entering
means defending an old claim. **Stay out.** The TC⁰ expressivity line is
likewise annexed by Merrill and Sabharwal. **[C-RASP](https://arxiv.org/abs/2404.04393)** is the one adjacent live
frontier (~10 papers, nearly all 2025-26), with the expressivity-to-learnability
bridge explicitly named as open in a self-described "preliminary" 9-page paper.

> **Gated 2026-07-25 — thin spot, not thin field.** The C-RASP learnability
> candidate found a real seam and mislocated it. Two funded, prize-winning,
> mutually collaborating groups placed roughly six ICML/ICLR 2026 papers
> between them, and the framing claim ("little work investigates learnability")
> is contradicted by five named papers. What is actually thin is *sample*
> complexity specifically; the least-contested seam is joint length × sample
> complexity, with the honest caveat that the accessible half is the upper
> bound and the paper-making half is the lower bound. Tempo warning on the
> page: an explicitly named open problem here was fully resolved inside one
> conference cycle. [[Direction-Gate-Results]], gate 4.

## Caveats the surveys raised about themselves

1. **Search budgets were exhausted**, so all crowding counts are lower bounds
   from arXiv/OpenReview/DBLP rather than exhaustive.
2. One agent **retracted its own grokking dismissal**: never verified, and ICML
   2026 gave an Honorable Mention to *To Grok Grokking: Provable Grokking in
   Ridge Regression* while provable grokking has ~3 papers total. Possibly a
   missed candidate.
3. **In-context-learning mechanism and CoT expressivity were not scouted** —
   three of four sub-sweeps never returned.
4. Several venue attributions were flagged unverified and must be re-checked
   before appearing in any writeup.
5. One agent reported fabricated findings in an intermediate message and
   retracted them. Only the verified content is recorded here.

## Related

[[Direction-Gate-Results]] — the four gates run on the candidates above; read
this before acting on any of them.
[[Direction-Reevaluation-2026-08]] — current direction ranking; re-scopes the
KV-cache candidate.
[[Temperature-Confound-Preregistration]] — the failed gate on the distillation
capacity-gap follow-up.
[[Field-Scouting-Survey]] — the empirical/measurement legs.
Stage-E-Prior-Art-Audit (svib repo wiki) — the standing rule requiring a gate
before any run; the general form of it is rule 1 on [[Home]].

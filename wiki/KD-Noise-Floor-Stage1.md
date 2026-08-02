# Are Published Method Orderings Resolvable? — Pre-Registration

*(Filename is legacy — this began as a distillation-only Stage 1 and is now a
two-literature study. Kept for link integrity.)*

*Updated 2026-08-02 for the general research wiki.*

Later development: see [[Direction-Reevaluation-2026-08]], whose meta-lesson 2
("method spaces saturate in months; arbitration/diagnostic spaces stay empty")
bears on the class of work this protocol belongs to. **It does not reverse the
suspension** — the objection below is about audience and incentives, not about
emptiness — but it is why the wiki's standing rules now pair every arbitration
with a released artifact ([[Home]], rules 3-4 and 6).

Status: **SUSPENDED 2026-07-26 on owner objection. Do not proceed without
resolving the two problems below.** The measurements remain verified and the
protocol is sound; the difficulty is audience and incentives, which no
prior-art gate tests.

## The two blocking objections (owner-raised, and both correct)

**1. The finding is not information — the field already knows.** This is the
serious one. The evidence is inside the protocol itself:

- **QTIP's checklist states the convention openly.** I had filed this as
  *evidence for* the thesis. It is equally evidence that the thesis is common
  knowledge: the field discloses the practice in a mandated form and continues.
- **SparseGPT and Wanda both measured seed variance, reported it, and shipped
  single-run comparison tables anyway.** They knew.
- **Williams & Aletras published the full study in ACL 2024** — 1,800 models,
  19,800 evaluations — and the practice did not change.
- **Bouthillier et al. gave the field `P(A > B)` in MLSys 2021.** Five years.

So the norm is known, disclosed, measured, published against, and continued.
A paper arguing "the field should know" therefore changes nothing, because
**the barrier is not knowledge, it is incentives**: reporting variance makes
results look less impressive and nothing forces the change. What would actually
move it is infrastructure that makes variance reporting nearly free, or a venue
policy — neither of which is a paper a single researcher writes.

**2. Reviewer conflict of interest.** An audit of N papers draws reviewers from
the authors of those papers. The session's own evidence supports the concern:
SWE-Bench+ was withdrawn from ICLR 2026 and rejected from ICLR 2025 before
landing at AIWare 2026; Xu et al.'s diagonalization-based hallucination paper
was rejected by TMLR while a friendlier formulation landed; *The Invisible
Leash* was rejected at ICLR 2026 while its purely empirical twin took a NeurIPS
Oral.

This objection is real but **weaker than the first**, and bounded. Audit papers
do land — MaxCut-Bench, FrontierCO (ICLR 2026), *The dark side of the forces*
(ICML 2025 Oral), *Faults in Our Formal Benchmarking* (ICML 2026), *Token
Pruning: Are We Solving the Right Problem?* (ACL Findings 2025), *Revisiting
RaBitQ and TurboQuant* (overturning a Google result). The pattern among
survivors is that they ship a **tool, benchmark, or positive protocol**, and
frame against standards rather than against papers. Objection 1 is what actually
kills this.

## What this means for the session's record

Fourteen candidates. Thirteen died at prior-art gates. This one survived every
gate and then failed on **audience and incentive grounds that no gate tests**.
That is a finding about the search process, not about this protocol: novelty
screening cannot tell you whether anyone will act on a true result.

**Any future protocol must answer, before drafting: who changes their behaviour
if this is true, and what stops them today?** If the answer is "the field
already knows and the incentives are unchanged," the work is not worth doing
regardless of how well it gates.

---

## Original protocol below, retained. Measurements remain valid and verified.

Written 2026-07-26 after thirteen gates left this as the sole surviving
direction. Three open items blocked locking.

## The claim

> Across two independent subfields, published method orderings are frequently
> **unresolvable at the variance those subfields' own papers report** — and the
> review process does not check, by explicit convention rather than oversight.

Zero compute. Every number comes from published PDFs. Every step is
reproducible by a reader with the same PDFs.

## Why two case studies rather than one

A single-literature version invites "this is one group's practice." Two
independent literatures — LLM knowledge distillation and LLM post-training
compression — with **different methods, different authors, different venues, and
different metrics**, showing the same pattern, makes the subject the norm rather
than the practitioners.

---

## Case study A — knowledge distillation (VERIFIED)

**The variance estimate.** DistiLLM-1 (`2402.03898`, ICML 2024) publishes mean
and standard deviation over five seeds for **GPT-4 Eval**, not merely ROUGE-L.
Parsed directly from Tables 11-13 (GPT-2, OPT, OpenLLaMA2 families), taking the
`GPT-4 Eval` columns for Dolly, Self-Instruct and Vicuna: **52 data rows, 156
standard deviations.**

| Statistic | Value |
|---|---:|
| n | 156 |
| min | 0.02 |
| **median** | **0.46** |
| mean | 0.55 |
| max | 1.83 |
| fraction `> 0.21` | **75.6%** (118/156) |

Reproduction recipe: extract Tables 11-13 from `2402.03898`, match
`(\d+\.\d+)\s*\(\s*(\d+\.\d+)\s*\)`, keep rows with exactly 8 matched cells,
take columns 0, 2, 4.

**The comparison point.** DistiLLM-2 (`2503.07067`, **ICML 2025 Oral**) orders
GKD `56.14` below DistiLLM `56.35` — a **0.21** gap — from a **single unseeded
run**, with student sampling at temperature 0.8 / top-p 0.95 and the judge at
temperature 0.7. Zero variance reporting anywhere in the camera-ready.

**Critical scope limit.** DistiLLM-1's five seeds are **eval-time decoding
seeds** with the model held fixed. They therefore **exclude training-seed
variance** and are a **lower bound** on the total variance relevant to comparing
two independently trained methods. This must be stated wherever the number
appears.

---

## Case study B — post-training compression (VERIFIED)

**The variance estimate is stronger here**, because it is the right component.
Wanda (`2306.11695`) Appendix D.2, Table 18 reports mean±std over **five
calibration-draw seeds** — i.e. across *different compressed models*, which is
exactly the variance relevant to a method comparison. SparseGPT (`2301.00774`)
independently reports `13.52 ± 0.075` over five data seeds and concludes it is
"quite robust to the precise calibration data being used."

**The result falls out with arithmetic.** From Wanda Table 18, SparseGPT versus
Wanda, four of twelve published comparison cells are coin flips:

| Setting | SparseGPT | Wanda | P(inversion) |
|---|---:|---:|---:|
| LLaMA-7B 50% | 7.25±0.03 | 7.25±0.01 | **~0.50** |
| LLaMA-13B 4:8 | 7.43±0.03 | 7.43±0.03 | **~0.50** |
| LLaMA-2-13B 2:4 | 8.28±0.05 | 8.28±0.02 | **~0.50** |
| LLaMA-7B 4:8 | 8.67±0.08 | 8.65±0.01 | **~0.40** |

Remaining eight cells resolve cleanly. **Report all twelve** — the denominator
is part of the finding.

**Larger-scale corroboration.** Williams & Aletras (ACL 2024, `2311.09755`) run
1,800 compressed models and 19,800 evaluations, reporting across-draw std of
0.1-0.6pp on aggregate accuracy with single-task ranges up to 9pp, plus the
directly relevant observation that *"a seemingly robust perplexity of
12.72±0.18"* coexists with BoolQ accuracy spanning 57.0% to 71.6%.

---

## Direct evidence about the norm itself

Two documentary artifacts, both public and quotable, neither requiring any
reproduction. **These are the study's distinctive contribution** — every other
element is arithmetic on published tables.

**1. A public review record showing the check was never made.** ICML 2025
submission 5637, scores **4 / 3 / 4 / 4**, Accept (Oral). Across four reviews
and the AC meta-review there is **not one mention** of seeds, variance, error
bars, statistical significance, or run-to-run noise. All four reviewers
affirmatively certified the evidence — *"clear and convincing evidence"*
(nZhc), *"clear and convincing evidences"* (F2H5), *"sound and valid"* (wyun),
*"Supported Claims"* (WSDW) — and the AC called the evaluation *"extensive"* and
*"well justified."*

**These were engaged reviewers.** F2H5 caught a real technical problem — the
first-order Mercator approximation assumes `p(y|x) -> 1`, which fails as
per-token probabilities compound over long sequences — and demanded empirical
evidence, receiving source code and token-level quartiles in reply. WSDW
requested speculative-decoding acceptance rates and got them.

**The asymmetry is the finding.** One reviewer demanded empirical validation of
a series approximation; none asked whether a 0.21-point margin was
distinguishable from noise. WSDW wrote *"I'm not 100% sure I'm super qualified
to look thru this"* about the proofs. **Reviewers worried about their competence
to check the mathematics, and nobody worried about competence to check the
statistics.**

**2. A second literature stating the convention outright, in a mandated
disclosure.** QTIP's NeurIPS reproducibility checklist, verbatim:

> *"Answer: No. Justification: **It is standard practice in LLM quantization
> papers to not report error bars on metrics.**"*

That is not inference from absence. It is the field describing its own norm in a
form designed to surface exactly this question.

---

## Endpoints

**Primary.** For each harvested published comparison `(m, n)`, the resolvability
statistic **`P(A > B)`** — *"the probability of measuring a better performance
for A than B across fluctuations"* — **of Bouthillier et al., MLSys 2021
(`2103.03098`)**, with their threshold `gamma = 0.75`. A claimed ordering is
**unresolved** when `P(A > B) < 0.75`.

**Attribution is mandatory and non-negotiable.** This statistic is Bouthillier's.
It must be cited as theirs and must not be renamed. Reinventing it as "inversion
probability" already cost one prior-art gate.

**Variance sourcing, with a required labelling rule.** Every comparison is
labelled by how its variance was obtained:

- **Direct** — the paper or an immediate sibling reports mean±std for the same
  metric and setting (e.g. Wanda Table 18, SparseGPT's seed ablation).
- **Imputed** — no variance reported; the nearest same-literature estimate is
  substituted (DistiLLM-1 for KD win rates, Williams & Aletras for compression).

**Direct and imputed comparisons are reported separately and never pooled.**
Headline claims rest on *direct* only. Imputed comparisons are descriptive.

**Difference variance.** Where two methods are independently seeded,
`sigma_diff = sqrt(sigma_A^2 + sigma_B^2)`. For Case A, since the available
`sigma` covers decoding only, the resulting `P(A > B)` is an **upper bound on
resolvability** — the true figure is worse. State this at every use.

**Secondary, descriptive.** Fraction of harvested papers reporting any variance;
fraction stating which calibration or decoding seeds were used; and the
distribution of published deltas against the applicable noise scale.

---

## Harvest protocol — the only remaining work, and it is pre-specified

1. **Declare the paper list before any extraction**: venue set and date range
   fixed in advance, restricted to papers whose headline table compares two or
   more methods within one of the two case-study families.
2. **Headline/main-results table only.** Never an appendix table selected after
   seeing results.
3. **Report the full denominator**, including comparisons that resolve cleanly.
   Selective reporting here would be precisely the failure the paper diagnoses,
   and a reviewer will say so.
4. Where a paper reports its own variance, that supersedes any imputation.
5. Extraction is mechanical and scripted; the script and its output ship with
   the paper.

---

## Decision gate

**Write it** if, among **direct-variance** comparisons, a named set of published
orderings is unresolved at `gamma = 0.75`. Four such cells already exist in Case
B before any harvesting, so this is partly de-risked.

**Write a narrower version** if unresolved orderings appear only under imputed
variance. The claim then concerns reporting standards rather than specific
published conclusions — weaker, still honest.

**Report the defense of the field** if the harvest shows published deltas
comfortably clearing the applicable noise in the large majority of cases. That
is a real and publishable outcome: *the field's margins are adequate, and here
is the measurement establishing it.* Pre-declared so it cannot become a
disappointment.

---

## What may not be claimed

- **Not** that variance is unmeasured in these literatures. It is measured —
  MiniLLM (5 seeds), GKD (3), DistiLLM (5 with std), SparseGPT (5), Wanda (5),
  Williams & Aletras (10 draws x 5 sources). The failure is that it is **not
  used when orderings are asserted**.
- **Not** that calibration or seed choice is a novel concern. Williams & Aletras,
  Ji et al. and Bouthillier own that ground.
- **Not** that `P(A > B)` is new. See attribution above.
- **Not** any accusation of poor practice by any author. In Case A the authors
  passed unanimous peer review under the standard the field applied; in Case B
  the papers **do** report seed variance in their appendices. **The subject is
  the convention, not the practitioners.**
- **Not** that these methods do not work. The claim concerns the resolvability
  of *orderings*, not the value of distillation or compression.

## Handling rules, carried forward and binding

- Framing is literature-wide throughout; DistiLLM and Wanda/SparseGPT are case
  studies, never targets.
- **The numerical irregularities recorded in [[Compression-Audit-Direction]]
  stay out entirely.** With a review record showing careful reviewers who simply
  never looked at variance, adding table-integrity observations would convert a
  clean norm-level finding into an author-level accusation.
- Notify the authors of both case-study papers privately before submission.

## Open items blocking lock

1. **Final prior-art gate**, executed under the standing rule below: has anyone
   applied `P(A > B)` or an equivalent resolvability audit across a published ML
   literature? Check the reproducibility, meta-science and benchmark-reliability
   literatures by task, not by method name. Note `2605.20798` ran an
   iso-compute noise-floor audit for transformer modifications — read its
   **method section** and quote the sentence that distinguishes it.
2. Declare the harvest paper list, venue set and date range.
3. Confirm the DistiLLM-1 seed protocol wording against the camera-ready.

## Standing rule this protocol must satisfy

> Before any protocol claims a gap, the two or three nearest prior works are read
> in their **method sections**, and the specific sentence establishing the gap is
> **quoted**. A gap asserted without a quote is not a gap.

Two of thirteen gate failures — the temperature confound and the calibration
draw — died to papers already cited in the protocol and mischaracterized from
their framing rather than their methods.

## Venue

TMLR is the natural fit: its scope explicitly invites reproducibility and
analysis studies, and it accepts modest significance where evidence is
convincing. An ICBINB-style workshop is the fallback. A main-track submission
framed as discovery would be correctly rejected — this is consolidation, and the
positioning must say so.

## Related

[[KD-Evidence-Audit-Gate]] — the gate that produced Case A and its corrections.
[[Calibration-Draw-Preregistration]] — the failed compression gate that yielded
Case B and the QTIP disclosure.
[[Direction-Gate-Results]] — the thirteen-gate record and failure modes.

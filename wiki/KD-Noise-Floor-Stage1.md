# Can We Trust Published Method Rankings? — Pre-Registered Plan

*(The filename is old. This started as Stage 1 of a knowledge-distillation-only
study and later grew to cover two research areas. The filename stays so old
links keep working.)*

*Updated 2026-08-02 for the general research wiki.*

A **noise floor** is the amount a score changes because of randomness alone.
If the score difference between two methods is smaller than this floor, their
published order may not be trustworthy. **Knowledge distillation (KD)** trains
a smaller student model to copy a larger teacher.

Later update: [[Direction-Reevaluation-2026-08]] gives a useful general lesson:
new method ideas can become crowded within months, while work that checks,
compares, or diagnoses methods can remain open. This does **not** reverse the
suspension below. The problem here is not whether the topic is open. It is
whether readers will act and whether their incentives favor change. The lesson
does explain why the standing rules now require every audit to ship a useful
tool; see [[Home]], rules 3–4 and 6.

Status: **SUSPENDED 2026-07-26 after the owner's objection. Do not continue
until both problems below are solved.** The numbers have been checked, and the
study design is sound. The unsolved problems are audience and incentives, which
a search for earlier work cannot detect.

## The two problems that stop the study

### 1. This is not new information; the field already knows

This is the main problem. The plan itself contains the evidence:

- **[QTIP](https://arxiv.org/abs/2406.11235)'s checklist openly states the normal practice.** I first treated this
  as support for the paper. It also proves that the field already knows: authors
  disclose the practice in a required form and keep doing it.
- **SparseGPT and Wanda measured variation across seeds, reported it, and still
  published one-run comparison tables.** They knew.
- **Williams & Aletras published the full test at ACL 2024:** 1,800 models and
  19,800 evaluations. Practice did not change.
- **Bouthillier et al. published `P(A > B)` in MLSys 2021.** The field has had
  the needed measurement for five years.

The norm is known, openly stated, measured, and criticized, yet it continues.
A paper telling people to notice it would not add the missing piece. **The real
barrier is incentives, not knowledge.** Reporting variation can make a result
look less impressive, and no rule requires authors to do it. Useful change
would need software that makes multi-run reports almost free or a conference
policy. A single researcher cannot create either outcome merely by writing this
paper.

### 2. Likely reviewers have a conflict of interest

An audit of N papers will probably be reviewed by authors of some of those
papers. Evidence from this research session supports that concern:

- [SWE-Bench+](https://arxiv.org/abs/2410.06992) was withdrawn from ICLR 2026 and rejected from ICLR 2025 before
  appearing at AIWare 2026.
- [Xu et al.](https://arxiv.org/abs/2401.11817)'s diagonalization-based hallucination paper was rejected by TMLR,
  while a friendlier version was published.
- *The Invisible Leash* was rejected at ICLR 2026, while a purely experimental
  twin became a NeurIPS Oral.

This problem is real but **less important than the first**, and it has limits.
Audit papers do get published: [MaxCut-Bench](https://arxiv.org/abs/2406.11897), [FrontierCO](https://arxiv.org/abs/2505.16952) (ICLR 2026),
*[The dark side of the forces](https://arxiv.org/abs/2412.11569)* (ICML 2025 Oral), *[Faults in Our Formal
Benchmarking](https://arxiv.org/abs/2606.29493)* (ICML 2026), *Token Pruning: Are We Solving the Right
Problem?* (ACL Findings 2025), and *Revisiting RaBitQ and TurboQuant*. The
successful papers usually release a **tool, benchmark, or better process** and
compare practice with a standard rather than attacking papers. Problem 1 is
what truly stops this study.

## Lesson for future research searches

Fourteen ideas were considered. Thirteen failed checks of earlier work. This
one passed every such check but then failed because **the audience already knew
and had no reason to change**. That is a lesson about the search method, not
about whether the statistical plan is correct. A novelty search cannot say
whether anyone will act on a true result.

**Before writing any future plan, answer two questions: who changes their
behavior if the result is true, and what stops them now?** If people already
know and incentives still favor the old behavior, the study is not worth doing,
even if the idea is new and the experiment is clean.

---

## Original plan below — kept as a record

The measurements remain correct and checked. This plan was written 2026-07-26
after thirteen gates left it as the only surviving direction. Three items still
blocked the final lock.

## Original claim

> In two separate research areas, published method rankings often cannot be
> separated from the amount of variation reported by those fields' own papers.
> The peer-review process does not check this, because the field openly accepts
> the convention rather than simply overlooking it.

The study needs no new computing. Every number comes from published PDFs, and
any reader can repeat every step using those PDFs.

## Why use two research areas?

A study of one area can be dismissed as one research group's habit. The two
planned examples—LLM knowledge distillation and post-training LLM
compression—use **different methods, authors, venues, and measurements**. If
both show the same problem, the target is a field-wide norm rather than named
researchers.

## Case A: knowledge distillation — CHECKED

### Estimate of random variation

DistiLLM-1 ([`2402.03898`](https://arxiv.org/abs/2402.03898), ICML 2024) reports mean and standard deviation over
five seeds for **GPT-4 Eval**, not only ROUGE-L. Tables 11–13 cover GPT-2, OPT,
and OpenLLaMA2 model families. Taking the `GPT-4 Eval` columns for Dolly,
Self-Instruct, and Vicuna gives **52 data rows and 156 standard deviations**.

| Statistic | Value |
|---|---:|
| number of values | 156 |
| minimum | 0.02 |
| **median** | **0.46** |
| mean | 0.55 |
| maximum | 1.83 |
| fraction `> 0.21` | **75.6%** (118/156) |

To reproduce the extraction, read Tables 11–13 in [`2402.03898`](https://arxiv.org/abs/2402.03898), match
`(\d+\.\d+)\s*\(\s*(\d+\.\d+)\s*\)`, keep rows with exactly 8 matched
cells, and take columns 0, 2, and 4.

### Published comparison

DistiLLM-2 ([`2503.07067`](https://arxiv.org/abs/2503.07067), **ICML 2025 Oral**) puts GKD at `56.14` and
DistiLLM at `56.35`, only **0.21 points apart**. This came from **one run with
no stated seed**. The student samples with temperature 0.8 and top-p 0.95; the
judge samples with temperature 0.7. The final paper reports no variation.

### Important limit

DistiLLM-1's five seeds change **only evaluation-time decoding**. The trained
model stays fixed. They do not include differences from training seeds and
therefore give a **lower bound** on all the variation that matters when two
separately trained methods are compared. This limit must appear every time the
number is used.

## Case B: post-training compression — CHECKED

Here the variation estimate is stronger because it measures the right source.
Wanda ([`2306.11695`](https://arxiv.org/abs/2306.11695)) Appendix D.2, Table 18 reports mean ± standard deviation
over **five random calibration samples**. Each sample creates a different
compressed model, exactly matching the variation that affects a method
comparison. SparseGPT ([`2301.00774`](https://arxiv.org/abs/2301.00774)) independently reports `13.52 ± 0.075`
over five data seeds and says it is robust to the exact calibration data.

Simple arithmetic on Wanda Table 18 shows that four of twelve published
SparseGPT-versus-Wanda cells are close to coin flips:

| Setting | SparseGPT | Wanda | chance the order reverses |
|---|---:|---:|---:|
| LLaMA-7B 50% | 7.25±0.03 | 7.25±0.01 | **about 0.50** |
| LLaMA-13B 4:8 | 7.43±0.03 | 7.43±0.03 | **about 0.50** |
| LLaMA-2-13B 2:4 | 8.28±0.05 | 8.28±0.02 | **about 0.50** |
| LLaMA-7B 4:8 | 8.67±0.08 | 8.65±0.01 | **about 0.40** |

The other eight cells are clearly separated. **Report all twelve.** The full
count is part of the result.

Williams & Aletras (ACL 2024, [`2311.09755`](https://arxiv.org/abs/2311.09755)) provide a larger check: 1,800
compressed models and 19,800 evaluations. The standard deviation across random
calibration samples is 0.1–0.6 percentage points for average accuracy, while
individual tasks span as much as 9 points. A seemingly stable perplexity of
`12.72±0.18` occurs alongside [BoolQ](https://arxiv.org/abs/1905.10044) accuracy from 57.0% to 71.6%.

## Direct evidence about the field's normal practice

These two public records require no reproduction. **They were meant to be the
study's special contribution.** Everything else is arithmetic on published
tables.

### 1. Public reviews show that nobody asked for this check

ICML 2025 submission 5637 received scores **4 / 3 / 4 / 4** and was accepted
as an Oral. Across four reviews and the area chair's meta-review, nobody
mentions seeds, variation, error bars, statistical significance, or noise
across runs. All reviewers said the evidence supported the claims: "clear and
convincing evidence" (nZhc), "clear and convincing evidences" (F2H5), "sound
and valid" (wyun), and "Supported Claims" (WSDW). The area chair called the
evaluation "extensive" and "well justified."

These reviewers were paying attention. F2H5 found a real technical problem:
the first-order Mercator approximation assumes `p(y|x) -> 1`, but probabilities
for each token multiply across long sequences, so the assumption can fail. The
reviewer asked for evidence and received code and token-level quartiles. WSDW
asked for speculative-decoding acceptance rates and received them.

The difference in scrutiny was the planned finding. A reviewer demanded a test
of a series approximation, but nobody asked whether a 0.21-point score gap was
larger than random variation. WSDW wrote, "I'm not 100% sure I'm super
qualified to look thru this" about the proofs. **Reviewers questioned their
ability to check the mathematics, but nobody questioned the statistical check.**

### 2. Another field states the convention openly

[QTIP](https://arxiv.org/abs/2406.11235)'s required NeurIPS reproducibility checklist says:

> *"Answer: No. Justification: **It is standard practice in LLM quantization
> papers to not report error bars on metrics.**"*

This is not a guess from missing information. The field states its own practice
in a form meant to reveal exactly this issue.

## Measurements planned in advance

### Main measurement

For each published comparison `(m, n)`, use **`P(A > B)`** from Bouthillier et
al., MLSys 2021 ([`2103.03098`](https://arxiv.org/abs/2103.03098)). It means the chance that method A measures
better than method B when random factors change. Bouthillier et al. use
`gamma = 0.75`; a claimed order is **unresolved** when `P(A > B) < 0.75`.

**Always credit Bouthillier et al. and keep their name for the statistic.** An
earlier plan renamed it "inversion probability," which helped cause a failed
earlier-work check.

### Where each variation estimate comes from

Every comparison must have one of two labels:

- **Direct:** the paper or an immediate related paper reports mean ± standard
  deviation for the same measurement and setting. Examples are Wanda Table 18
  and SparseGPT's seed test.
- **Imputed:** the paper reports no variation, so use the nearest estimate from
  the same research area. Examples are DistiLLM-1 for KD win rates and Williams
  & Aletras for compression.

**Report direct and imputed results separately. Never combine them.** Main
claims can use direct measurements only. Imputed results are descriptive.

When two methods use independent seeds, the standard deviation of their
difference is
`sigma_diff = sqrt(sigma_A^2 + sigma_B^2)`. For Case A, available `sigma`
values cover decoding only. Its `P(A > B)` is therefore an **upper bound on how
well the methods can be separated**. Including training randomness would make
the true result worse. State this at every use.

Secondary descriptions include the fraction of papers reporting any
variation, the fraction naming calibration or decoding seeds, and the
distribution of published score differences compared with the relevant noise
scale.

## Fixed rules for collecting papers and numbers

1. List all papers before extracting numbers. Fix the venues and date range.
   Include only papers whose main table compares two or more methods within one
   of the two case-study families.
2. Use only the headline or main-results table. Never pick an appendix after
   seeing its numbers.
3. Report every eligible comparison, including those clearly above the noise
   floor. Picking only weak results would repeat the behavior being criticized.
4. A paper's own variation estimate always replaces an imputed value.
5. Extract numbers with a script. Release the script and its output.

## Decision rules

- **Write the paper** if named published rankings are unresolved at
  `gamma = 0.75` among comparisons with direct variation. Four Case B cells
  already meet this condition before the larger collection, so the outcome is
  partly known.
- **Write a narrower paper** if unresolved rankings appear only with imputed
  variation. The claim would concern reporting standards, not a named
  conclusion. That is weaker but honest.
- **Report a defense of current practice** if most published gaps are much
  larger than the relevant noise. This would be a real result: the field's
  score margins are large enough, and the study proves it. Declaring this
  outcome in advance prevents the authors from treating it as failure.

## Claims this study cannot make

- Not that these areas never measure variation. [MiniLLM](https://arxiv.org/abs/2306.08543) uses 5 seeds, GKD 3,
  DistiLLM 5 with standard deviation, SparseGPT 5, Wanda 5, and Williams &
  Aletras 10 samples from each of 5 sources. The problem is that **papers do not
  use those estimates when they state method rankings**.
- Not that calibration samples or random seeds are a new concern. Williams &
  Aletras, Ji et al., and Bouthillier already established that.
- Not that `P(A > B)` is new. It belongs to Bouthillier et al.
- Not that any author behaved badly. In Case A, the work passed unanimous peer
  review under normal field standards. In Case B, the papers **do report seed
  variation in appendices**. The study concerns the convention, not the people.
- Not that the methods fail. It asks whether method **rankings** are clear, not
  whether distillation or compression is useful.

## Required safety and fairness rules

- Discuss the research areas as a whole. DistiLLM, Wanda, and SparseGPT are
  examples, not targets.
- **Do not include the odd table values recorded in
  [[Compression-Audit-Direction]].** The public reviews already give a clean
  field-level result: careful reviewers did not check variation. Adding table
  concerns would turn that into an accusation against authors.
- Privately tell the authors of both case-study papers before submitting.

## Items that still blocked the final plan

1. **Run one final earlier-work check:** has anyone already used `P(A > B)` or
   an equivalent test to audit rankings across a published machine-learning
   literature? Search reproducibility, meta-science, and benchmark-reliability
   research by task, not just by method. [`2605.20798`](https://arxiv.org/abs/2605.20798) already ran an equal-compute
   noise-floor audit for transformer changes. Read its **methods** and quote
   the exact sentence that makes this proposal different.
2. Fix the paper list, venues, and date range.
3. Check the exact wording of DistiLLM-1's seed process in the final paper.

## Standing rule

> Before claiming a research gap, read the **methods** of the two or three
> closest papers and quote the exact sentence that proves the gap. A gap without
> such a quote is not established.

Two of thirteen failed gates—the temperature plan and the random calibration-
sample plan—were killed by papers already cited in those plans. Both plans had
described the papers from their framing without reading their methods.

## Intended publication venue

TMLR was the natural fit because it explicitly welcomes reproduction and
analysis studies and accepts modest importance when evidence is convincing. An
ICBINB-style workshop was the backup. A main-conference submission sold as a
new discovery should be rejected. This is a consolidation study and must call
itself one.

## Related

[[KD-Evidence-Audit-Gate]] — the gate that produced Case A and corrected its
facts.
[[Calibration-Draw-Preregistration]] — the failed compression idea that still
provided Case B and the QTIP statement.
[[Direction-Gate-Results]] — the thirteen gates and their failure patterns.

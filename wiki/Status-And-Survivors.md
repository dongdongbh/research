# Where SVIB Stands, and Everything Still Alive

Status: **Orientation document, 2026-07-27.** Written in plain language. If you
read one page in this wiki, read this one.

---

# Part 1 — SVIB: what happened, and what to do with it

## The short version

The submitted paper's headline was wrong, and we found it ourselves. But the
work produced **five things that are genuinely useful and true**, and those are
publishable — just not as "we invented a better method."

## What went wrong

**The headline number was an artifact.** The frozen-CLIP baseline was computed
through a code path that applied the text projection twice. Corrected, the
baseline goes from `42.27` to `66.34` on SugarCrepe++, so the claimed `+25.23`
improvement becomes `+1.16`. The corrected Winoground baseline (`9.00`) also
matches published values, while the submitted one (`0.50`) never did — which is
how we caught it.

**The mechanism was not what we said.** Under equal training:

- a **deterministic grid of crops** with zero object awareness **beats SAM3**
  proposals (`69.04` vs `68.43` on CLIP);
- **plain self-attention beats the directed sparse graph**;
- the **variational bottleneck** contributes nothing after fusion (`beta = 0`
  matches the tuned value);
- **14 crops match 20** (`-0.11`).

So none of the paper's distinctive components — object proposals, sparse
directed routing, the information bottleneck — is doing the work.

**The gate does the opposite of what it claims.** The adaptive caption-overlap
gate fires on **81.4%** of cases the global scorer already gets right and only
**59.6%** of cases it gets wrong. It is worth `+0.80` where simply blending
everything is worth `+0.71` — nine hundredths of a point for the paper's second
headline contribution. And perfect routing would only be worth `+3.99`.

**Six probes, one answer.** Beyond SVIB itself we tried pooled patch tokens,
caption decomposition into claims, equivalence-class dispersion, marginal
calibration, and exclusive optimal-transport assignment. Under honest
validation-locked selection, **all six were rejected in favour of the raw global
score** (the tuning procedure picked `alpha = 1.0`).

## What is actually true and worth publishing

These are real findings, they are ours, and nobody has published them:

1. **Cheap beats clever.** A deterministic multi-scale grid plus one
   self-attention layer beats SAM3 proposals plus a variational graph, at
   **8-11x** overhead instead of **258-366x**. That is a useful engineering
   result with a clean cost table.
2. **But crop re-encoding is load-bearing.** One-pass pooled patch tokens are
   nearly free (`1.06x`) and **lose** (`-1.32` on CLIP). So the expensive part
   is not the proposals — it is encoding regions at full resolution.
3. **An evaluator bug with a diagnostic signature**, plus a reusable fix: any
   fused system at `alpha = 1` must exactly reproduce its standalone backbone.
   We also caught a QuickGELU-versus-standard-GELU reference mismatch that
   silently corrupts source-matched comparisons.
4. **Hand-designed semantic gates are anti-aligned** with where the expert
   actually helps, and the oracle ceiling is small. This is measured, not argued.
5. **Benchmark conclusions flip** under the second, equally valid positive
   caption that SugarCrepe++ already provides — four sign reversals.

## How to deal with it

**Do not resubmit SVIB as a method paper.** The method claim does not survive
its own controls.

**Do write it up as a controls-first study.** The honest and interesting title
is something like *what actually drives compositional gains in frozen
dual-encoder VLMs* — with the grid-beats-SAM3 result as the positive
contribution, the cost table as the practical takeaway, and the evaluator suite
as a released artifact.

Three things make this stronger than a normal negative result:

- It examines **our own work**, so no reviewer has to be told they were wrong,
  and the reviewer-conflict problem disappears.
- The experiments **already exist**, pre-registered and provenance-hashed. This
  is a writing task, not a compute task.
- The selection protocol is the transferable contribution: published methods
  report gains at test-selected operating points and get rejected at
  `alpha = 1.0` under validation-locked selection. We can show that from our own
  runs.

**Venue: TMLR** (its scope explicitly covers this) or **ICBINB** (the
NeurIPS negative-results workshop). Not a top-tier main track, and it should not
be aimed there.

**Do this soon.** It is the only item on the whole list where the work is done
and only the writing remains.

---

# Part 2 — everything still alive, in plain language

## The strong ones

**T1 — Should you freeze the vision encoder, and does the training objective
change the answer?**
Three respected papers disagree about the most basic recipe question in VLM
training. One says unfreezing the vision tower badly hurts fine-grained tasks;
two others say unfreezing clearly helps. The first paper even guesses why —
maybe language-generation training is the wrong objective for learning visual
detail — and nobody has tested that guess. Run the full grid: frozen versus
unfrozen, language loss versus RL, one training stage versus two. The reference
run takes nine hours.

**T4 — What data should you feed a model while the learning rate decays?**
Every big lab saves its best data for the final "cooldown" phase of pretraining.
It is pure folklore — there is no method paper on it. Meanwhile the data used in
this window is worth **+17 to +28 points** on hard reasoning benchmarks, while
changing the reinforcement-learning recipe afterwards is worth **under 2**. The
trick that makes it affordable: train the shared trunk once, then fork many
short decay phases from it, so every comparison is paired.

**SVIB writeup** — described in Part 1.

**T2 — Does reinforcement learning actually teach a model to *see* better?**
Apparently not. One paper ran a proper statistical test and found RL gives **no
significant improvement in visual perception** despite raising headline scores —
it just makes the model more likely to output answers it already half-knew.
Another found **67% of remaining errors are perception failures**. The null is
already published; extending it is cheap.

**B1 — Does post-training make models boring because of the method or the
data?**
Fine-tuned models produce less varied outputs. Nobody has separated whether that
comes from the training algorithm or from the narrow data. The authors say so
themselves. Olmo 3 releases both open weights and open training data at every
stage, so for once the confound can actually be pulled apart.

## The medium ones

**T3 — Data mixing beats data filtering.**
A large study found *no* quality filter reliably helps (best `+0.8`), but
changing the *ratio* of instruction data to captions gives `+5.4`, and careful
curation at fixed compute gives `+11.7` overall and `+57.1` on grounding. Huge
effects. The catch: mixture optimization as a field is saturated, so the
contribution has to be the constant-compute framing, not another mixer.

**A1 — Were the video-model benchmarks ever checked against humans?**
Mostly no. The flagship one computes its human-agreement claim from **four
models**, where the statistic literally cannot reach significance. Another calls
its judge "human-aligned" and reports no agreement number at all. And one
benchmark released 102,000 human labels plus its judge, so you can check this
without generating a single video.

**KV-cache allocation for the right objective.**
Methods that decide how many bits each part of the attention cache gets all
optimize a one-step error measure. Two other papers show the real damage is
different — errors compound over long generations, and safety behaviour lives in
a fragile subspace that loses 15% of refusals at almost no perplexity cost.
Neither fixed the allocator. Two to four GPUs, no training.

**C1 — Most LLM ablations use the wrong statistics.**
When you test three models across many prompts, you cannot treat all runs as
independent — model is a "hard-to-change" factor and prompt is not. Pooling them
makes model-level effects look more significant than they are. The fix is
standard in industrial statistics and ships as working code.

**C2 — Detecting contamination with psychometrics.**
There is a mature technique for spotting test questions that behave oddly for
equally-able subjects. That is exactly the shape of "this benchmark item is easy
for a contaminated model." Two separate searches ranked it highly.

## The weak ones

**P3, P4** — more work on the frozen dual-encoder local branch. Six probes have
already been rejected there, and the official code release is still a
placeholder. **Recommended against.**

**A2, A3** — extensions of A1; only worth it if A1 lands.

**C3, C4** — narrow statistical transfers (survival analysis for agent failures,
queueing theory for agent fan-out). Real but small.

**AHD survivors** — classical optimizers as a rival to LLM algorithm discovery
at matched cost. Real gap, but three control papers landed in five months and
the window is 3-6 months.

**Weather** — needs a meteorology collaborator to be credible.

---

# Part 3 — the table

Priority is **overall recommendation**, weighing fit, cost, crowding and how
likely it is to produce a paper.

| # | Direction | Type | Field crowding | Compute | Pros | Cons | Priority |
|---|---|---|---|---|---|---|---|
| 1 | **T1** freeze x objective x stage | Method | Medium | 300-600 GPU-h | Resolves a live 3-way disagreement; 9-hour reference run; tests a mechanism the original authors named; every outcome publishes | Adjudication-flavoured; needs careful matched-budget design | ★★★★★ |
| 2 | **T4** anneal-window data | Method | **Very low** | ~1,000 H100-h | Biggest effect size found (+17-28); empty lane confirmed 3 ways; paired shared-trunk design | Most compute; furthest from prior experience; needs two scales | ★★★★★ |
| 3 | **SVIB writeup** | Both | Low | **~0** | Work is done; own data so no reviewer conflict; releases a reusable artifact | Not top-tier; negative-results framing | ★★★★☆ |
| 4 | **T2** RL perception decoupling | Method | Medium | ~256 GPU-h | Published null to build on; statistical; cheap | Extension of others' finding; RL space moves fast | ★★★★ |
| 5 | **B1** diversity collapse | Method | Low-medium | ~200-400 GPU-h | Positive contribution; authors named the missing design; Olmo 3 makes it manipulable | Ungated; 2026 paper so someone may be on it | ★★★★ |
| 6 | **T3** constant-compute mixture | Method | **High** | ~400 GPU-h | Largest measured effects (+5.4 to +11.7) | Mixture optimization is saturated (22+ methods, 2 surveys) | ★★★ |
| 7 | **KV allocation** (M1/M2) | Method | High | 2-4 GPUs | Clean "beat this number"; two documented failure modes; no training | Industry-heavy; needs custom kernels to be competitive | ★★★ |
| 8 | **A1** world-model judge audit | Evaluation | **Very low** | <100 GPU-h | Cheapest real result; finding verifiable from a PDF today; gated and held | Audit-shaped, and audits are being written faster than they publish | ★★★ |
| 9 | **C1** split-plot ablations | Evaluation | Low | Very low | Correctness claim; ships a method not a complaint; cheap formalism | Window tight (a competitor shipped 2 weeks ago); still measurement | ★★★ |
| 10 | **C2** contamination via DIF | Evaluation | Low-medium | Medium | Two searches converged; best use of GPUs among evaluation items | Ungated; adjacent privacy literature is active | ★★★ |
| 11 | **A2/A3** world-model extensions | Evaluation | Low | 1-3 weeks, 8 GPUs | Natural follow-ons to A1 | Only sensible after A1 lands | ★★ |
| 12 | **AHD** classical-optimizer comparison | Evaluation | Medium-high | Low | Real remaining gap on the standard benchmark set | 3 control papers in 5 months; 3-6 month window | ★★ |
| 13 | **C3/C4** frailty, fork-join | Evaluation | Low | Low | Genuinely unoccupied | Narrow; small audience | ★★ |
| 14 | **P3/P4** more local-branch work | Method | Medium | Medium | — | Six probes already rejected this family; code release is a placeholder | ★ |
| 15 | **Weather** twPCRPS / METAR | Evaluation | Medium | Very low | Near-zero compute | Needs a domain collaborator; main claim already scooped | ★ |

## If you only do two things

**Write up SVIB** (zero compute, converts finished work into an output) **and
start T1** (nine-hour reference run, resolves a real disagreement, and it
subsumes the freeze/unfreeze idea that was separately on the list).

If T1 goes well and you want a bigger swing afterwards, **T4** is the strongest
paper on the list.

## Related

[[Method-Opportunities]] — full detail, baselines and numbers to beat.
[[Live-Research-Opportunities]] — the evaluation-side directions.
[[Post-Rebuttal-Measurement-Sprint]], [[Cluster-1-Compositional-Scoring]] — the
SVIB evidence base.

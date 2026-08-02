# Where SVIB Stands, and Everything Still Alive

Status: **Orientation document, 2026-07-27.** Written in plain language.
Updated 2026-08-02 for the general research wiki: Part 1 condensed to its
transferable lessons, and the star table in Part 3 marked superseded.

> **Star table superseded 2026-08-02.** The Part 3 ranking below reflects the
> July gate. A dedicated re-evaluation moved six of its rows — T4 down to ★★,
> B1 down to ★★★, T3 up to ★★★★½, T1 to ★★★★½, AHD up to ★★★★, KV-cache up if
> narrowed — and reversed the separate "do not enter self-improving AI"
> verdict. Current authority: [[Direction-Reevaluation-2026-08]], with the
> people-and-openings layer in [[Top-Researcher-Scan-2026-08]]. The table is
> kept as the record of what was believed in July, not as guidance.

---

# Part 1 — SVIB: what happened (condensed)

Project-specific numbers, probes and provenance live in the svib repo wiki.
This is the short version, kept only because the lessons underneath it get
reused on every project.

The submitted paper's headline was an artifact: the frozen-baseline code path
applied the text projection twice, and correcting it shrank the claimed
improvement to roughly a point. We caught it ourselves, because the corrected
baseline matched published values and the submitted one never had. Under equal
training, none of the paper's distinctive components — object proposals, sparse
directed routing, the variational bottleneck — was doing the work; a
deterministic grid of crops plus one plain self-attention layer matched or beat
the whole apparatus at a small fraction of the overhead. The hand-designed
semantic gate fired mostly on cases the global scorer already got right, and
even perfect routing had a small ceiling. Six further probes were all rejected
in favour of the raw global score under validation-locked selection.

**The transferable lessons, which are the part worth keeping:**

1. **Cheap beats clever often enough that the deterministic baseline has to be
   run before the clever one is believed.** But note the flip side we also
   measured: the expensive step was not the proposals, it was re-encoding
   regions at full resolution — free one-pass alternatives lost.
2. **A fused system at `alpha = 1` must exactly reproduce its standalone
   backbone.** That invariant is a cheap, reusable evaluator-bug detector.
   Watch also for QuickGELU-versus-standard-GELU reference mismatches, which
   silently corrupt source-matched comparisons.
3. **Hand-designed semantic gates can be anti-aligned** with where the expert
   actually helps. Measure the oracle ceiling before building the gate.
4. **Benchmark conclusions flip** under a second, equally valid positive
   caption. Check the ranking's stability before trusting it.
5. **Test-selected operating points inflate reported gains.** Under
   validation-locked selection, methods collapse to the trivial setting. The
   selection protocol is the transferable contribution, not the method.

**Disposition:** not a method paper. Write it as a controls-first study of what
actually drives compositional gains in frozen dual-encoder VLMs, at TMLR or
ICBINB, with the evaluator suite as the released artifact. It examines our own
work, so there is no reviewer-conflict problem, and the experiments already
exist — this is a writing task, not a compute task.

Full detail: svib repo wiki, pages Post-Rebuttal-Measurement-Sprint and
Cluster-1-Compositional-Scoring.

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

> **Superseded 2026-08-02.** The "no method paper on it" claim was already false
> when written — DiReCT had been public for about eight weeks. T4 was scooped
> three times, the small-scale forked-decay moat is now citable against us, and
> the direction drops to ★★. Only a narrow residual question survives. See
> [[Direction-Reevaluation-2026-08]].

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

> **Superseded 2026-08-02.** Scooped in April 2026; the specific question is
> closed and B1 drops to ★★★. The remaining causal crossover also costs more
> than the estimate below. See [[Direction-Reevaluation-2026-08]].

## The medium ones

**T3 — Data mixing beats data filtering.**
A large study found *no* quality filter reliably helps (best `+0.8`), but
changing the *ratio* of instruction data to captions gives `+5.4`, and careful
curation at fixed compute gives `+11.7` overall and `+57.1` on grounding. Huge
effects. The catch: mixture optimization as a field is saturated, so the
contribution has to be the constant-compute framing, not another mixer.

> **Superseded 2026-08-02.** The saturation was text-only — the multimodal
> mixture lane is close to empty, and T3 rises to ★★★★½ with public
> checkpoints cutting the cost by ~50×. See [[Direction-Reevaluation-2026-08]].

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

> **Superseded 2026-08-02.** Claims are outrunning controls roughly 10:1 and
> the cost-normalized frontier still does not exist; AHD rises to ★★★★, with a
> second unoccupied novelty-audit half. See [[Direction-Reevaluation-2026-08]].

**Weather** — needs a meteorology collaborator to be credible.

---

# Part 3 — the table

> **Superseded 2026-08-02 — historical record, not guidance.** This ranking was
> built on crowd count. That criterion has since been replaced by remaining
> opportunity, and every crowd-count downgrade in the table below reversed while
> both emptiness-credited rows fell. Read
> [[Direction-Reevaluation-2026-08]] for the current stars, and
> [[Top-Researcher-Scan-2026-08]] for the newer cross-person openings that are
> not in this table at all.

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

> **Superseded 2026-08-02.** The write-up and T1 both still hold, but T4 is no
> longer the bigger swing — it fell to ★★. T3 and the reversed self-training
> collapse lane are the current second bets, and T1 now needs an immediate
> re-gate against Darrell's encoder-fix cluster. See
> [[Direction-Reevaluation-2026-08]] and [[Top-Researcher-Scan-2026-08]].

## Related

[[Direction-Reevaluation-2026-08]] — current star ranking; supersedes Part 3.
[[Top-Researcher-Scan-2026-08]] — people-level openings, current.
[[Method-Opportunities]] — full detail, baselines and numbers to beat.
[[Live-Research-Opportunities]] — the evaluation-side directions.
Post-Rebuttal-Measurement-Sprint, Cluster-1-Compositional-Scoring (svib repo
wiki) — the SVIB evidence base.

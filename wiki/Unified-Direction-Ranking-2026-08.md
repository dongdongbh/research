# Unified Research-Direction Ranking — version 3, updated 2026-08-07

**Status: THIS IS THE CURRENT RANKING — everything in one place, and as
of Aug 6 every row has a formal verdict.** On Aug 6 we ran a full
adjudication day on one held 2-GPU node: thirteen directions were killed
by their own pre-stated rules or by verified prior work, one survived its
check (the sparsity-premise test), two committed pre-registrations moved
forward (1-NFE passed its week-1 check; the contextualization pipeline
was built and smoke-tested), and two big results landed (the verb-role
probe replication and the MOCHI cross-task contrast). Total compute for
all of it: about 20 GPU-hours. Every kill below names its evidence so the
idea is not re-proposed. The version-2 page (Aug 3) is in git history.

**The one lesson to carry:** across 18 formally checked ideas this month,
scout/first-look ratings dropped by 1–2 stars on average once a deep check
ran, and the two worst kills were papers less than eight weeks old with
zero citations. Nothing enters a pre-registration without the two-pass
check, an OpenReview sweep, and a verified first-cheap-step.

## Part 1 — Committed work (not ranked; already chosen)

| Deadline | Project | State |
|---|---|---|
| ICLR (Sep 18) | **RoboJudge audit** ★★★★½ — do robot-policy evaluators recover the human ranking | Week 1 done; ranking frozen; **ready to lock** at sign-off (4 pinned choices) |
| ICLR (Sep 18) | **Crop-consistency distillation** | **BENCH** — insight published April ([2604.11496](https://arxiv.org/abs/2604.11496)); the fold target (Cell A) was killed Aug 6, so per the pre-stated rule the fold decision collapses to bench; standalone ★★ remains the only alternative at sign-off |
| CVPR (~Nov 13) | **1-NFE diversity** (one-step image generators) | Prereg drafted; unchanged — now the only CVPR method-slot candidate |
| CVPR (~Nov 13) | ~~Readout-budget vs binding frontier (Cell A)~~ | **KILLED Aug 6** by its own pre-stated pilot rule: best untrained readout minus R0 = **−21 points** on SugarCrepe++ (both backbones, CIs exclude zero the wrong way); the readout ladder is flat (2.6-point spread K=1→256), so there is no frontier to measure. Bug-hunted: retrieval control healthy, wiring checks exact, reruns bit-identical. Report: cropdistill `runs/cellA_pilot/20260806-report/` |
| ICML (~Jan 28) | **Epistemic contextualization** | Prereg drafted; citation corrected to [2606.29657](https://arxiv.org/abs/2606.29657). **Pipeline BUILT and smoke-verified Aug 6** (1.2 GPU-h): full train→checkpoint→resume→eval loop proven on H100; 2B-token mixture staged, contamination-clean; six pre-study bugs fixed. Owner decisions before the study: budget re-estimate (165 GPU-h for the eight runs vs the prereg's 100, + 85–190 h rewriting), rewriter yield strategy (42–46% pass rate — align vs loosen), and the H1 headroom problem (the 1B base is at chance on ConflictBank; a probability-mass metric was added). Readiness note: `code/ctxprereg/READINESS.md` |
| TMLR (no deadline) | **SVIB audit paper** | Decided Aug 4: one paper — audit + locked protocol + grid+attention positive + released suite; writing task ~2–3 weeks |
| Running | **Role-decodability program** | COMPLETE at power on both strata (Aug 6): spatial 99.5% in patches → 50.1% at the score; verbs n=279 — patches 74%, pooled 52–57%, score at chance (the pilot's below-chance 29.8% was small-n noise), region-readout inversion independently replicated (33–39% on unseen items). Both strata land in the suppression-at-readout cell. See [[Binding-Root-Cause-Analysis]] §8. **⚠ Aug-6 scoop alert for the planned method paper:** [2608.00726](https://arxiv.org/abs/2608.00726) (Aug 1) publishes the core patch-vs-global finding, and the method cell is occupied by DisCoCLIP (see the killed algebraic-binding row). What remains uniquely ours: the powered two-strata corpora, the inversion result, the Cell A dissociation, and the MOCHI specificity contrast. The probe→method-paper plan needs an owner decision at sign-off |

## Part 2 — The all-in-one ranking of available directions

Stars are post-check where a check ran (marked ✓); pre-check otherwise.
"First step" is the cheapest action that decides whether to continue.

### ★★★★½–★★★★ — strongest available

| Direction | ★ | Cost (GPU-h) | First step | Note |
|---|---|---|---|---|
| Abbeel parallel-RL factorial (which fast-RL ingredients matter) | ★★★★½ | 400–650, OrangeGrid | re-check the source group's latest releases | **Expiring** — the authors are working through the axes; swap-in for a slate slot or it dies |
| ~~Replay-mechanism arbitration~~ | **KILLED Aug 6 — false premise + Level 1** | Embarrassing double failure caught at the gate for 0 GPU-h. (1) The row misstated its source: [2603.04964](https://arxiv.org/abs/2603.04964) is about replay improving FINE-TUNING data efficiency (1.9–2.1×), not pretraining stability — loss spikes appear only in its Appendix B.1, where the authors call the spike explanation "one relatively vague hypothesis" and have not shown spikes are harmful. The "five candidate causes with cheap falsifiers" exist in no wiki record — the claim was unsourced. (2) The corrected question is fully published: [2605.26097](https://arxiv.org/abs/2605.26097) runs the capacity-vs-optimization-vs-replay arbitration; [2607.00634](https://arxiv.org/abs/2607.00634) owns the transition-transient arm (our own wave-2 record had already flagged it); [2506.04805](https://arxiv.org/abs/2506.04805) publishes the spike-detection instrument. The fast-moving cluster here is Raghunathan/Springer + Wilson (four mechanism papers Mar–May), not the source authors, who moved to synthetic data in March. Verdict: replayarb `runs/20260806-stage1/stage1_verdict.json` |
| ~~RLVR vs self-consistency calibration~~ | **DIED at its pilot, Aug 6** | Gate PASSED (Level 3; six OLMo-3 pairs isolate the RLVR stage; gate record [[RLVR-Calibration-Gate]]), and the pre-registered pilot then answered the question honestly: RLVR does NOT break the calibration of self-consistency agreement — the pre/post ECE difference excludes zero on neither dataset (Holm p 0.77 / 0.15), and the agreement signal is already well-calibrated on BOTH checkpoints (ECE 0.06–0.13). The named secondary even moved the other way: post-RLVR agreement became MORE informative on MMLU-Pro (slope +0.95 [+0.23, +1.74]). A calibrated-aggregation method has nothing to fix, so the row closes. This is a clean, well-powered negative worth one paragraph in any future aggregation paper. ~12 GPU-h. Report: cropdistill `runs/rlvr_calib/20260806/` |
| ~~KV-cache footprint under agentic workloads~~ | **SCOOPED Aug 6 — Level 1** | The gate found BOTH halves published: [MemDecay (2607.10582)](https://arxiv.org/abs/2607.10582) fits attention half-lives on agent traces (with CIs) and ships a turn/region-aware eviction policy; [IntentKV (2606.09916)](https://arxiv.org/abs/2606.09916) publishes our exact diagnosis plus a cross-turn policy at 8–14B; [TraceLab (2606.30560)](https://arxiv.org/abs/2606.30560) released 4,300 real agent sessions and names agent-aware eviction as its own next step; an SGLang RFC is building it in production. Worse for any revival: MemDecay's data suggests our premise was backwards — accumulated-attention retention (H2O) STRENGTHENS with scale; only the recency family is mistuned. The neighborhood publishes ~one agent-KV paper per week. Retire; do not re-propose (third KV-family death this cycle). 0 GPU-h spent. Verdict: kvagent `runs/20260806-stage1/stage1_verdict.json` |
| Bengio sparsity-premise test ✓ (Aug 6) | ★★★★ **SURVIVES — design ready for sign-off** | **~70 GPU-h** (measured, not the old ~250 guess) | Approve `code/sparsityprem/DESIGN.md`, then run on **OrangeGrid** (300 short independent jobs — the HTCondor profile; measured: H100 concurrency buys exactly 1.00×) | Gate: Level 3 vs the ≤2 bar; the only citing paper (Oberman/LawZero, zero experiments) names our exact hypothesis as untested — expiry risk, move fast. Two corrections from reading the source: the row's label conflated two premises ([2606.29657](https://arxiv.org/abs/2606.29657): sparsity = Def. 5.22 R-shell; Req. 5.23 = no-enrichment — the design measures both, plus loss-visibility); and the contextualization link was overstated — a result here buys that paper a paragraph, not an arm; judge this as a standalone safety-empirics paper |
| [MOCHI](https://arxiv.org/abs/2409.05862) VLM arm | ★★★★ | 20–60 | **DONE Aug 6 (0.7 GPU-h!)** — sanity gate passed twice (r>0.988 vs published per-trial data). Results: SigLIP2-L **ties the best published model** (DINOv2-giant, 0.442 vs 0.443); base-size SigLIP2 beats every published CLIP; at fixed architecture the pretraining recipe changes the score 2.5× (OpenAI vs LAION B/32). Humans (0.782) still far ahead. **Cause test: NO suppression and NO inversion anywhere** — view-consistency information is never built, not destroyed at the readout. The explicit mismatch with the role-binding cell is the finding: readout-repair methods should NOT be pointed at 3D view tasks. Report: cropdistill `runs/mochi/20260806/REPORT.md`. Next decision: is this a paper arm or a completed side result — owner call at sign-off | — |
| ~~GMP three-mechanism arbitration~~ | **KILLED Aug 6 — Level 2 + infeasible** | GMP = **Gated Memory Policy**, [2604.18933](https://arxiv.org/abs/2604.18933) (Song group; the acronym was never expanded anywhere in our records until this gate). The "read the PDF first" warning was right twice over: the paper partly self-ablates (the noise mechanism fully; the authors admit the gate is NOT behind the MIKASA headline), and [Tedrake-group 2606.16447](https://arxiv.org/abs/2606.16447) (June) already runs the credit-reassignment study on the sibling method — including the encoder-freezing confound we would have found — while showing "naive context scaling is not as brittle as advertised." [RMBench](https://arxiv.org/abs/2603.01229) and [μVLA](https://arxiv.org/abs/2606.12497) isolate the rest. Also infeasible here: ~220 H100-h floor (the paper's own cost table), two tasks need real robots, MIKASA needs Vulkan. 0 GPU-h spent. Files: `code/gmparb/` |

### ★★★½–★★★ — real but narrowed (all ✓ checked)

| Direction | ★ | Cost | First step (all cheap and decisive) | What the check found |
|---|---|---|---|---|
| **Robot-evaluator uncertainty audit** ✓✓ (headline CONFIRMED Aug 6) | ★★★ **proceeds to design** | 200–400 | Next: write the boundary-vs-RoboJudge note, then design | The Fisher-z afternoon ran: **the field's headline correlations are statistically empty** — WorldEval's n=4 r=0.942 has interval [−0.20, +0.999]; 40 of 74 published intervals contain zero; a claimed 0.700→0.875 improvement has p=0.63; one paper's interval halves once its 7×3 pseudoreplication is unwound. **Correction: RoboDojo ([2607.04434](https://arxiv.org/abs/2607.04434)) publishes NO correlations** — it is the best *target* (30 policies, 4× anyone), not a source; its own tables show task-level real-robot noise (SD up to 23 points) twice the entire leaderboard spread. Full table: decisive0806 `runs/partB_fisherz/` |
| ~~Video role-direction test~~ | **KILLED Aug 6 by its own pre-stated rule** | The judge audit ran on VELOCITI's Agent Binding test (official gated release; owner accepted the license; order-robust A∧B protocol): best of four judges 67.3% [59.5, 74.3] vs the 85% bar — five standard errors short, with humans at 92.9%. No automatic judge exists for the construct, so the 300–600 GPU-h direction cannot be built. Clean kill for 0.4 GPU-h. Results: decisive0806 `runs/partA_velociti/` |
| ~~Algebraic role-binding embeddings~~ | **SCOOPED Aug 6 — Level 1 (full overlap)** | The pre-design re-check found [DisCoCLIP (2509.21287)](https://arxiv.org/abs/2509.21287) (Sept 2025): a frozen CLIP tower + a small trained tensor-product composition head + one vector per side scored by plain cosine, evaluated on subject–object swaps with a commutative control — our claim on all four axes, in the cacheable dual-encoder regime. Second blocker: [2605.31503](https://arxiv.org/abs/2605.31503) (ICML 2026) publishes our diagnosis AND the multiplicative-mechanism thesis, with evidence against the frozen-CLIP route. Also [2608.00726](https://arxiv.org/abs/2608.00726) (Aug 1) publishes the probe program's core finding (patch tokens retain binding; the global embedding is the bottleneck). Why our earlier check missed it: the "43 HRR papers, 0 on CLIP" emptiness was VOCABULARY-keyed — DisCoCLIP does the same algebra under "tensor network" and never says HRR. Honest residue (diagnostic, not a method): DisCoCLIP's 93.68% is on 95 pairs with no text-prior control, and our probe says the pooled vector it scores is role-blind — auditing that number is cheap and would be a real correction. Owner decision needed on the probe→method-paper plan. Verdict record: cropdistill `.orchestrator/tasks/rb-design-20260806-01/stage1_verdict.json` |
| ~~Anthropic VLA-supervision replication~~ | **KILLED Aug 6 — false premise** | The [post](https://www.anthropic.com/research/claude-plays-robotics) finds the OPPOSITE of our row: "every tested model still performs substantially worse than MolmoAct does on its own" — supervision *hurts* (headline cell n=200, not 36; the n=36 cell is the one place the authors DID report confidence intervals, by their own choice). A replication is not a method under our definition; the missing repo is a corrected fact, not the reason. Residue: a zero-cost binomial-interval comment |
| Choi hivemind decomposition | ★★★½ | low | resolve the 404'd dataset | Blocked on data (unchanged) |
| ~~T1 narrowed interaction study~~ | **KILLED Aug 6** | [PIVOT](https://arxiv.org/abs/2510.16333) (ICLR 2026 — our records missed the acceptance) itself publishes the GRPO arm our row claimed as its delta (§5), plus PPO/MPO; its own Table B shows every {freeze, update}×{GRPO, PPO} cell has a running repo. The only unpublished piece is an interaction term — statistics, excluded by the method definition — at ~860 H100-h true cost (the paper's own 18 h × 8 H100 per recipe), 3.5–6× the row's budget |
| ~~KV three-way interaction~~ | **KILLED Aug 6 — unaffordable, not unoriginal** | The three-way heads×tying×window factorial at matched KV bytes is genuinely unclaimed (verified across three vocabularies; the agentic-KV blockers do NOT cover it) — but it means something only at ~1,000 H100-h, and [2606.15378](https://arxiv.org/abs/2606.15378) shows small-scale versions measure emergence *speed*, not converged quality. The Tsinghua group behind Cost-Optimal GQA is systematically walking the axes. Revival test (0 GPU): check whether their convergence curves leave any three-way interaction at the converged point |

### ★★½ and below — bench (run only as cheap side arms)

| Direction | ★ | First step |
|---|---|---|
| Video coverage re-measurement ✓✓ (pilot DONE Aug 6, 1 GPU-h) | ★★½ audit-remnant only | The ρ>0.8 kill bar was NOT met (ρ=0.62 [0.48, 0.73]), but a structural finding decides more: off-the-shelf STREAM-D is a two-sample recall statistic that needs a real-video reference, and against any available reference the generated and real sets are disjoint (precision 0.004, coverage 0.001) — the metric reads out the generated set's own spread, not coverage of anything. "Coverage of what?" now has a measured answer: no runnable reference exists for T2V. The AUDIT remnant survives concretely; the metric route is closed. Report: cropdistill `runs/stream_check/report/` |
| ~~Fork-preserving context construction~~ | **KILLED Aug 6 at its own 48-h re-check** | The long-horizon home the wave-2 record reserved is occupied: [PivoARL (2607.03702)](https://arxiv.org/abs/2607.03702) retries from the pivotal wrong turn reusing the correct prefix (code released, beats full retry — our falsifier — by 42% fewer turns) and [CWL (2606.11213)](https://arxiv.org/abs/2606.11213) ships the keep-exploration/shed-persisted-actions eviction. The lane moves ≥1.7 papers/month, not ~1. What remains is two missing ablations inside someone else's shipped method |
| ~~Cell B: per-item position split~~ | **KILLED Aug 6** | Pilot ran (0.2 GPU-h): 17% of items flag as position-dependent, clearing the 10% bar — but an equal 17% flag in the MIRROR direction and cross-backbone agreement is κ=0.11 vs the pre-stated 0.40, so the per-item split is model-specific fragility, not a property of items. Survivor worth keeping: at the SUBSET level, relation and object-swap items reliably lose more to patch shuffle than to accuracy-matched noise on both backbones (attributes the reverse) — real signal, but SugarCrepe++'s existing labels already carry it, so no new paper. Also refuted: LaSt-ViT's lazy-aggregation prediction (full shuffle costs 19.8 points, not ~0). Report: cropdistill `runs/cellB_pilot/20260806-report/` |
| ~~Spectral-band × knowledge injection (MiCA)~~ | **KILLED Aug 6** | The K1 reproduction gate ran (1.8 GPU-h, 192 training runs) and FAILED: with an exact reimplementation (trainable-parameter count matches the paper), the minor-subspace arm never beats the random-subspace control at any setting. Stronger still: on the most direct absorption measure the ordering REVERSES — the major subspace absorbs new text better than the minor one on 8 of 8 seeds — and no subspace arm acquires any retrievable knowledge despite near-zero training loss. The paper's own standard errors exceed its claimed gains, and its corpus, questions, and ablation settings were never released. O1 is dead. Report: cropdistill `runs/mica_k1/2026-08-06/RESULT.md` |
| Accept-rule run-level residual | ★★ | ~20 GPU-h seed-variance check | PACE is the mandatory baseline |
| V2A secondary analysis (salvage) | note-scale | days: bootstrap CIs + power curves on [SynthSync's released 306K ratings](https://arxiv.org/abs/2607.09091) | Not a paper; a methods note |
| Wei verifier-rule Q1 · polychromic audit · Arora drag-fork addon · SigLIP-2 ladder (next-cycle, Delta) · environment-provenance study | ★★–★★★ | as in v2 | Environment-provenance decision window has lapsed — revisit or drop at sign-off |

## Part 3 — Killed, vetoed, or expired (consolidated — do not re-propose)

**Aug-6 pilot kills:** Cell A readout-budget frontier (killed by its own
pre-stated rule, −21 points with the CI excluding zero in the wrong
direction on both backbones; details in Part 1) · Cell B per-item
position split (κ=0.11 vs the 0.40 bar; equal flags in both directions;
the surviving subset-level effect already lives in SugarCrepe++'s own
labels — details in the Part 2 row). Two Cell A by-products survive
and feed Cell B: patches help retrieval monotonically but never binding
(a clean dissociation on 4,757 items), and the apparent sign inversion at
battery scale is a MaxSim caption-length artifact — the real inversion
lives only in unprojected token space, matching [[Binding-Root-Cause-Analysis]] §8.

**Aug-5 checks:** language-necessity index (Level-1 kill — [2606.04233](https://arxiv.org/abs/2606.04233) published the cross-benchmark study in June; LIBERO-Plus ran real policies blind) · V2A metric validation (killed — [SynthSync 2607.09091](https://arxiv.org/abs/2607.09091), 306K annotations, plus Omni-Judge/PEAVS/AVBench) · **seed-noise/variance of agent leaderboards (OWNER VETO: not significant, no barrier, not novel — includes the design-layer arm)**.

**Waves 1–3 and earlier (see [[Method-Gates-Wave-2-2026-08]], [[Method-Gates-Wave-3-2026-08]], v2 in git history):** autoresearch accept rule (PACE) · cross-scaffold critic (source group published) · active-view spatial (World2VLM/SIMS-V) · compositional merging (AlignMerge) · frozen-VLM binding heads (DCSM/Q-Former) · KV from-scratch factorial (CLA/MixAttention) · late-interaction distillation (ComAlign) · spectral-PEFT LR-artifact question (answered 4×) · verifier hardening · safety-aware KV · Hyperball · CAID · multi-agent budget-matching · LPT control · PULSE · BPP (LIBERO-Gen does not exist) · DriveJudge (labels unreleased; watchlist) · weather · novelty forensics (biology-side owned) · T4 anneal-window · B1 diversity attribution.

## Part 4 — The cheap decisive steps, gathered in one list

1. ~~Fisher-z afternoon~~ DONE Aug 6 — headline EXISTS; audit proceeds
   to design (details in the Part 2 row).
2. ~~VELOCITI judge audit~~ DONE Aug 6 — no judge clears 85%; video
   role direction killed.
3. ~~Cell A dynamic-range pilot~~ DONE Aug 6 — kill-arm 2 fired; Cell A dead.
4. MOCHI — 20–60 GPU-h — RUNNING (Aug 6, GPU 1 of the held allocation).
5. ~~STREAM-D vs dispersion~~ DONE Aug 6 (1 GPU-h) — metric route closed,
   audit remnant survives (details in the Part 2 row).
6. ~~MiCA repro gate~~ DONE Aug 6 (1.8 GPU-h) — FAILED, O1 dead; the
   subspace ordering reverses on absorption (details in the Part 2 row).
7. ~~Owner: verb labeling sessions~~ DONE Aug 6 — 401/401 labeled; the
   powered rerun corrected the below-chance score claim (it was noise) and
   confirmed the region-readout inversion ([[Binding-Root-Cause-Analysis]] §8).

## Part 5 — Standing lessons (updated)

All of v2's lessons stand. Added from Aug 5: (6) **scout ratings are
provisional by ~1.5–2 stars** — never cite a scout star in a decision
document; (7) **zero-citation papers under eight weeks old are the main
scoop source now** — recency-weighted search matters more than breadth;
(8) two ideas this month died to papers that our own earlier sweeps had
already surfaced for a different purpose — check our own wiki first;
(9, Aug 6) **emptiness checks must key on mechanism SHAPE, not one
community's vocabulary** — the "43 HRR papers, 0 on CLIP" search gave
eleven months of false confidence while DisCoCLIP did the same algebra
under the name "tensor network"; (10, Aug 6) **read the source paper
before rating any row that cites one** — a row reached ★★★★ and a
funded task while misstating what its source paper claims; ten minutes
with the abstract would have caught it; (11, Aug 6) **pull the source
paper's citing papers as a required gate pass** — twice today the
citation graph, not keyword search, found the blocking work (papers
indexed under "forgetting"/"overtraining", not "replay").

## Related

[[Direction-Scouts-2026-08-05]] · [[Method-Gates-Wave-3-2026-08]] ·
[[Compositional-VLM-Survey]] · [[Binding-Root-Cause-Analysis]] ·
[[Prereg-RoboJudge-Audit]] · [[Prereg-Crop-Consistency-Distillation]] ·
[[Prereg-1NFE-Diversity]] · [[Prereg-Epistemic-Contextualization]] ·
[[Top-Researcher-Scan-2026-08]] · [[Home]]

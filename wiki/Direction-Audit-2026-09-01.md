# Direction Audit — 2026-09-01

This page audits every opening listed in [[Researcher-Scan-Refresh-2026-08-31]]
(§1 project feeds, §4 "intact" list, §5 new openings), asks whether each is
still open for us, filters and ranks them, and names the few that go to a
pre-registration gate. Eight Opus agents ran in parallel, one batch each; the
per-item evidence (searches, deep-read quotes, links) is in
`research/.orchestrator/audit-20260901/*.md`. The coordinator ranked.

## Words used on this page

- **Method** — the owner's definition: (1) a new mechanism, (2) an improvement
  on a current method, or (3) an old method applied to a new problem.
  Diagnosis, ablation, replication, audit, and statistics alone are not
  methods. An audit-shaped item passes only if its deliverable is a shipped
  working mechanism and it names the method paper it unlocks.
- **Reframing** — the smallest change that turns an item into a method. Every
  item below was rated after reframing, and the table says so.
- **Scoop level** — how much of the idea is already published. Level 5 means
  no overlap; Level 1 means fully published. An item must beat Level 2.
- **GPU-h** — one hour of one graphics card. Our ceiling for one direction is
  100–1,000 GPU-h; OrangeGrid is free with no time limit.
- **First decisive step** — the cheapest action that decides whether to
  continue, with a kill number written down before it runs.
- **CANDIDATE / BENCH / KILL** — go to a gate; keep but do not fund now; do not
  propose again.

## Headline

**Of 40 audited items, 11 are candidates, 7 are benched, 22 are killed.** Four
candidates go to pre-registration gates today. Three more are absorbed into
live papers rather than started as directions. The refresh page itself
carried eight factual errors and re-listed four directions that our own
ranking had already killed; the corrections and the process rule are at the
end.

## The ranking

Stars are the auditor's rating after the coordinator's adjustment; the
"feeds" column names the live project the item strengthens.

| Rank | Direction (reframed) | Route | Level | GPU-h | Feeds | ★ | Decision |
|---|---|---|---|---|---|---|---|
| 1 | **Role-preserving canonicalization**: the Isola/Gupta orthogonal map Q that aligns two image-text models passes classification and (we predict) silently destroys who-does-what-to-whom; fit Q under a role-swap constraint and repair it ([2602.17584](https://arxiv.org/abs/2602.17584) has one citation; features already on our disk) | 1 | 4 | 30–80 | binding programme, gives it a method | ★★★★½ | **GATE** |
| 2 | **Horizon-calibrated world-model policy evaluator**: borrow off-policy-evaluation horizon selection and ship a tool that returns the rollout length at which a world model's policy ranking best matches reality, a bias-corrected ranking, and a "do not trust past H*" interval; tests WorldGym's untested "rankings are preserved" claim ([2506.00613](https://arxiv.org/abs/2506.00613)); sim-only on [jepa-wms](https://github.com/facebookresearch/jepa-wms) | 3 | 3 | 20–40 first step, 150–350 full | the method paper the RoboJudge audit must name | ★★★★ | **GATE** |
| 3 | **Ground-truth-anchored judge card for generated-video judges**: port the RoboJudge validity protocol to judges of generated robot video (DreamGen Bench's >90% human agreement rests on a handful of per-model averages with no interval); the judge is public ([GR00T-Dreams](https://github.com/NVIDIA/GR00T-Dreams), Apache-2.0) | 3 | 4 | 20–40 | RoboJudge (DreamGen is a named audit target) | ★★★★ | **GATE** |
| 4 | **Sparsity-premise instrument** (our `sparsityprem` design, un-run since 2026-08-07): a loss-band sampler plus coordinated-failure estimator that measures the "argued sparsity" premise of [2606.29657](https://arxiv.org/abs/2606.29657) v2; still 0 empirical citers | 1 (only via the released-instrument clause) | 4 | ~70, free on OrangeGrid | standalone safety-empirics | ★★★½ | **GATE, conditional**: the design must first name the training rule Arm C unlocks (measure and decorrelate training-visible features that correlate with the safety-critical query set) |
| 5 | **Discrimination-first evaluator report card**: split each judge's failure into "soft cut-off" and "cannot discriminate" (signal detection, [Mazaheri 2608.16003](https://arxiv.org/abs/2608.16003)) and repair the repairable kind by emitting a finer score; the first step already ran on our scores and passed: three judges tied at τ = 0.714 have AUC 0.829 / 0.724 / 0.680 with non-overlapping d′ intervals; SigLIP2's reversal is a tiny per-episode tilt (d′ −0.11) amplified by averaging; InternVL3 ties 56% of sessions | 1 + 2 | 4 | 10–25 | ICLR paper directly | ★★★★ | **ABSORB into ICLR**: it is the resolution route of the ratified Deviation 3 and two report-card fields; not a separate direction |
| 6 | **G-Vendi as a diversity lever**: choose fine-tuning data to hold gradient-space diversity high and measure the output-diversity gain against single-answer accuracy; premise now established by [2604.16027](https://arxiv.org/abs/2604.16027); the "404" blocker in our records was wrong, the human data is public as `liweijiang/infinite-chats-*` (100 queries, 1,250 annotations, not the 26k corpus) | 3 | 3 | 150–300 | nfe1 diversity harness | ★★★½ | BENCH until H3's fate is decided; second-wave gate |
| 7 | **Planning-budget allocation on jepa-wms**: the paper never sweeps planning compute; borrow test-time-scaling budget allocation | 3 | 4 | 100–300 | none | ★★★ | BENCH (thin reframing, no asset reuse) |
| 8 | **Wei's verifiability rules as a released scorer** | 1 | 3 | <100 | none | ★★★ | BENCH |
| 9 | **ZID paired dispersion test** shipped in the nfe1 harness ([2608.24881](https://arxiv.org/abs/2608.24881)) | 2 | 3 | ~2 | nfe1 | ★★★ | ABSORB into the nfe1 harness when H3 resumes |
| 10 | Sony circuit-repair proposal (status check) | 1 | 3 | proposal | Sony | ★★★½ | already a proposal; two edits required, see the binding-feeds record |
| 11 | He's 1-NFE averaging test (live H3) | 1 | 4 | 0 new | H3 | ★★★★ | live; see [[Prereg-1NFE-Diversity]] for the tick-500 result |
| — | Puro-2B ([2608.27370](https://arxiv.org/abs/2608.27370)) as a from-scratch pretraining substrate | resource | — | ~15 to adopt | contextualization, sparsity | ★★★ enabler | ADOPT when a project needs controlled pretraining |

## Benched (keep, do not fund now)

Multilingual binding through the pooled score in SigLIP 2 (★★½, Level 3, needs
translation work) · injective leave-one-cell-out control (a TMLR control arm,
<20 GPU-h, not a direction) · Karpathy accept/reject ledger replay (★★; the
refresh's "±0.5% compute" and "no seed replication" claims are both false; the
real finding is that 37 single-run accept/reject calls sit beside a measured
seed spread 27 times the clearance margin; crowded room) · M. Li
static-to-active (★★) · entrance-locking reframed as checkpoint-interpolation
repair (★★½) · Karpathy and Wei items above.

## Killed (do not propose again), with the one reason each

| Item | Reason |
|---|---|
| Contrastive-score foveated follow-up (TMLR) | mechanism and headline already published by [2604.11496](https://arxiv.org/abs/2604.11496) (region-level readout inside the similarity score, frozen CLIP/SigLIP 2/PE); keep only as a TMLR citation and comparison |
| Beyer "where binding fails" halves | diagnosis; same paper as above plus [Foveated Probes](https://arxiv.org/abs/2608.00726) |
| Role-agnostic region readout as a method | Level 2; required as a TMLR fix (running now), not a paper |
| Training-set-size sweep vs Farnia | a readable pair costs ~1,236 H200-h; tick-500 arms are on the floor |
| Entrance-locking port (as written) | the one-step trajectory carries no measurable signal at tick 500 |
| SimFoundry r = 0.911 audit | check ran: it is a mean of seven correlations over 3–5 policies, not one at n = 7; no reframing survives |
| Replayability census | deliverable is a table |
| HarnessEval-W ablation | the refresh was wrong: it has a 5,000-judgment human study; blocked |
| OpenHands critic audit | validity (AUC vs merged PRs) is already in the source |
| Wei five properties vs saturation | scooped by the artifact itself |
| Fork-rate intervention on Arora | the method-shaped version is a 101-citation March paper |
| LpWM sparse-vs-dense replication | replication; reframed version Level 3 but the field moved |
| Memory Anchors replication | Level 1 |
| ARLI / Slow-Brain shared cause | at least ten groups on it; third paper 2026-08-30 |
| Shazeer equal-KV-bytes MQA from scratch | already answered by CLA [2405.12981](https://arxiv.org/abs/2405.12981); ~1,000 H100-h, not 400 |
| Liang replay-cause, Song GMP three-cause | already killed 2026-08-06 in [[Unified-Direction-Ranking-2026-08]] |
| Levine OGBench four-cause, Fan EgoScale, Sadigh polychromic | false premises, checked against the source papers |
| DriveJudge audit | the refresh conflated two projects with the same name (Fidler paper vs an EPFL repo); no auditable artifact for the target |
| Malik MOCHI + VLM arm | overtaken; our MOCHI arm already spent |
| M. Li AdaptVis-cause, B. Zhou both openings | Level 2 or infeasible |
| Neubig CAID 2×2, PACE future-time test | CAID killed 2026-08-05 (five answers incl. Nature MI 2026-07-24); PACE is not Neubig's paper |
| Choi cause-breakdown, generative side | Level 2 |

## Corrections to the refresh page

1. SimFoundry's r = 0.911 is a mean of seven per-task correlations over 3–5
   policies, never n = 7.
2. HarnessEval-W is validated (5,000 A/B judgments, Bradley–Terry, ρ = 0.93).
3. PRM-as-a-Judge 1.5's data went public on 2026-08-31 at
   [lyy0715/RoboPulsePlusPlus](https://huggingface.co/datasets/lyy0715/RoboPulsePlusPlus)
   (700 episodes, exactly 2,244 intervals, verified by download).
4. Karpathy's `dev/LOG.md` contains no "±0.5% compute" claim, and his
   leaderboard reports five-seed repeats.
5. The OpenHands critic ships a validity number (AUC), not only a speed number.
6. "DriveJudge" names two different projects; the refresh rated one and meant
   the other.
7. GR00T N1.7 is public ([nvidia/GR00T-N1.7-3B](https://huggingface.co/nvidia/GR00T-N1.7-3B),
   73k downloads); the Wiki-Citation-Audit line saying otherwise is stale.
8. PACE is not Neubig's paper, and "CAID" is never spelled out in our wiki.
9. Two April papers the refresh missed matter for the TMLR paper:
   [2604.11496](https://arxiv.org/abs/2604.11496) (states our "readout, not
   representation" thesis and ships a trained token-level alignment that
   lifts SugarCrepe Swap 63.3 → 76.3 on CLIP; its repo says "Coming soon")
   and [2608.15971](https://arxiv.org/abs/2608.15971) (already cited by us).

**Process rule, adopted:** before any scan lists an opening as "intact", it
must be matched against Part 2 and Part 3 of the current ranking; four
re-listed items today were already dead there.

## What happens next

Gates for ranks 1–4 run now under the two-pass rule of the prereg workflow
(a second, independent scoop pass; deep-read of the top threats; first
decisive step with a kill number; what the direction unlocks). Results go to
[[Method-Gates-Wave-4-2026-09]]. Rank 5 is folded into the ICLR paper's
Deviation 3 arm. Rank 9 is folded into the nfe1 harness.

## Related

[[Researcher-Scan-Refresh-2026-08-31]] · [[Unified-Direction-Ranking-2026-08]] ·
[[Method-Gates-Wave-3-2026-08]] · [[Prereg-RoboJudge-Audit]] ·
[[Prereg-1NFE-Diversity]] · [[Binding-Root-Cause-Analysis]]

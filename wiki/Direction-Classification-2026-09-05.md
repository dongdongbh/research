# Direction Classification — 2026-09-05: diagnostic or method

This page sorts every live direction from [[Direction-Audit-2026-09-01]] and
[[Method-Gates-Wave-4-2026-09]] into two kinds. The owner asked for this split
on 2026-09-05 ("divide directions to diagnostic and method").

## Words used on this page

- **Method** — the owner's definition: (1) a new mechanism, (2) an improvement
  on a current method, or (3) an old method applied to a new problem. The
  deliverable is a working thing that changes what a model or a pipeline does.
- **Diagnostic** — a study whose deliverable is a measurement: an audit, a
  probe, a replication, a statistic. A diagnostic can fund a method, but it is
  not one.
- **Unlocks** — for a diagnostic, the method paper its result would justify.
  A diagnostic with no named unlock is not worth running.
- **Week-1 step** — the cheapest run that decides whether the direction
  continues, with a kill number written before it runs.

## The split

| # | Direction | Kind | Route | Why it is that kind | Unlocks (diagnostics only) | Status 2026-09-05 |
|---|---|---|---|---|---|---|
| 1 | Role-preserving canonicalization ([[Gate-2026-09-01-canonicalization]]) | **Method** | 1 | Deliverable is a new way to fit the alignment map Q so role information survives; the week-1 measurement only decides whether the repair is needed | — | **DEAD** (week-1 result 2026-09-05: the rotation keeps role structure; the drop's interval sits below the 2-point line) |
| 2 | Horizon-calibrated world-model policy evaluator ([[Gate-2026-09-01-horizon]]) | **Method** | 3 | Deliverable is a tool: it returns the trusted rollout length H*, a corrected ranking, and an interval | — | **DIES, provisionally** (week-1 result 2026-09-06: τ(H*) − τ(H_max) interval contains zero in all three runnable environments; the horizon effect is at the short end) |
| 3 | Recorded-outcome anchor for generated-video judges ([[Gate-2026-09-01-dreamgen]]) | **Diagnostic** | 3, as an audit | Deliverable is a validity measurement of an existing judge; it ships as fields of the RoboJudge audit card, not as a new judge | The ICLR paper's audit protocol; a judge-repair paper only if the anchor shows a fixable failure (soft cut-off, not blindness) | **STRONG** (week-1 result 2026-09-05: shipped judge d′ 0.035; 88% of the gap is frame handling) |
| 4 | Sparsity-premise instrument ([[Gate-2026-09-01-sparsity]]) | **Diagnostic**, conditional | 1 only through the released instrument | The study can only refute a premise; the harness is a measuring device | The feature-admission training rule (measure the mutual information between each training-visible feature and the safety-critical query set; decorrelate above the measured threshold), with LARF as the baseline | Condition accepted by the owner 2026-09-05; design edits and pilot running |
| 5 | Discrimination-first evaluator report card | **Diagnostic** | — | Splits a judge's failure into cut-off and blindness; a measurement | The finer-score repair of a soft cut-off, which is now the Deviation 3 arm of the ICLR paper | Absorbed into the ICLR paper |
| 6 | G-Vendi as a diversity lever | **Method** | 3 | Chooses fine-tuning data to hold gradient diversity high; a training procedure change | — | Benched until H3's fate is decided |
| 7 | Planning-budget allocation on jepa-wms | **Method** | 3 | Transfers test-time budget allocation to world-model planning | — | Benched (thin, no asset reuse) |
| 8 | Wei's verifiability rules as a released scorer | **Method** | 1 | A scorer is a shipped mechanism | — | Benched |
| 9 | ZID paired dispersion test in the nfe1 harness | **Diagnostic** | — | A statistical test | The 1-NFE diversity method (H3) it would score | Absorbed into nfe1 when H3 resumes |
| 10 | Sony circuit-repair proposal | **Method** | 1 | Phases 2 and 3 are repairs (steering edit, re-routing, minimal targeted training); Phase 1 is the diagnostic that chooses among them | — | Proposal being edited with the PI |
| 11 | He's 1-NFE averaging test (H3) | **Method**, if the arm wins | 1 | The one-step averaging is a sampler change; the tick-500 result says it is unresolved | — | Live; see [[Prereg-1NFE-Diversity]] |
| — | Multilingual binding through the pooled score (benched) | **Diagnostic** | — | A cross-language probe | A multilingual readout repair, if the failure differs by language | Benched |
| — | Karpathy accept/reject ledger replay (benched) | **Diagnostic** | — | A replay of decisions against seed spread | A shipped promotion rule inside a training harness (would then be a method under the owner's statistics clause) | Benched |
| — | Entrance-locking as checkpoint-interpolation repair (benched) | **Method** | 2 | A repair to a training schedule | — | Benched |
| — | M. Li static-to-active (benched) | **Method** | 3 | A transfer of active data selection | — | Benched |
| — | Puro-2B pretraining substrate | **Resource** | — | Neither; an enabler for controlled pretraining | — | Adopt when needed |

## What the split says

- **Six live methods, four live diagnostics.** The four gates split two and
  two: gates 1 and 2 are methods; gates 3 and 4 are diagnostics that each
  name the method they unlock.
- **Every diagnostic above has a named unlock.** If a week-1 result kills the
  unlock (for gate 3: the judge is blind, not miscalibrated; for gate 4: the
  premise survives every band), the diagnostic stops there and does not
  become a paper on its own.
- **The pre-registration rule that follows:** a diagnostic gets a
  pre-registration page only together with the method it unlocks, so the
  kill number for the diagnostic and the entry condition for the method are
  written on the same page.

## Related

[[Direction-Audit-2026-09-01]] · [[Method-Gates-Wave-4-2026-09]] ·
[[Researcher-Scan-Refresh-2026-08-31]] · [[Unified-Direction-Ranking-2026-08]]

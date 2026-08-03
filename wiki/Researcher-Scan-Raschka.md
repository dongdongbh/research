# Researcher Scan — Sebastian Raschka (2026-08-02)

Single-person addendum to [[Top-Researcher-Scan-2026-08]], run under the
`researcher-scan` skill. Prompt-injection check on his pages: clean.

## Current true focus (verified, dated)

**No lab, no students** — "his lab" is a solo LLC ("Raschka AI Research
(RAIR) Lab LLC", newsletter footer); PyCon DE 2026 lists him as independent
with "no corporate affiliation". Ex-UW-Madison professor (resigned ~2022),
ex-Lightning AI (departure date unpinned). Output now: (1) architecture
reverse-engineering journalism at ~30 posts/2026 (200k subscribers) —
[Kimi K3 notes](https://sebastianraschka.com/blog/2026/kimi-k3-architecture-notes.html)
et al.; (2) books — *Build a Reasoning Model (From Scratch)* shipped
2026-06-30; (3) exactly one last-author research paper in five years:
[MiCA (arXiv 2604.01694)](https://arxiv.org/abs/2604.01694), Apr 2026,
2 authors, 0 citations. **No VLM/compositional work at all — no seam for
that lane here.**

## Artifact map (the real value)

- **[rasbt/LLMs-from-scratch](https://github.com/rasbt/LLMs-from-scratch)**
  (100k★, Apache-2.0, pushed 2026-07-29): `ch04/` holds **matched,
  CI-tested implementations of the entire 2026 efficiency zoo** — KV-cache,
  GQA, MLA, SWA, MoE, DeltaNet, DeepSeek sparse attention, KV-sharing —
  one codebase, one style. Kills the mixed-implementation confound in
  architecture arbitration. Pedagogical-first: use as **reference oracle
  and unit-test source, not as a training harness**.
- **[rasbt/llm-architecture-gallery](https://github.com/rasbt/llm-architecture-gallery)**
  (pushed 2026-08-02): 93 models with per-model KV-bytes/token, attention
  type, config links + diff tool — a **structured registry of unarbitrated
  vendor claims**. Our claim-mining input, maintained weekly, free.
- [rasbt/reasoning-from-scratch](https://github.com/rasbt/reasoning-from-scratch)
  (4.9k★): GRPO/eval/distillation on Qwen3-0.6B/1.7B — cluster-S substrate.
- `local-coding-agent-evals`: 5 tasks, single-run, no seeds, **no license**
  — evidence-quality warning, not a substrate.

## Claims that outrun evidence

1. **MiCA omits MiLoRA** ([NAACL 2024, ~96 cites](https://arxiv.org/abs/2406.09044)
   — minor-singular-component PEFT) from its references. **Gate correction
   (2026-08-02, deep read):** the scan's other two charges were wrong —
   MiCA's LoRA arm IS budget-matched (independently searched rank/LR/epochs
   both sides; only full-FT is asymmetric), it cites
   [2602.04998](https://arxiv.org/abs/2602.04998), and §6 already runs the
   random-subspace control (Minor 75.63 > Random 73.75 > Major 74.21 at
   matched everything). Its real defects: no MiLoRA/PiSSA arm, selection on
   test curves, probes of 300/102 items (SE 2.8–4.8pp vs 1.8–3pp claimed
   gaps), unreleased data.
2. **Reasoning-effort control is 4 uncomparable recipe families** (his own
   Jul 2026 admission): prompt-conditioning / RL length penalties /
   mixed-mode SFT / specialist-distill (K3 trains 9 specialists). Partially
   claimed (ThinkDial); no multi-level 4-family arbitration exists.
3. **NoPE-everywhere as frontier default is untested independently** — the
   live cell is the *conditional* claim (NoPE viable only with linear/KDA
   layers carrying position), but expect S-M1-class effect sizes —
   underpowered at our budget.
4. mHC/attention-residual ablations — **saturated 2026 lane, do not enter**.
5. His empirical posts are n=5 single-run, no seeds — the man who wrote
   the canonical model-selection tutorial ships none of that rigor in his
   LLM-era evals. That gap is cluster V's thesis in one person.

## Openings (claimed-or-not checked)

| # | Study | Cost | Verdict | ★ |
|---|---|---|---|---|
| O1 | **Spectral band × knowledge-injection regime arbitration** — GATED 2026-08-02, demoted: the LR-artifact question is answered 4× for task adaptation ([2602.04998](https://arxiv.org/abs/2602.04998) covers PiSSA+MiLoRA; [2602.03493](https://arxiv.org/abs/2602.03493) band sweep + forgetting; [LoRAFactory 2601.22708](https://arxiv.org/abs/2601.22708); [batch-size bias 2602.09492](https://arxiv.org/abs/2602.09492)). Surviving cell: band × knowledge injection + TOST equivalence + released contamination-screened benchmark (3-tier: post-cutoff real / synthetic biographies / paired-regression forgetting). Kill-arm ladder: K1 = 10 GPU-h MiCA repro gate (no code/data — fail kills the project). Venue ICML/COLM; ICLR impossible. Distribution edge is ONE-SIDED (he won't amplify a debunk of his own paper) | 400–600 GPU-h (150–200 de-scoped) | SURVIVES-NARROWED | ★★ |
| O2 | **4-family reasoning-effort-control arbitration** on his reasoning-from-scratch substrate; deliverable = recipe + effort-conditioned checkpoints; must include ThinkDial arm | 300–600 GPU-h | SURVIVES-NARROWED | ★★½ |
| O3 | Adopt `ch04` implementations as arbitration oracles + Gallery as claims registry | ~0 | infrastructure, not a paper | — |
| O4 | Harness-vs-model variance on his task pack | 50–120 | crowded (OctoBench, AgentSpec, ReliabilityBench 2026) — 5 GPU-h kill-arm at most | ★½ |

Distribution note: 200k subscribers + he writes up others' papers weekly —
an arbitration of a claim he raised has an unusually short path to
visibility.

## Star

**★★★** — 2 as a person to track (no lab, no scoop risk, one preprint),
**4 as a claims-and-substrate feed**: he does our claim-mining step for
free, ships the matched implementations to test the claims, and will never
run the arbitrations himself.

## Convergence with the scan clusters

**S** — strong: he is the small-scale regime's supply chain (oracle, not
harness). **A** — he is the best public *index* of
claims-outrunning-arbitration. **V** — witness and target, not
contributor. **D** — zero engagement; expect nothing.

## Coverage caveats

WebSearch budget exhausted early (Scholar never attempted); S2 rate-limited
throughout; arXiv Atom API empty via curl (worked around via OpenAlex +
WebFetch); Lightning departure date inferred, not confirmed.

## Related

[[Top-Researcher-Scan-2026-08]] · [[Method-Gates-Wave-2-2026-08]] ·
[[Direction-Reevaluation-2026-08]]

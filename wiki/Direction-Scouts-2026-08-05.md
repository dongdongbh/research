# Direction Scouts — 2026-08-05

Three new research areas scouted in parallel by three Opus agents, at the
owner's request: (1) generative video and video-to-audio, (2) LLM/VLM
models for robots, (3) AI for science and automated ML. Each scout mapped
its field, checked every proposed opening against published work, and
priced it against our budget (100–1000 GPU-hours, no robot hardware, no
video-model training).

**Read the stars with care: these are scout ratings, before a formal
novelty check.** This month, scout ratings dropped by about one star on
average once a deep check ran. The recommendation at the end is to
formally check only the top few.

## The combined ranking (all three directions)

| Rank | Opening | From | Cost (GPU-h) | Scout ★ | What it connects to |
|---|---|---|---|---|---|
| 1 | **Sim-and-world-model evaluator uncertainty audit** — published sim-vs-real ranking claims rest on 5 policies with no confidence intervals; measure the rank-stability floor of simulators against themselves; ship an "evaluator power calculator" | Robotics | 200–400 | ★★★★★ | The sibling of RoboJudge: it audits the *environments*, RoboJudge audits the *judges* — a two-paper arc on shared data and code |
| 2 | **Video mode-coverage measurement** — every "distillation kills diversity and we fixed it" paper measures spread, not coverage; no video coverage metric exists; build it and re-test the claims | Video | 250–400 | ★★★★½ | The video arm of our active 1-NFE diversity project (same question, new modality) |
| 3 | **Language-necessity index for robot benchmarks** — robot models largely ignore instructions, yet all benchmarks report language-conditioned scores; measure the blind floor per task, re-weight leaderboards | Robotics | 100–250 | ★★★★ | RoboJudge's blind-floor arm, pointed at environments |
| 4 | **Video-to-audio metric validation** — the metrics grading generated soundtracks were never checked against human listeners (music and text-to-audio were; video-to-audio was not) | Video | 80–150 + listening tests | ★★★★ | Our judge-audit template, new target |
| 5 | **T2V role direction** — "a dog chases a cat": no video-generation benchmark tests direction; our binding program pointed at generators | Video | 300–600 | ★★★★ | Binding program + a future SWAP category donated to T2V-CompBench |
| 6 | ~~Seed-noise floor of ML-engineering-agent leaderboards~~ — **VETOED by owner 2026-08-05**: effect not significant enough to matter, anyone can run it, not novel. Closed along with opening #8 (the merged design-layer arm). | AI4Sci | — | closed | — |
| 7 | Powered replication of Anthropic's "LLM supervision helps robots" claim (their n=36; code unreleased — repo 404 verified) | Robotics | 100–200 + API | ★★★½ | Standalone; supplies trajectories for #1/#3 |
| 8 | Statistical design layer for agent-proposed experiments (blocking/replication as a module) | AI4Sci | 150–350 | ★★★½ | Merges with #6 as one project, two arms |
| — | Not worth running: VLA binding probe (crowded), V-JEPA role probe (bundle-only), novelty forensics (owned by biology-side work) | | | ≤★★★ | |

## What each scout found, in one paragraph

**Robotics.** The Anthropic page shows 12 models on 5 control levels:
models mostly fail at driving joints directly, and "supervising a trained
policy helps" rests on 36 trials with no released code. The field itself
has moved from "build a better robot model" to "trust our evaluator" —
and the scout's deep read found the flagship trust claims are computed
over five policies with no uncertainty reported anywhere. Full report
links in the task record; key papers: [LIBERO-Plus
(2510.13626)](https://arxiv.org/abs/2510.13626) ("models tend to ignore
language instructions completely"), [sim-real recipe
(2606.10366)](https://arxiv.org/abs/2606.10366) (deep-read: no CIs).

**Video.** Open models (Wan, Hunyuan, LTX) made evaluation the bottleneck.
Three measurement holes: diversity measured as spread rather than
coverage, video-to-audio metrics never human-validated
([FoleyBench (2511.13219)](https://arxiv.org/abs/2511.13219) correlates
metrics against other metrics), and no benchmark tests who-does-what-to-whom
in generated video ([T2V-CompBench V2 (2407.14505)](https://arxiv.org/abs/2407.14505)
and [VBench-2.0 (2503.21755)](https://arxiv.org/abs/2503.21755) have no
direction category — verify by cloning the repo before any plan locks).

**AI for science (strictest scout).** Mostly confirmed kills: the validity
layer went from open to crowded in eight weeks (claim-integrity,
fabrication rates, benchmark audits, even an audit-of-audits); classical
optimizers beating LLM agents at matched compute
([2603.24647](https://arxiv.org/abs/2603.24647)) closes the AutoML cell
the way AHD died. One clean survivor: nobody in 36 ML-agent-benchmark
papers audits run-to-run noise, even though
[MLE-bench (2504.01848)](https://arxiv.org/abs/2504.01848) itself
prescribed seeds and standard errors. Six-week re-check clock on anything
chosen here — this layer moves fast.

## The pattern, stated once

All eight viable openings are measurement-validity work that reuses
infrastructure we already own. None requires training a large model. Each
direction's best opening extends an existing project rather than starting
from zero: robotics → RoboJudge, video → 1-NFE diversity + the binding
program, AI-for-science → our statistics house rules.

## Recommended next steps (not yet done)

1. Formal novelty checks (with the second adversarial sweep and an
   OpenReview pass — all three scouts had no web-search budget and could
   not see under-review papers) on **#1 and #2 only**.
2. Decide at the sign-off meeting whether either enters the next-cycle
   slate; the current cycle is full.
3. Slate note: #2 (video coverage) could extend the existing 1-NFE CVPR
   prereg rather than compete with it — one team, one framework, two
   modalities.

## Coverage caveats (all three scouts)

Session web-search budget was exhausted before the scouts ran; DBLP and
OpenReview were rate-limited or down. All verdicts rest on arXiv/Semantic
Scholar/OpenAlex/Crossref plus targeted full-PDF reads. Under-review
work is invisible; every "not claimed" verdict needs the standard second
pass before anything is committed.

## Related

[[Method-Gates-Wave-3-2026-08]] · [[Compositional-VLM-Survey]] ·
[[Prereg-RoboJudge-Audit]] · [[Prereg-1NFE-Diversity]] ·
[[Top-Researcher-Scan-2026-08]]

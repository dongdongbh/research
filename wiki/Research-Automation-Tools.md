# Research Automation Tools

## Installed

Installed globally for Codex:

```text
/home/dli160/.codex/skills/codex-autoresearch
```

Source:

```text
https://github.com/leo-lilinxiao/codex-autoresearch
```

Why it fits SVIB2:

- Codex-native skill for measurable improve/verify loops.
- Useful once SVIB2 has a stable metric, such as VLM2Vec+SVIB2 pair accuracy,
  selective accuracy, or verifier training loss.
- Supports foreground/background runs and keeps experiment logs under
  `autoresearch-results/`, which should remain uncommitted.

Good first uses:

```text
Improve VLM2Vec+SVIB2 evaluation on a fixed tiny SCPP++/VSR sample.
Reduce missing-score failures in cached baseline generation.
Improve test coverage for the training/evaluation CLI surface.
```

## Not Installed

`https://github.com/uditgoenka/autoresearch`

- It now documents Codex support, but it overlaps strongly with
  `codex-autoresearch`.
- It has a larger command and hook surface. Keep it as a reference for now
  rather than installing a second autoresearch loop framework.

`https://github.com/alvinreal/awesome-autoresearch`

- It is a curated index, not a directly installable Codex skill.
- Keep it as a discovery source for later, especially if SVIB2 needs a more
  specialized GPU experiment runner or literature-review workflow.

## Use Policy

Do not use autoresearch for ordinary one-shot implementation work. Use it when
the goal has a mechanical metric, a bounded scope, and a verification command.
For SVIB2, that means benchmark accuracy, selective risk, latency, training
loss, cache coverage, or test coverage.

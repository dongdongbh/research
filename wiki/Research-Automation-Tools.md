# Research Automation Tools

Updated 2026-08-02 for the general research wiki.

## Installed

Installed globally for Codex:

```text
~/.codex/skills/codex-autoresearch
```

The originally recorded absolute path was `/home/dli160/.codex/skills/codex-autoresearch`,
which is **account- and machine-specific** (the account names differ per
cluster: Delta account `dli26`, Anvil account `x-dli26`). Resolve `~/.codex/skills`
on the machine you are on rather than copying the absolute path; on Anvil this
skill is not currently installed under `/home/x-dli26/.codex/skills`.

Source:

```text
https://github.com/leo-lilinxiao/codex-autoresearch
```

Why it fits our work:

- Codex-native skill for measurable improve/verify loops.
- Useful once a project has a stable metric, such as pair accuracy, selective
  accuracy, or training loss.
- Supports foreground/background runs and keeps experiment logs under
  `autoresearch-results/`, which should remain uncommitted.

Good first uses (phrase the goal as a metric plus a fixed evaluation slice):

```text
Improve <model+method> evaluation on a fixed tiny benchmark sample.
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
- Keep it as a discovery source for later, especially if a project needs a more
  specialized GPU experiment runner or literature-review workflow.

## Use Policy

Do not use autoresearch for ordinary one-shot implementation work. Use it when
the goal has a mechanical metric, a bounded scope, and a verification command:
benchmark accuracy, selective risk, latency, training loss, cache coverage, or
test coverage.

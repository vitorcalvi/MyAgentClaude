# Operator setup

Generic install notes. This is not a dump of any one author's laptop.

## Prerequisites

- Claude Code
- `goose` CLI, with MiniMax configured as the active provider and model `MiniMax-M3[1m]` (or the current MiniMax coding model your provider block expects)
- `MINIMAX_API_KEY` in the environment of the shell that launches `goose`
- Optional: `agy` (Google Antigravity CLI) on `PATH` for Lane B
- Optional failover CLIs only if you add them yourself (not required by this spec)

Never print the API key. Never commit provider configs that embed secrets.

## Layout

```text
your-project/
  CLAUDE.md                 # copy from this repo
  agy-swarm/
    AGY_COORDINATOR.md
    SWARM_PROTOCOL.md
    swarm-team.yaml
  .runs/                    # gitignored; created per swarm run
```

Point `agy` persistent context at `agy-swarm/` (for example by copying `AGY_COORDINATOR.md` to the file your CLI already loads as project instructions).

## Sanity checks

```bash
command -v goose
command -v agy          # optional
[ -n "$MINIMAX_API_KEY" ] && echo set   # never echo the value
agy models              # if using Lane B; copy a slug exactly
```

## What Claude is allowed to run

Claude may only compose briefs and invoke `goose run` / `agy --print=...`. Claude may not Read/Edit/Bash the target repo "to be helpful." If a dispatch fails with a missing key, stop and tell the user. If a dispatch fails with quota on goose, follow the failover you have actually configured; do not invent runners.

## Safety preamble (every brief)

```
Ground every claim in this repo: grep/read before asserting a function, field, or file exists.
If you cannot verify something from the actual codebase, stop and state exactly what's blocking
you — don't invent it.
Do not shell out to goose run or spawn another agent process under any circumstance.
Keep your final answer concise.
Never cat/read the full contents of a file that may hold a credential.
```

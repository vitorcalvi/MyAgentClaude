# Media kit

**One line:** Claude is the architect. Cheap workers write and test. A third worker grades the evidence. Nobody grades their own work.

## Boilerplate (120 words)

MyAgentClaude is an open operating protocol for Anthropic's Claude Code. It treats Claude as a planner only: Claude does not read, edit, or test the target repository. Implementation runs on a MiniMax worker pool driven by the goose CLI, with at most five concurrent workers. Large goals can be handed once to Google's Antigravity CLI (`agy`), which decomposes work into a DAG of briefs, retries ordinary failures, and returns a single report. A separate goose worker then verifies raw artifacts against the user's original request. The design exists to stop frontier models from burning quota on mechanical edits and to stop agents from grading their own success. The repository is a specification, not a hosted product.

## Pull quotes

> "Claude is the architect. Cheap workers write and test. A third worker grades the evidence. Nobody grades their own work."

> "A worker that reports a clean build while the file was never written is a known failure mode. That is why verification is always a different process."

> "Quota exhaustion and false assumptions are not retries. They are stops."

## Diagram caption

Flow of MyAgentClaude: Claude plans; small tasks go to a MiniMax/goose worker pool; large tasks go through an optional `agy` coordinator; both paths merge into an independent verifier that reads raw run artifacts and the user's original request.

Use [docs/diagram-prompt.md](diagram-prompt.md) to render a wide PNG. Commit the PNG as `docs/architecture.png` when generated. Until then, GitHub renders the Mermaid chart in the README.

## Facts you can print

- Not a startup product. Open spec under MIT.
- Three roles: architect (Claude), workers (goose + MiniMax-M3), optional coordinator (`agy`).
- Hard cap: five concurrent workers, two ordinary retries per brief.
- Verification is disjoint: the coordinator cannot see, shape, or grade it.
- Author: Carlos Vitor Botti Calvi — vcalvi@gmail.com — https://github.com/vitorcalvi/MyAgentClaude

## Facts you must not invent

Do not cite monthly prices, token allotments, concurrent-agent counts from third-party blogs, or "1 million token context in goose" as proven. Do not describe MiniMax as a second installed CLI. Do not describe Perplexity/GPT/Kimi research as automated.

## Contact

vcalvi@gmail.com
https://github.com/vitorcalvi/MyAgentClaude

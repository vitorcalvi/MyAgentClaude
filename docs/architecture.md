# Architecture

Claude is the architect. Cheap workers write and test. A third worker grades the evidence. Nobody grades their own work.

## Tiers

1. **Claude** — Writes one plan. For small work: 3–5 bounded briefs. For large work: one GOAL packet. Reads worker reports and the verifier report. Does not open target source, diffs, or test output.
2. **`agy` (optional)** — Google Antigravity CLI as tactical coordinator. Used only when a goal decomposes into 3+ dependent briefs. Decomposes, assigns, retries ordinary failures, consolidates. Never verifies. Never marks the overall goal successful.
3. **`goose` / MiniMax-M3** — The only worker runtime. Roster IDs `goose-a`, `goose-b`, `minimax-a`, `minimax-b` are roles/task-shapes on that same runtime. `goose-verify` is reserved and excluded from the implementation fan-out.

If `agy` hits quota, timeout, or infra failure, Claude coordinates the fan-out directly. `agy` is not a hard dependency.

## Two lanes

**Lane A** — recon or a single-file change. Claude launches at least three and at most five concurrent goose workers. Competing edits use separate git worktrees. Claude selects a candidate from reports only.

**Lane B** — large DAG. Claude dispatches `agy` once with:

```text
GOAL:
<one sentence>

CONTEXT (optional):
<files, constraints, prior outputs>

DECOMPOSITION SKETCH (optional):
<Claude's prior; agy follows unless there is a structural reason not to>

HARD CONSTRAINTS:
<mechanically checkable>
```

agy splits into at most five briefs, each with `id`, `objective`, `inputs`, `constraints`, `done_criteria`, `assigned_worker`, `depends_on`, `budget_hint`.

## Short circuits

| Class | Retry? | Effect |
| --- | --- | --- |
| TOOL_ERROR / MODEL_ERROR / CRITERIA_NOT_MET / TIMEOUT | Yes, max 2 | Attempt 2: same worker class + failure tail. Attempt 3: switch task-shape (code vs prose) + slimmer brief. Never weaken `done_criteria`. |
| QUOTA_EXHAUSTED | No | Brief BLOCKED; cascade to queued briefs on that worker class |
| ASSUMPTION_INVALID | No | Brief BLOCKED; cascade to dependents; Claude must revise the goal |

A BLOCKED brief is never folded into an apparent full success.

## Verification

Claude writes `verify/verify-brief.md` from the original user request, the GOAL, hard constraints, each brief's `done_criteria`, and paths to **raw** worker outputs — not `consolidated-report.md` as evidence.

The verifier starts with `find <run_dir> -type f` and adapts. Real runs may emit `.log` / `.md` instead of the documented JSON schema.

Pass/fail is per criterion with evidence paths. Claude replies to the user only after reading that raw report.

## Artifact layout

```text
.runs/<run_id>/
  goal.md
  briefs/<brief_id>.md
  results/<brief_id>.attempt-<n>.json
  retry-ledger.json
  consolidated-report.md
  verify/verify-brief.md
  verify/raw-verify-report.md
```

Illustrative example: [examples/run/](../examples/run/).

## Dispatch shapes

Goose default:

```bash
goose run --no-session --quiet --text "$(cat <<'BRIEF_EOF'
<brief>
BRIEF_EOF
)"
```

agy (flag order matters; `--print` takes the prompt with `=`):

```bash
agy --model "<slug-from-agy-models>" --dangerously-skip-permissions \
  --print-timeout 30m --print="$(cat <<'AGY_GOAL_EOF'
GOAL:
...
AGY_GOAL_EOF
)"
```

Use an exact slug from `agy models`, never a display name.

## Non-goals

- Native Claude Code subagents, Agent Teams, or `ANTHROPIC_BASE_URL` splitting. Those cannot put workers on a non-Anthropic token pool.
- Automating Perplexity, GPT-5.6, or Kimi K3.
- Nested `goose run` from inside a worker.

# AGY SWARM PROTOCOL

Message formats, schemas, and the parallel fan-out loop for the agy-coordinated swarm.
Companion to `AGY_COORDINATOR.md` (roles and policies) and `swarm-team.yaml` (roster).

## 1. Chain of custody

```
[Claude] goal.md                     → [agy]
[agy]    briefs/*.md + ledger.json   → [workers]
[workers] result.json + output       → [agy]
[agy]    consolidated-report.md      → [Claude]
[Claude] verify-brief.md             → [fresh goose worker]  (agy absent)
[verifier] raw-verify-report.md      → [Claude] → user
```

Every artifact is a file in the run directory `.runs/<run_id>/` so the verification
worker can read raw inputs without asking agy anything.

Run layout:

```text
.runs/<run_id>/
  goal.md
  briefs/<brief_id>.md
  results/<brief_id>.attempt-<n>.json
  retry-ledger.json
  consolidated-report.md
  verify/verify-brief.md        # written by Claude, not agy
  verify/raw-verify-report.md   # written by goose verifier
```

## 2. Brief schema

```json
{
  "id": "B1",
  "objective": "Rewrite retry logic to use exponential backoff",
  "inputs": ["src/swarm/dispatch.py", "GOAL.md"],
  "constraints": ["no new dependencies", "keep public API"],
  "done_criteria": ["dispatch.py compiles", "pytest swarm/ green", "backoff max 3 levels"],
  "assigned_worker": "goose",
  "assignment_reason": "touches code + runs tests",
  "depends_on": [],
  "budget_hint": "small"
}
```

`done_criteria` are observable and checkable by a third party (the verifier).

## 3. Worker result schema

```json
{
  "brief_id": "B1",
  "worker": "goose",
  "attempt": 1,
  "status": "DONE",
  "output_ref": "results/B1.attempt-1/",
  "summary": "<= 20 lines, factual",
  "done_criteria_check": [
    {"criterion": "dispatch.py compiles", "met": true, "evidence": "output_ref/build.log"}
  ],
  "artifacts": ["src/swarm/dispatch.py"],
  "errors": []
}
```

Failure result: `status: "FAILED"`, `errors: [{class, message, tail}]`.
Error classes: `TIMEOUT`, `TOOL_ERROR`, `MODEL_ERROR`, `CRITERIA_NOT_MET`, `QUOTA_EXHAUSTED`, `ASSUMPTION_INVALID`.

> The `worker` field is a **roster id** from `swarm-team.yaml` (`goose-a`, `goose-b`, `minimax-a`, `minimax-b`, `goose-verify`). The runtime is always `goose`; `minimax-*` IDs are task-shapes (prose / long-context) routed onto the same executable.

## 4. Agy fan-out loop

```python
def run_swarm(goal, context, hard_constraints):
    briefs = decompose(goal, max_briefs=5)          # agy decides
    ledger, retry_ledger = [], []

    ready = [b for b in briefs if not b.depends_on]
    while ready or in_flight:
        while ready and len(in_flight) < MAX_WORKERS:      # cap = 5
            b = ready.pop()
            in_flight[b.id] = dispatch(b)                  # goose or MiniMax, agy decides

        result = next_finished(in_flight)                  # async collect

        if result.status == "DONE":
            ledger.append(result); del in_flight[result.brief_id]
            ready += dependents_of(result.brief_id)        # unlock DAG edges
        else:
            a = attempts(result.brief_id)
            if a < MAX_RETRIES:                            # cap = 2
                retry = build_retry(result)                # see §5
                in_flight[result.brief_id] = dispatch(retry)
                retry_ledger.append((result.brief_id, a + 1, result.error_class))
            else:
                result.status = "BLOCKED"
                ledger.append(result); del in_flight[result.brief_id]

    return consolidate(ledger, retry_ledger)               # ONE report → Claude
```

## 5. Retry construction

| Attempt | Worker class | Brief changes |
|---|---|---|
| 2 (retry 1) | same class | append failure context verbatim (error tail); criteria unchanged |
| 3 (retry 2) | switch class (goose ↔ MiniMax) | slim scope; sharpen `done_criteria`; never weaken them |
| — | — | if it fails: `BLOCKED` |

## 6. Consolidated report (agy → Claude)

```markdown
# Consolidated report — <run_id>
Goal: <one line>

## Status
| brief | objective | worker | attempts | status |
|---|---|---|---|---|
| B1 | ... | goose | 1 | DONE |
| B3 | ... | minimax | 3 | BLOCKED (TOOL_ERROR) |

## Deliverables
<merged outputs in brief order>

## Conflicts / uncertainties
<B2 says X, B4 says Y — kept X because ... | unresolved>

## Retry ledger
<brief_id, attempt, worker, error_class, action_taken>

## Artifacts
<paths>
```

## 7. Independent verification (Claude only, agy absent)

1. Claude writes `verify/verify-brief.md` containing: the original goal, hard
   constraints, per-brief `done_criteria`, and refs to the **raw worker outputs** —
   not agy's consolidation, not agy's conclusions.
2. Claude dispatches a **fresh goose worker** (clean session/workspace) with that brief.
3. The verifier checks each `done_criterion` against the artifacts and returns
   `PASS / FAIL per criterion + evidence paths`.
4. Claude reads `raw-verify-report.md` itself. Only then does it reply to the user —
   reporting its own verdict, not agy's.

This path never passes through agy. Agy cannot see it, shape it, or grade it.

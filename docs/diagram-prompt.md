# Architecture diagram — image prompt

Paste into an image model (Gemini or equivalent) to render `docs/architecture.png`. Wide infographic, not photorealistic.

## Prompt

Create a clean, modern horizontal flowchart infographic, wide-aspect pipeline left to right, technical architecture-diagram style, flat design, soft color-coded boxes with rounded corners, thin arrows with visible arrowheads, legible bold sans-serif labels, white or light-gray background, minimal icons, no clutter.

Flow, strictly left to right, with two lanes that both merge into one point:

1. START box: "New Claude Code chat" →
2. "Claude — Planner/Architect only (never reads/edits/runs code directly)" →
3. Diamond: "Task size?" splitting into two parallel lanes stacked, ending at the same horizontal position.

Lane A, top, label "Small / single-file / recon":
A1 "Claude composes 3-5 bounded briefs" → A2 "Dispatch via goose run → MiniMax-M3 workers (at most 5 concurrent)" → A3 "Pick best candidate from reports"

Lane B, bottom, label "Large task — 3+ dependent briefs":
Amber dashed badge: "Scope discipline: chunk into 2-3 smaller runs if a wrong guess would be costly"
B1 stacked lines: "Claude writes ONE goal" / "GOAL — one sentence" / "CONTEXT — files, constraints" / "DECOMPOSITION SKETCH optional prior" / "HARD CONSTRAINTS — mechanically checkable"
→ B2 "Dispatch to agy (Google Antigravity CLI) — once"
→ B3 "agy decomposes into a DAG of at most 5 briefs"
→ B4 "agy dispatches goose/MiniMax workers, ordinary retries max 2"
→ B5 stacked: "QUOTA_EXHAUSTED → BLOCKED, no retry" / "ASSUMPTION_INVALID → BLOCKED + cascade"
→ B6 "One consolidated report — BLOCKED briefs listed plainly"

Fallback: a short red dashed curved arrow under B2 looping to B1, label "fallback: quota/timeout/infra fail only". Keep it local to B1/B2. Do not span B3-B6.

Merge A3 and B6 into a junction dot, then:
4. "Verification — Claude's job alone" with "reads RAW artifacts" and "checks the user's ORIGINAL request"
→ "Fresh independent goose worker → PASS/FAIL"
→ "Claude reads the raw verification report"
→ END "Claude replies; BLOCKED pieces flagged"

Detached dashed side-box off the left of box 2, dotted line "for deep research only":
"Perplexity / GPT-5.6 / Kimi K3 — manual hand-off"

Footer strip: "Ground every claim · no self-spawned agents · never read files that may hold credentials"

Colors: Claude blue, goose/MiniMax green, agy purple, scope badge amber dashed, short-circuit orange, verification red outline, manual gray dashed, fallback arrow red.

## Negative prompt

No duplicated words in labels. No photorealism, 3D, drop shadows, gradients, textures, watermarks, sketch style, isometric view. No arrows through text. Do not omit the merge junction. Do not add extra boxes. Do not truncate text. No extra colors. No duplicate or mirrored boxes.

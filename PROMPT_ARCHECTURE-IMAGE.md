  Prompt:

  Create a clean, modern horizontal flowchart infographic, wide-aspect single-line pipeline (left to right),
  technical/architecture-diagram style, flat design, soft color-coded boxes with rounded corners, thin arrows with visible arrowheads,
  legible bold sans-serif labels, white/light-gray background, minimal icons, no clutter.

  Flow, strictly left to right, with two lanes that both explicitly merge into one point:

  1. [START] box: "New Claude Code chat" → arrow to
  2. "Claude — Planner/Architect only (never reads/edits/runs code directly)" → arrow to
  3. Diamond decision node: "Task size?" splitting into two parallel lanes, stacked one above the other, ending at the same horizontal
     position so their arrows converge cleanly:

     Lane A (top row, section label "Small / single-file / recon"):
     A1 "Claude composes 2-5 bounded briefs" → A2 "Dispatch via goose run → MiniMax-M3 workers (up to 5 concurrent)" → A3 "Pick best
     candidate from reports" — LAST box of Lane A.

     Lane B (bottom row, section label "Large task — 3+ dependent briefs"):
     Small amber dashed badge above the lane: "Scope discipline: chunk into 2-3 smaller runs if a wrong guess would be costly to
     discover only at the end"
     B1, a wide box with four stacked lines inside it: "Claude writes ONE goal:" / "GOAL — one sentence" / "CONTEXT — files,
     constraints" / "DECOMPOSITION SKETCH (optional) — Claude's own brief breakdown + dependency guesses, for novel tasks; a strong
     prior, not a mandate" / "HARD CONSTRAINTS — must be mechanically checkable, never a qualitative judgment"
     → arrow to B2 "Dispatch to agy (Google Antigravity CLI) — once" → arrow to B3 "agy decomposes into its own DAG of ≤5 briefs —
     follows Claude's sketch if given, else own judgment (tested sound either way)" → arrow to B4 "agy dispatches goose/MiniMax
     workers, retries ordinary failures (max 2)" → arrow to B5, a box with two stacked short-circuit lines inside it: "QUOTA_EXHAUSTED
     → no retry, BLOCKED immediately" / "ASSUMPTION_INVALID → no retry, BLOCKED + cascades to dependent briefs" → arrow to B6 "agy
     returns ONE consolidated report — any BLOCKED brief listed plainly, never hidden inside an apparent success" — LAST box of Lane
     B, positioned directly below A3.

     Fallback arrow (keep this small and local — do NOT draw a long line across the whole diagram): a short red dashed curved arrow
     drawn immediately underneath box B2 ("Dispatch to agy") only, looping up to point at box B1 directly above it, with a small label
     on the arrow: "fallback: quota/timeout/infra fail only". This arrow must stay compact, contained between B1 and B2 — it must NOT
     extend across B3/B4/B5/B6.

     Explicit merge: directly below A3 and B6 (vertically aligned as the last box of each lane), draw two short arrows — one from A3
     pointing straight down, one from B6 pointing straight down then left — both meeting at a small filled junction dot, continuing as
     one arrow into:
  4. "Verification — Claude's job alone, never delegated": one wide box with two labeled sub-bullets: "① reads RAW artifacts by
     finding the run directory first — never assumes a fixed file schema" and "② checks against the user's ORIGINAL request, not just
     Claude's compressed goal text — catches scope drift" → arrow to "Fresh independent goose worker checks evidence → PASS/FAIL"
  5. → arrow to "Claude reads the raw verification report"
  6. → arrow to [END] box: "Claude replies to user — any BLOCKED piece flagged explicitly, with a re-dispatch decision"

  Add one small detached side-box (dashed border, positioned off to the left, connected to box 2 with a dotted line labeled "for deep
  research only") reading: "Perplexity / GPT-5.6 / Kimi K3 — manual hand-off: Claude states the question, user runs it, reports back 
  (no automation)"

  Add a thin footer strip along the very bottom, full width, reading: "Every brief carries a standing preamble: ground every claim · 
  no self-spawned agents · never cat/read files that may hold credentials, even as an intermediate step"

  Color coding: Claude boxes blue, goose/MiniMax boxes green, agy boxes purple, the scope-discipline badge amber/dashed, the
  short-circuit box (B5) orange, verification boxes orange/red outline, the manual hand-off box gray/dashed, the fallback arrow red.
  Keep all box text short and legible; B1's four stacked lines and B5's two stacked lines may use smaller text than other boxes since
  they hold more content.

  Negative prompt / things to avoid:
  No duplicated words, letters, or punctuation in any text label — render every string exactly once (check especially "Google
  Antigravity CLI) — once" and "agy decomposes into its own DAG"). No photorealism, no 3D, no drop shadows, no gradients, no
  decorative textures, no watermarks, no hand-drawn/sketch style, no isometric perspective. Do not let any arrow cross through text or
  overlap another box. Do not omit the merge junction between Lane A and Lane B — both lanes must visibly connect into the
  Verification box, not dead-end. Do not draw the fallback arrow as a long line spanning the diagram — keep it short and local to the
  B1/B2 area only. Do not add extra boxes, icons, or steps not listed above. Do not truncate, cut off, or overlap any text. Do not use
  more than the six specified colors. Do not add a border/frame around the entire image beyond a plain background. Do not render
  duplicate or mirrored copies of any box.


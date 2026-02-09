## Continuity Ledger (compaction-safe)
Maintain a single Continuity Ledger for this workspace in `CONTINUITY.md`.The ledger is the canonical session briefing designed to survive context compaction; do not rely on earlier chat text unless it's reflected in the ledger.

### How it works:
-At the start of every assisstant turn:read `CONTINUITY.md`,update it to reflect the latest goal/constraints/decisions/state,then proceed with the work.
-Update `CONTINUITY.md` again whenever any of these change:goal,constraints,assumptions,key decisions,progress state(Done/Now/Next),or important tool outcomes.
-Keep it short and stable:facts only,no transctipts.Prefer bullets.Mark uncertainty as `UNCONFIRMED`(never guess).
-if you notice missing recall or a compaction/summary event:refresh/rebuild the ledger from visible context,mark gaps `UNCONFIRMED`,ask up to 1-3 targeted questions,then continue.

### `function.update_plan` vs the ledger
-`function.update_plan` is for short-term execution sacffolding while you work(a small 3-7 step plan with pending/in_progress/completed)
-`CONTINUITY.md` is for long-running continuity across compaction(the "what/why/current state"),not a step-by-step task list.
-keep them consistent:when the plan or state chages,update the ledger at the intent/progress level (not every micro-step).

### in replies
-begin with a brief "ledger snapshot" (goal + now/next + open questions).Print the full ledger only when it materially changes or when the user asks.

### `CONTINUITY.md` format(keep headings)
-goal (incl.success criteria):
-constraints/assumptions:
-key decisions:
-state:
 -done
 -now
 -next
-open questions: (UNCONFIRMED if needed):
-working set (files/ids/commands):

You are running PLAN — the last MANUAL step before the autonomous loop. Input:
`SCOPE.md` (the WHAT) + `AGENTS.md`. Output: `PLAN.md` — the HOW.

**Language:** talk to Gabriel in the chat in Brazilian Portuguese (pt-BR); write
`PLAN.md` in English.

This is where Gabriel's technical judgement goes, and it is deliberately NOT a task
list. SCOPE says what v1 must do; PLAN says how it will be built. The autonomous loop
will later cut PLAN into atomic tasks — your job here is to get the *approach* right
and agreed, so the build rests on a design a human understood and approved (this is
the comprehension-debt control, moved upstream of the diff).

Think it through WITH Gabriel — propose, and let him steer. Then write `PLAN.md`:
- **Approach / architecture** — the shape of the solution: the main components, how
  data flows between them, the key modules and their responsibilities.
- **Key technical decisions + why** — stack specifics, storage, the trade-offs that
  matter, and the rationale (so the reasoning survives past this session).
- **Data model / interfaces** — the important types, schemas, and boundaries.
- **Risks / tricky bits** — what needs care, what could go wrong, what to get right.

Do NOT write tasks, checkboxes, or a step-by-step build order — that is the cut's job.
Keep it to the design decisions a reviewer must agree with before code exists.

Stop after `PLAN.md` and let Gabriel confirm it. Once approved, the autonomous part is
`forge loop .` — which cuts PLAN into tasks, gets a second opinion on the cut, and builds.

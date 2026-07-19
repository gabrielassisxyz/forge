You are running the TASK CUT — the first autonomous step. Input: `PLAN.md` (the approved architecture / the HOW) + `SCOPE.md` (the v1 must-haves / the WHAT) + `AGENTS.md`. Output: the planning-with-files 3-file spine the loop will execute.

`PLAN.md` already decided the approach — do NOT re-litigate architecture here. Your job is to atomize that approved design into tasks. If `PLAN.md` is absent (older project), fall back to deriving the approach from `SCOPE.md`.

**Language:** reply to the user in whatever language they write to you; write all file content (task_plan.md / findings.md / progress.md) in English.

Turn the v1 must-haves into ATOMIC tasks. Err on the side of MORE, SMALLER tasks — a too-big task overruns the agent's per-call time budget and stalls; a too-small task costs only one cheap extra iteration. An atomic task is:
- ONE logical change, implementable and checkable in a single short agent turn;
- small enough that its diff is reviewable on its own;
- ordered so dependencies come first (scaffolding → core → wiring → polish).

**Split anything that bundles concerns.** These are each their OWN task, never one:
- adding a dependency (`go get` / `npm i` + wiring) is separate from using it;
- a `done when:` with multiple clauses ("does X AND Y") → one task per clause;
- a change spanning multiple files/layers (endpoint + client + UI) → one per layer;
- frontend especially: one task per component/interaction, kept small (rendering + driving a browser to check it eats into the same time budget). If you can't state a task's outcome in one line, it's two tasks.

Write three files:

**task_plan.md** — the executable checklist. Group tasks under phase headings. Every task is a `[ ]` line stating a concrete outcome AND its acceptance check:
```
## Phase 1 — <name>
- [ ] <what to build> — done when: <the observable check, e.g. a passing test / a command's output>
```
The `done when:` clause is not optional — it is what the verifier and the agent both check against. Vague tasks produce vague code.

**findings.md** — start with the key decisions from IDEA.md/SCOPE.md (stack, storage, anti-goal) so every loop iteration inherits them. Iterations append to it.

**progress.md** — empty log; the loop appends one line per completed task.

Front-load thinking here: planning tokens are cheap, implementation tokens are not. A plan checked twice implements once. The loop runs against this plan unattended and Gabriel does NOT review task_plan.md (he gated SCOPE.md already) — so the plan's quality is the run's quality. A vague or oversized task here becomes a stalled or escalated iteration there. Stop after the three files.

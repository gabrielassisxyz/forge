You are running PLAN BREAKDOWN — step 4 of forge. Input: `SCOPE.md` + `AGENTS.md`.
Output: the planning-with-files 3-file spine that the loop will execute.

Turn the v1 must-haves into ATOMIC tasks. An atomic task is:
- one logical change, independently implementable and testable in a single agent turn;
- small enough that its diff is reviewable on its own;
- ordered so dependencies come first (scaffolding → core → wiring → polish).

Write three files:

**task_plan.md** — the executable checklist. Group tasks under phase headings.
Every task is a `[ ]` line stating a concrete outcome AND its acceptance check:
```
## Phase 1 — <name>
- [ ] <what to build> — done when: <the observable check, e.g. a passing test / a command's output>
```
The `done when:` clause is not optional — it is what the verifier and the agent
both check against. Vague tasks produce vague code.

**findings.md** — start with the key decisions from IDEA.md/SCOPE.md (stack, storage,
anti-goal) so every loop iteration inherits them. Iterations append to it.

**progress.md** — empty log; the loop appends one line per completed task.

Front-load thinking here: planning tokens are cheap, implementation tokens are not.
A plan checked twice implements once. Stop after the three files; the human reviews
task_plan.md before `forge-loop` runs.

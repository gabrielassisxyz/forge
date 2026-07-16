# Architecture

forge is a thin orchestration over tools that already exist. It builds almost
nothing new: the two genuinely new pieces are **the loop with a verifier** (step 5)
and **the graduation gate** (step 7). Everything else composes skills you already
run — `project-bootstrap`, `planning-with-files`, `code-review`.

## The layered model

Three techniques stack; they are not alternatives (a common confusion):

- **Ralph loop** = the inner engine. One process, one repo, one task per iteration,
  fresh context each turn, state on disk. Great on greenfield (~90%); the last 10%
  is slop unless something pushes back.
- **forge orchestration** = the steps that feed the engine (interview → bootstrap →
  scope → plan) and gate its output (verifier, graduation). This is where the missing
  10% is recovered.
- **Loop-engineering discipline** = the safety rubric borrowed on top: maker≠checker,
  attempt caps, structured state, escalation. We steal its *checklists*, not its
  *patterns* (its patterns are maintenance loops — wrong shape for greenfield tools).

## Why the harness, not the prompt, makes it usable

The old `idea-to-planning` produced prototypes because its loop was ~50 words of
prompt with hardcoded constraints and **no AGENTS.md, no CI gate, no verifier**. The
Ralph "minimal prompt" is about the *iteration instruction* — Ralph itself demands
rich anchor files (`AGENTS.md`, the plan). Quality lives in four backpressure layers:

| Layer | Catches | Where |
|---|---|---|
| AGENTS.md (in the built project) | wrong conventions, wrong shape | written by `project-bootstrap`, read every iteration |
| `bin/ci` (deterministic) | broken build, failing/absent tests, secrets | run after each task in `forge-loop` |
| verifier agent (semantic) | placeholders, stubs, task-not-actually-done | separate invocation, default REJECT |
| retry cap | infinite fix loops, token burn | `FORGE_MAX_RETRIES` → escalate |

The verifier is the load-bearing addition. loop-engineering's #1 anti-pattern is
"same agent implements and verifies" — confirmation bias rubber-stamps weak work. So
the checker is a *fresh* invocation reading the diff, told to find reasons to reject.

## The loop (step 5) in detail

`bin/forge-loop`, per iteration:

1. Count remaining `[ ]` in `task_plan.md`; zero → `##DONE##`, stop.
2. Implementer agent does the first `[ ]` task (reads AGENTS.md + plan itself; files
   are not re-embedded — keeps the stack small, per Ralph's context discipline).
3. `bin/ci` if present. Fail → feed the tail back to the implementer, retry.
4. Verifier agent → `##ACCEPT##` or `##REJECT##` + defects. Reject → feed back, retry.
5. Accept → `git commit` the task (each task is one commit = the review surface).
6. Guard: if no checkbox advanced, escalate rather than spin.
7. Cap hit (`FORGE_MAX_RETRIES` per task, `FORGE_MAX_LOOPS` global) → write `STUCK.md`,
   exit non-zero. A human picks up with full context in the run-state dir
   (`~/.local/state/forge/<project>/`, override `FORGE_STATE_HOME`) + `progress.md`.

**Run state lives OUTSIDE the project tree** — the implementer runs `git clean -fdx`
/ `go clean` to reset build artifacts, which deletes an in-repo state dir mid-run and
takes the loop's own log with it (it did, on the first real run). Keeping state in
`~/.local/state/forge/<project>/` also stops it polluting the tool's git history.

State spine = planning-with-files' three files: `task_plan.md` (plan+checkboxes),
`findings.md` (durable decisions/discoveries), `progress.md` (trajectory log).

## Decisions

1. **New repo, not a fork of idea-to-planning.** The old skeleton's skills were
   placeholders (its own `processor.py` says so) and it was welded to OpenCode+ai-jail.
   Salvaged: the pipeline shape and the four prompt-specs. Rewritten: everything else.
2. **Verifier is mandatory and separate.** Without it, greenfield Ralph ships stubs.
   This is the one thing that makes output "usable" vs "prototype".
3. **Agent-agnostic via `FORGE_AGENT_CMD`.** No hard dependency on a specific CLI; the
   loop just needs a non-interactive agent with filesystem + tool access.
4. **planning-with-files over planning-workflow.** The latter is a heavy human-in-the-
   loop, multi-model-blending methodology — the opposite of autonomous. We keep its
   *philosophy* (plan hard, planning tokens are cheap) but use the lighter 3-file spine.
5. **beads deferred.** `task_plan.md` checkboxes are enough for v1-sized plans (10-20
   tasks). Adopt `beads-workflow` only if dependency ordering becomes a real pain —
   not before (YAGNI).
6. **Graduation gate never auto-approves.** It assembles the review surface (plan
   completion, CI, commits, a checklist) and hands it to Gabriel. The "is it usable /
   do I understand it by reading" judgement stays human — that is the comprehension-debt
   control, and it is the same rule as reading a PR.
7. **Human confirm between planning steps 1-4.** These are cheap tokens and decide
   quality; the human gates each artifact (IDEA.md, SCOPE.md, task_plan.md) before the
   unattended loop spends real implementation tokens against it.

## Open questions (to settle with real runs)

- Sentinel robustness: parsing `##ACCEPT##`/`##REJECT##` from agent stdout is fragile
  across CLIs. May need a structured output file instead.
- Where forge builds: a `projects/` subdir vs a sibling repo per tool. Leaning sibling
  repo (each forged tool is its own thing with its own git history).
- Whether steps 1-4 should also be scriptable end-to-end or stay human-gated prompts.
  Current bet: keep them human-gated until a real run proves the confirms are noise.
- **Skill routing for the autonomous implementer.** The loop's implementer currently
  has whatever skills its agent auto-discovers — nothing routes the *right* skill to a
  task. This matters most for **frontend work**, where a UI skill (design system, layout,
  component patterns) is the difference between usable and ugly. Open: does the plan-
  breakdown step tag tasks with the skill they need, and `forge-loop` pass `--skill` to
  the implementer? Pairs with the default visual-identity idea parked in llm-workflow
  IDEAS.md — a forged frontend should inherit Gabriel's taste, not reinvent it each run.

## First real run — what broke (hardware-usage, 2026-07-16)

Three bugs the first live run exposed, all now fixed. Kept here so they aren't relearned.

1. **No per-call timeout → a slow/stalled agent invocation blocked forever.** A Kimi
   call sat with the connection open, zero output, ~30 min. `run_agent` now wraps the
   agent in `timeout -k 10 $FORGE_AGENT_TIMEOUT` (600s); a timed-out call is a failed
   attempt and the retry continues from partial on-disk work.
2. **Run state lived inside the repo the agent edits → the agent deleted it.** The
   implementer ran `git clean -fdx` to reset Go build artifacts, wiping the in-repo
   `.forge/` (the loop's own log/state) and crashing the run. State moved to
   `~/.local/state/forge/<project>/`; `log()` is now non-fatal too.
3. **A fixed timeout fights genuinely heavy tasks.** The docker-collector task bundled
   *add dependency + interface + implement + test* — many slow round-trips plus a large
   `go mod tidy` — and overran 600s repeatedly. The durable fix is **smaller atomic
   tasks at plan-breakdown**, not a bigger timeout (a bigger timeout just lets a real
   hang waste more). Open follow-up: teach plan-breakdown to keep a task inside one
   agent-call budget.
4. **The implementer proxied the acceptance check instead of performing it.** It coded
   the process-monitor UI, made its Node string-tests pass, and called it done — shipping
   a sort control that reverts every SSE tick and confirm buttons a descendant CSS
   selector left unclickable. The tempting read was "it can't see the page"; that's
   **wrong** — it had the same shell the reviewer used and could have run headless
   Chromium. The reviewer (Opus) caught everything precisely because it *did* render and
   assert on the computed `display` — same tools, different behavior. (A sibling defect
   was backend, not frontend: a ~45s freeze from sequential Docker stat fetches the unit
   tests never timed.) **Root cause: an agent takes the cheapest path that turns a check
   green, and a string test is cheaper than rendering/measuring.** Fix was a prompt rule
   (`implement.md` rule 5): PERFORM the task's `done when:` literally — render/drive/
   measure — never substitute a proxy. Capability was never the lever; instruction was.
   The skill-routing follow-up (a UI skill + a browser-driving verify step) still helps,
   but the acute bug was prompting, not tooling.
5. **Two loop-integrity bugs (both fixed).** (a) A timed-out attempt fell through to
   ci/verify; the partial work compiled and the verifier ACCEPTED incomplete code, then
   the checkbox guard escalated — a timeout now short-circuits straight to retry. (b) The
   implementer marks a task `[x]` optimistically; when the verifier then rejected it, the
   mark stuck and the plan claimed a broken task was done — `escalate()` now reverts
   `task_plan.md` to the last committed state. **The deeper cause:** a `[x]` checkbox is
   trusted state that only the verifier should be allowed to set. This is the strongest
   argument yet for revisiting the **beads deferral** (decision #5) — a real tracker
   closes a task explicitly, which could be bound to verifier ACCEPT instead of the
   implementer's self-report.

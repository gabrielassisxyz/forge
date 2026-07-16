# forge

Turn a raw idea into a **usable v1 tool**, mostly autonomously — interview →
plan → autonomous implementation loop with a verifier → a review gate you read.

Successor to the (never-finished) `idea-to-planning` experiment. The difference
that matters: forge targets **"make it work" usable quality**, not a raw prototype.
The autonomous loop does not run blind — it runs inside a harness (AGENTS.md +
`bin/ci` + a separate verifier agent) that is what turns "compiles" into "works".
Then you refactor toward "right" — you never rebuild from scratch.

## The pipeline

```
idea (vague, one line)
 MANUAL — one guided session via the /forge-new skill (you gate each doc):
   1 interview        → IDEA.md    WHAT + anti-goal + done-check
   2 project-bootstrap→ AGENTS.md, bin/ci, gitleaks   (real guardrails)
   3 scope            → SCOPE.md   cut v1 HARD (the WHAT)
   4 plan             → PLAN.md    architecture / the HOW — your technical gate
 AUTONOMOUS — `forge loop .`:
   5 task cut         → task_plan.md + findings.md + progress.md   (atomizes PLAN)
   6 review the cut   → an independent model (codex) improves the cut in place
   7 dev loop         → per task: implement → bin/ci → verify → mark → commit
 THEN:
   8 graduate         → GRADUATION.md   the human gate: usable? understandable by reading?
```

You gate the **WHAT** (SCOPE) and the **HOW** (PLAN); everything after runs unattended.
The task cut is derived from PLAN and second-opinion-reviewed inside the loop — you
don't review it by hand. Run the whole manual half with **`/forge-new`**.

## Why not just a plain Ralph loop?

A raw Ralph loop on a greenfield repo gets ~90% of the way — and the last 10% is
placeholders, stubs, and "compiles but doesn't work". forge closes that with **four
backpressure layers**, because the prompt is not what makes code good:

1. **AGENTS.md** in the project — the direction the implementer must follow.
2. **`bin/ci`** — deterministic gate (tests/lint/build) after every task.
3. **A verifier agent** — a *separate* invocation, default stance REJECT, that reads
   the diff and asks "did it actually do the task? any stubs?". The implementer never
   grades its own homework.
4. **A retry cap** — bounded fix loop per task, then escalate to you with context.

## Usage

```bash
export PATH="$PWD/bin:$PATH"          # or symlink bin/forge somewhere on PATH

forge new ~/repositories/my-tool      # scaffold dir + 3-file spine + git init
cd ~/repositories/my-tool

forge interview                        # prints the interview prompt → run it → IDEA.md
# then run the project-bootstrap skill here → AGENTS.md + bin/ci
forge scope                            # → SCOPE.md   ← the last thing you gate by hand

forge loop .                           # autonomous: cut → review the cut → dev loop
forge graduate .                       # → GRADUATION.md — read it, run the tool, decide
```

### Configuring the loop (agent-agnostic)

`forge-loop` shells out to whatever coding agent you point it at:

Defaults are baked in for Gabriel's stack — implement with Kimi, verify with Opus —
so a normal run needs no env at all. Override only to change the setup.

| env | default | meaning |
|---|---|---|
| `FORGE_AGENT_CMD` | `pi -p -a --model litellm/kimi-k2.7` | **implementer** — cheaper/faster model, prompt passed as the last arg, CWD = project. Code-tuned variant: `--model openrouter/moonshotai/kimi-k2.7-code`. |
| `FORGE_VERIFY_CMD` | `claude -p --model opus --dangerously-skip-permissions` | **verifier** — the *stronger* model grades the implementer (maker/checker split). Pin with `--model claude-opus-4-8` for reproducible reviews. |
| `FORGE_PLAN_REVIEW_CMD` | `codex exec --dangerously-bypass-approvals-and-sandbox` | **second opinion on the task cut** (not per dev task) — an independent model reviews `task_plan.md` against SCOPE and improves it in place before the dev loop. Set empty to disable. |
| `FORGE_PLAN_CMD` | *(= `FORGE_VERIFY_CMD`)* | model that derives `task_plan.md` from `SCOPE.md` when no plan exists (planning is cheap + high-leverage → the strong model). |
| `FORGE_SANDBOX` | *(empty)* | wrapper prefixed to every agent call, e.g. `ai-jail`. **Strongly recommended** for unattended runs. |
| `FORGE_MAX_LOOPS` | `40` | global iteration ceiling |
| `FORGE_HEARTBEAT` | `60` | seconds between liveness lines logged during a running agent call |
| `FORGE_AGENT_TIMEOUT` | `600` | per-call wall-clock ceiling; a timed-out call becomes a failed attempt |
| `FORGE_MAX_RETRIES` | `3` | verifier rejections per task before escalating |

## Status

v0 — the loop, verifier, prompts, templates, and graduation gate exist and the
scripts are shellcheck-clean. Not yet run end-to-end on a real project; that is the
next step (there are several candidate tools to forge and iterate on). Expect the
prompts and the sentinel contract to move as real runs teach us what breaks.

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the design and the decisions.

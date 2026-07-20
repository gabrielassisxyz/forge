# Roadmap

What exists, what is missing, and what is deliberately out of scope. Details and the reasoning behind each decision live in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## What exists today

- **The full pipeline, end-to-end**: a manual half (interview → bootstrap → scope → plan, gated by a human at SCOPE and PLAN) and an autonomous half (`forge loop`: task cut → independent review of the cut → implement → `bin/ci` → verify → commit, per task).
- **Four backpressure layers** that turn "compiles" into "works": a per-project AGENTS.md, a deterministic `bin/ci` gate, a separate verifier agent with a REJECT default, and a bounded retry cap that escalates with context.
- **Agent-agnostic loop**: implementer, verifier, planner, and plan-reviewer are all pluggable via `FORGE_*` env vars; a sandbox wrapper (`FORGE_SANDBOX`) can prefix every call.
- **Hardening from real runs**: per-call timeouts, run state kept outside the built repo, loop-owned checkboxes (the implementer never marks its own task done), and escalation that reverts optimistic plan state.
- **A human graduation gate** (`forge graduate`) that assembles the review surface but never auto-approves.

## What is missing / natural next steps

- **UI quality of forged frontends** — the known open gap: the atomized dev loop builds functional-but-bland frontends. The candidate fix is skill routing: tag tasks at plan-breakdown with the skill they need and pass it to the implementer.
- **Sentinel robustness**: parsing `##ACCEPT##`/`##REJECT##` from agent stdout is fragile across CLIs; a structured output file may replace it.
- **Task sizing at plan-breakdown**: teach the cut to keep every task inside one agent-call budget, so heavy tasks stop fighting the fixed timeout.
- **A real task tracker (beads) if checkbox integrity frays again** — deferred on purpose; adopt only when dependency ordering or trusted-state bugs make checkboxes a real pain.
- **Where forge builds**: a `projects/` subdir vs a sibling repo per tool — leaning sibling repo, unsettled.
- **Whether the manual steps stay human-gated** or become scriptable end-to-end — current bet: keep the gates until real runs prove the confirmations are noise.

## Deliberately out of scope

- No web UI, no multi-user, no plugin system, no N-backend abstraction. The scope is "one person forges personal tools from the CLI"; anything drifting past that gets flagged, not built.
- forge is the factory, not the tools it builds — features that belong to a forged project (its stack, its deploy, its design system) do not belong here.
- No framework-ification: the system stays a few shell scripts + prompt files + templates.

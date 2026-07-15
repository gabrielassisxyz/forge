# AGENTS.md — forge

**What this repo is:** a thin orchestrator that turns a raw idea into a usable v1
tool via an autonomous implement→verify loop. It is NOT the tools it builds — it is
the factory. Read [README.md](README.md) and [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

> **Do not confuse the two AGENTS.md files.** *This* one governs work on forge
> itself. The one forge writes into each built project (via `project-bootstrap`)
> governs the agent building *that* tool — that is the direction layer that keeps
> forged code out of prototype territory.

## Current scope

Turn an idea into a **usable v1** (make-it-work quality), mostly autonomously, with a
verifier keeping the loop honest. Don't expand beyond it without a present need — no
web UI, no multi-user, no plugin system, no N-backend abstraction. If a change drifts
past "one person forges personal tools from the CLI", STOP and flag it.

## How to work here

- **Bash-first, KISS.** The whole system is a few shell scripts + prompt files +
  templates. Resist turning it into a framework — the old idea-to-planning died of
  over-engineering (a Python processor full of placeholders). Composition over code.
- **The prompts are the product.** `prompts/*.md` are load-bearing. Change them
  surgically; a word shift changes agent behavior. Keep them minimal (Ralph/RepoMirror:
  short prompts beat long ones) but never strip the anti-slop rules from `implement.md`
  or the REJECT-default from `verify.md`.
- **The verifier is sacred.** Never let the implementer verify its own work, never
  remove the retry cap, never default the verifier to ACCEPT. That split is the reason
  output is usable and not a stub farm.
- **shellcheck clean.** Run `shellcheck bin/*` before committing script changes.
- **Conventional Commits**, branch before non-trivial work, what+why in the message,
  no external attribution. English in files.

## Gotchas

- `FORGE_AGENT_CMD` must be non-interactive and permission-free (e.g. `claude -p
  --dangerously-skip-permissions`) or the loop hangs waiting for a prompt. Pair it with
  `FORGE_SANDBOX` for anything unattended — an agent that can `rm -rf $HOME` will
  eventually try.
- Each forged tool is its own git repo; per-task commits ARE the human review surface.
  Don't squash them away before graduation.

---
name: forge-new
description: >-
  Run the MANUAL front-end of a forge project in one guided session — the roteiro
  before `forge loop`. Walks interview → project-bootstrap → scope → plan, producing
  IDEA.md, AGENTS.md/bin/ci, SCOPE.md and PLAN.md, pausing for Gabriel's confirmation
  at each gate, and stopping before the autonomous loop. USE-WHEN: Gabriel says
  "/forge-new", "forge new project", "start a new tool with forge", or wants to spin up
  a new tool from an idea. NOT-FOR: the autonomous build (that is `forge loop .`), or
  editing an existing forge project's plan.
---

# forge-new — the guided manual front-end

Take a raw idea to a **human-approved PLAN**, ready for `forge loop`. This is the manual
half of forge (see `~/repositories/forge/README.md`): you conduct four steps in ONE
session, each producing a doc, and you **gate each one with Gabriel** before moving on.
The autonomous half (`forge loop .`) is NOT part of this skill — you stop before it.

The step prompts are the source of truth and live in the forge repo at
`~/repositories/forge/prompts/`. For each step, **read that prompt and follow it**, then
pause. Talk to Gabriel in **pt-BR**; write all doc files in **English**.

## Before you start

Ask Gabriel for the tool name and scaffold the sibling repo:

```
forge new ~/repositories/<name>      # creates the repo + 3-file spine + git init
cd ~/repositories/<name>             # CWD = the new tool for the whole session
```

Everything below happens with the CWD as the new tool's repo.

## The four steps — each ends at a checkpoint (⏸ = wait for Gabriel's OK)

1. **interview** → `IDEA.md`  ·  prompt: `prompts/interview.md`
   Extract a sharp WHAT + anti-goal + done-check from the raw idea. ⏸ Confirm IDEA.md.

2. **project-bootstrap** → `AGENTS.md`, `bin/ci`, gitleaks
   Invoke the **`project-bootstrap`** skill on this repo (it installs the guardrails the
   loop depends on). It runs greenfield here, so answer its questions from IDEA.md:
   personal tool, tier 1–2, no deploy, solo. ⏸ Confirm the AGENTS.md it wrote.

3. **scope** → `SCOPE.md`  ·  prompt: `prompts/vision-scope.md`
   Cut v1 HARD — must-haves only, everything else explicitly out of scope. ⏸ Confirm SCOPE.md.

4. **plan** → `PLAN.md`  ·  prompt: `prompts/plan.md`
   The HOW: architecture, key technical decisions + why, data model, risks. This is
   Gabriel's technical gate and it is NOT a task list. Think it through *with* him. ⏸ Confirm PLAN.md.

## Finish

Commit the docs, then hand off — do NOT start the loop:

```
git add -A && git commit -m "forge: interview + scope + plan (manual front-end)"
```

Tell Gabriel: *"Front-end pronto (IDEA/SCOPE/PLAN + guardrails). Quando quiser, rode
`forge loop .` — ele corta as tasks a partir do PLAN, pega uma segunda opinião no corte,
e constrói."*

## Rules

- **Never skip a checkpoint.** One session, but interactive — each doc is gated. Barrelling
  through to PLAN without confirmation defeats the point (the gates are where Gabriel's
  judgement enters and comprehension debt is paid).
- **Don't write tasks in PLAN.md** — atomic task-cutting belongs to `forge loop`, not here.
- **Files in English, chat in pt-BR.**

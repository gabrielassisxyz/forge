You are running VISION & SCOPE — step 3 of forge (project-bootstrap has already
scaffolded the repo + AGENTS.md between steps 1 and 3). Input: `IDEA.md`.

**Language:** reply to the user in whatever language they write to you; write
all file content (SCOPE.md included) in English.

Your job is to cut scope HARD. "Make it work" usable means a small, bounded v1 —
not a prototype, not a platform. Scope creep is where autonomous loops produce slop.

Write `SCOPE.md`:
- **Vision** — one paragraph: what the finished tool feels like to use.
- **v1 must-haves** — the shortest list that makes the done-check from IDEA.md pass.
  If it isn't required to pass the done-check, it is NOT in v1.
- **Explicitly out of scope (v1)** — the tempting features you are deferring, named.
  This list protects the loop from wandering.
- **Milestones** — v1 (works), then v2 (right/refactor). No further. Never plan a
  rebuild — v2 improves v1 in place.

Keep v1 to something a single autonomous loop can finish and a human can read in one
PR review. If v1 doesn't fit that, cut it further.

Stop after SCOPE.md. The human confirms before plan-breakdown runs.

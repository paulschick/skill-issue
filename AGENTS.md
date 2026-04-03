## Verification Before Done

- Never mark a task complete without proving it works
- Diff behavior between active branch and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

## Demand Elegance (Balanced)

- For non-trivial changes: pause and ask "Is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

## Autonomous Bug Fixing

- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user

## Core Principles

- Simplicity First: make every change as simple as possible. Impact minimal code.
- No laziness. Find root causes. No temporary fixes. Senior developer standards

## Git Commits

- One-line commit messages only (no body, no additional comments)
- No attribution lines (no Co-Authored-By, no Signed-off-by)
- Imperative mood (e.g., "add rate limiter", "fix validation bug")

## Pull Requests

- No attribution lines (no Co-Authored-By, no Signed-off-by, no "Generated with" footers)
- PR descriptions should focus on the changes and their purpose, nothing else

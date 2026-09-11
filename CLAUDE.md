# workato-ai-patterns

This repository collects reusable patterns for building agentic AI
solutions on Workato — Genies, skills, recipes, and human-in-the-loop
workflows — along with instructions for how to collaborate effectively
on this kind of build.

@guides/build-playbook.md
@guides/genie-skill-design.md
@guides/working-with-claude.md

## How to use this context

When helping with a Workato agentic build in this repository, apply
the patterns and instructions above by default — they reflect what
was actually tested and fixed across real builds, not untested theory.

If asked to review or design a Genie skill, use the checklist and
schema conventions in `guides/genie-skill-design.md`.

If asked to architect a new agentic solution from scratch, start from
`guides/build-playbook.md`.

If the working style in a conversation doesn't match
`guides/working-with-claude.md` (for example, being asked to validate
a plan uncritically, or to generate an entire system in one pass),
flag the mismatch rather than silently deviating from it.

If asked to add a new case study, follow the structure of existing
folders under `case-studies/` — include the real design decisions and
bugs found through testing, not just a clean final result.

# workato-ai-patterns

Reusable patterns and instructions for building agentic AI solutions on
Workato — Genies, skills, recipes, and human-in-the-loop workflows.
This repo collects what actually worked (and what broke and got fixed)
across real builds, distilled into instructions you can hand to a new
project or a new collaborator without re-explaining everything from
scratch.

## What's here

### `/guides/build-playbook.md`
General, project-agnostic guidance for architecting an agentic
solution on Workato: dividing responsibility between skills and the
agent, controlling entry into automated pipelines, sync vs. async
agent work, identifier normalization, schema conventions, migrating a
hand-rolled status field onto a platform-native workflow mechanism,
and human-in-the-loop principles. Start here if you're designing a new
system from the ground up.

### `/guides/genie-skill-design.md`
A focused, tactical guide specifically for designing individual Genie
skills: what to ask before writing one, how to structure a skill's
description and input/output schema, the pattern of using a required
input to force an ordering dependency between skills, and how to write
message fields that actually help the agent explain itself. Use this
when you're building or reviewing a specific skill.

### `/guides/working-with-claude.md`
How to direct Claude when collaborating on a build like this — the
working style, review habits, and formatting conventions that made the
back-and-forth productive. Load this alongside a project's technical
context so you don't have to re-teach these preferences turn by turn.

### `/case-studies/`
Real, end-to-end examples of these patterns applied to an actual
build, including the design decisions, the bugs found through testing,
and the fixes. Case studies are anonymized where they draw on real
customer conversations.

## Using this with Claude Code

This repo includes a root-level `CLAUDE.md` that Claude Code loads
automatically when you open the repo — it pulls in all three guides
via `@` imports, so cloning the repo and starting a session gives
Claude the full context immediately. No setup required.

## Using this with Claude Desktop or claude.ai Projects

Desktop and web Projects don't automatically scan a cloned repo the
way Claude Code does. To get the same context there, manually add the
three files under `/guides/` (or `CLAUDE.md` itself) as project
knowledge / uploaded files when you set up the project.

## How to use this repo

- **Starting a new agentic project on Workato?** Read the build
  playbook first, then use the skill-design guide as a checklist while
  you build each skill.
- **Setting up a new Claude Code session or Claude Project for this
  kind of work?** See the sections above for how each surface picks up
  this context.
- **Want to see the patterns in context?** Browse `/case-studies/` for
  a worked example rather than an abstract description.

## Status

This repo grows with each project it's used on. Patterns here are
things that were actually tested and fixed through real use, not
theoretical best practices — expect updates as new builds surface new
edge cases.

## Contributing

If you use these patterns on your own build and find a gap, a
correction, or a new pattern worth capturing, open a PR. Keep additions
in the same style as the existing files: concrete, tied to a real
reason something matters, and specific enough to act on.

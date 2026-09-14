# Case study: [Project name]

[One or two sentences: what kind of agentic solution this is, who it's
for, and what it's grounded in — a real requirements document, real
discovery conversations, an existing manual process. Say plainly if
any part of this case study is illustrative rather than a real build.]

This case study applies the patterns in [`/guides`](../guides) to a
concrete build. Read it alongside the guides, not instead of them —
this document should show *how* the patterns played out, including the
mistakes, not just the final architecture.

## The problem

[This section is the "what" that has to exist before any design work
starts — see `guides/build-playbook.md` section 0. Answer explicitly,
in plain language, before writing a single skill or recipe:]

- Who is affected, and what are they currently doing manually today?
- What's the actual scale — how many records, people, or transactions
  a year (or week, or day)? A vague "a lot" or "not much" isn't enough
  to size the build.
- What's broken or missing about the current process — no structured
  data, no shared criteria, too slow, error-prone, doesn't scale?
- Why does this matter now — a deadline, a cost, a compliance need, a
  growth constraint?

If any of this is unclear or unstated, ask directly rather than
assuming — this is exactly the gap `guides/build-playbook.md` section 0
exists to close.

## What was built

[List the entry points into the system — how does something new enter
this pipeline (self-service, conversational, automated discovery, bulk
import, etc.)? Then describe the shared pipeline every entry point
funnels into: research/analysis, scoring, human review, and whatever
final action completes the flow.]

### Core components

[List each real component and its job, one line each: the Genie, each
skill (name and single responsibility), any knowledge base, the data
tables, the recipes and what each one triggers on, any Workflow App
pages, any MCP server and what it exposes externally.]

## Key design decisions

[A handful of decisions worth calling out specifically, because they
came from real problems discovered during the build, not from upfront
planning. For each one: what was decided, and what problem it actually
solved. If nothing like this has happened yet, leave this section
explicitly marked as not yet populated rather than inventing decisions
that didn't really happen.]

## What testing actually caught

[A running smoke-test list, re-run after every fix — see
`guides/build-playbook.md` section 12. Name real bugs found as concrete
examples, not abstractions: what broke, why, and what the fix actually
was. This section is what makes a case study worth reading instead of
just the final diagram — don't skip it or thin it out to look tidy.]

## Resources

[Links to diagrams, recordings, or other artifacts. If a diagram is
presentation-oriented rather than a maintained spec, say so explicitly
and point to where the actual ground truth lives instead (the live
workspace, the skill/recipe definitions).]

## Status

[Where this actually stands right now — in design, built and being
tested, demoed, in production — stated plainly, not aspirationally.]

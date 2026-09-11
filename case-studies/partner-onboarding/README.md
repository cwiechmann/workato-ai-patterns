# Case study: AI-based partner onboarding

An agentic solution built on Workato for an experience-platform
business: discovering, researching, scoring, and onboarding new
partners, with a human approval step before anything reaches
Salesforce. Built from a real requirements document and grounded in
real customer discovery conversations about the current, manual
process.

This case study applies the patterns in [`/guides`](../../guides) to a
concrete build. Read it alongside the guides, not instead of them —
this document shows *how* the patterns played out, including the
mistakes, not just the final architecture.

## The problem

The business onboards new experience providers (tour operators,
activity businesses) as sales partners, at meaningful volume: roughly
500 in-depth analyses, 2,000 prioritized candidates, and 1,000
self-onboarded experiences a year. The process was entirely manual:
partner data copy-pasted across disconnected systems, no lead scoring,
no structured self-service onboarding, and heavy reliance on someone
personally researching each provider's website and reputation before
deciding whether to pursue them.

## What was built

Three entry points feed one shared pipeline:

- **Self-service** — a partner submits themselves through a Workflow
  App form.
- **Conversational / agent-driven** — a person (support agent, BDR, or
  anyone else) tells the Genie about a partner directly.
- **Automated discovery** — the Genie searches the web for candidates
  matching criteria (region, experience type) and a human selects which
  ones to pursue.

From there, every partner goes through the same flow: research (its
website, public reviews, bookable products), scoring against a shared,
externally stored criteria catalog, human review and approval in a
Workflow App, and — only after approval — a Salesforce lead is
created.

### Core components

- **A Genie** ("Partner Genie") that does the research, reasoning, and
  scoring, instructed with a small set of principles rather than a
  fixed script.
- **Eight skills**, each with a single, specific job — from registering
  a partner and checking its status, to researching a website, to
  saving scored products. Every skill's own description states when
  and when not to use it; the Genie's instructions don't repeat that.
- **A knowledge base** holding the scoring criteria catalog, consulted
  fresh for every partner rather than baked into the Genie's own
  instructions — so the rubric can be edited without touching the
  agent.
- **Two data tables** (`Partner onboarding`, `Partner products`),
  migrated partway through the build from a hand-rolled `Status` column
  onto Workato's native Workflow App stages.
- **Three recipes** handling the deterministic parts of the pipeline:
  kicking off analysis, routing to human approval, and creating the
  Salesforce lead — each triggered by a stage change, deliberately kept
  as separate recipes so a Salesforce failure can't corrupt the
  approval step.
- **A Workflow App** with a self-service submission page and a review
  page showing the Genie's score, reasoning, and flagged manual-check
  items to a human reviewer.
- **An MCP server** exposing a subset of the skills to other systems,
  with its own instructions on which tools are safe to call directly
  and which require research context only the Genie can provide.

## Key design decisions

A few decisions are worth calling out specifically, because they came
from real problems discovered during the build, not from upfront
planning:

- **Controlling pipeline entry explicitly.** A boolean
  (`enter_pipeline`) plus an optional target-state override, set
  deliberately by whichever caller registers a partner, prevents any
  entry point — especially exploratory or automated ones — from
  silently triggering expensive analysis.
- **Splitting synchronous and asynchronous analysis.** One or two
  explicitly confirmed partners are analyzed live, in conversation;
  three or more are proposed as background work. Confirmation is
  required either way — volume changes *how* the work happens, not
  *whether* it needs agreement first.
- **Migrating fully onto Workflow App stages.** A plain `Status` column
  was replaced entirely by Workato's native stage mechanism partway
  through the build, once it became clear running both would just
  reintroduce the duplication being solved. Every skill and recipe that
  wrote or triggered on the old field was updated, not left running in
  parallel.
- **A required input to force an ordering dependency.** The three
  research skills each require an `already_analyzed` flag that can only
  come from calling `Get partner status` first — added after testing
  showed the Genie would otherwise parallelize a status check with the
  very research it was meant to gate.
- **Separating "analyze" from "commit."** The Genie presents a finished
  analysis as a proposal and only saves it — moving the partner into
  human review — once the user explicitly confirms. This was a direct
  fix after early testing showed analysis being saved automatically the
  moment reasoning finished, with no chance for the user to react to it
  first.
- **Rewriting the Genie's instructions from a script to principles.**
  The original instruction set was a numbered procedure that
  accumulated one rule per bug found. It was deliberately rewritten
  into a small set of durable principles once that pattern became
  visible — see `/guides/genie-skill-design.md` and
  `/guides/working-with-claude.md` for the reasoning.

## What testing actually caught

A running smoke-test list, re-run after every fix, surfaced several
real bugs worth naming as examples rather than abstractions:

- The Genie fabricating a plausible-but-wrong URL for a partner it
  hadn't actually looked up.
- A status/stage check running in parallel with the research it was
  meant to gate, because nothing structurally required it to happen
  first.
- Field-name mismatches between what one step produced and what the
  next step read (`name` vs. `name1`, `status` vs. `stage`) — the
  single most common bug class across the whole build.
- A skill receiving a JSON array as a stringified value instead of a
  native array, because the schema didn't strongly enforce the type.
- An assumed duplicate-delivery risk on Workato triggers that turned
  out not to exist — Workato guarantees exactly-once trigger
  processing, confirmed by checking documentation rather than building
  unnecessary defensive logic.

## Resources

- `diagrams/` — presentation-oriented visuals (process flow, feature
  mapping) used for the demo and video series. These are illustrative
  snapshots, not a maintained spec, and may not reflect every later
  change to the actual build — refer to the skill and recipe
  definitions in the Workato workspace for ground truth.
- See `/guides` at the repo root for the general, reusable patterns
  this build follows.

## Status

Demoed live; used as the basis for a customer- and partner-facing video
series on building agentic solutions on Workato.

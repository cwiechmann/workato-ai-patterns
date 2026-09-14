# Case study: AI-based partner onboarding

A proposed agentic solution on Workato for an experience-platform
business: discovering, researching, scoring, and onboarding new
partners, with a human approval step before anything reaches
Salesforce. Grounded in a real requirements document and real customer
discovery conversations about the current, manual process — but the
build itself has not started yet. This is an initial project, not a
completed one.

This case study applies the patterns in [`/guides`](../../guides) to a
concrete build. Read it alongside the guides, not instead of them — as
the build progresses, this document should come to show *how* the
patterns played out, including the mistakes, not just a clean final
architecture.

## The problem

The business onboards new experience providers (tour operators,
activity businesses) as sales partners, at meaningful volume: roughly
500 in-depth analyses, 2,000 prioritized candidates, and 1,000
self-onboarded experiences a year. The process today is entirely
manual: partner data copy-pasted across disconnected systems, no lead
scoring, no structured self-service onboarding, and heavy reliance on
someone personally researching each provider's website and reputation
before deciding whether to pursue them.

## What was built

Nothing yet — this project is still at the design/kickoff stage. An
initial architecture sketch exists (see `diagrams/`), covering two of
the three flows in the source requirements (see `requirements.md`);
the third — self-onboarding of individual experiences — is explicitly
deferred for now.

A partner is always identified by its website URL, and the agent's
first real research step is to read that website directly, rather
than reasoning from the name or URL alone:

1. **Single-partner analysis.** A URL is provided directly
   (conversational registration), the agent reads the partner's own
   website to summarize the business and identify its sellable
   products, researches public reviews, then scores the partner
   against a shared criteria catalog and produces a recommendation.
2. **Search & prioritization of candidates.** Given freely definable
   criteria (region, experience type, price segment, etc.), the agent
   discovers candidate partners automatically — each represented by
   its website URL — and a human selects which to pursue; selected
   candidates go through the same per-partner analysis as flow 1.

Both flows converge on the same shared pipeline: research, scoring,
human review, and — only after approval — a Salesforce lead. Treat
this as a starting proposal, not a committed design; it should be
expected to change once real skills, recipes, and testing are actually
underway.

## Key design decisions

Not yet populated. No build decisions have been made yet — this
section should be filled in as real decisions get made during the
build, each with the actual problem it solved, not written ahead of
time as if they'd already happened.

## What testing actually caught

Not yet populated. No testing has happened yet, since nothing has been
built.

## Resources

- `requirements.md` — the source requirements (three flows, volumes,
  a worked example, and the flow explicitly deferred out of scope).
  Treat this as ground truth ahead of the diagram, this README, or
  anyone's recollection of the requirements.
- `diagrams/` — an initial architecture sketch (a Workato canvas
  diagram) used to kick off this build. This is a starting point for
  discussion, not a maintained spec — once the build begins, refer to
  the actual skill and recipe definitions in the Workato workspace as
  ground truth instead.
- See `/guides` at the repo root for the general, reusable patterns
  this build should follow.

## Status

In design — not yet built. Intended as the basis for a customer- and
partner-facing video series demonstrating how to build an agentic
solution on Workato with Claude's help, from a standing start.

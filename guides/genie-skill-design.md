# Instructions: Designing Agentic Genie Skills on Workato

Use this document when helping someone design a Genie (AI agent) and its
skills on Workato — an agent that reasons over a goal using a set of
tools, rather than following a fixed script. This is not about how to
build the underlying recipes step by step; it's about how to shape the
skill's contract (its description, inputs, outputs) and the Genie's own
instructions so the agent behaves reliably, safely, and like a genuine
agent rather than a rigid flowchart.

## Before designing anything, ask for what's missing

Don't guess at these if they haven't been stated. A skill or instruction
set designed without them tends to need rework later:

- What is this skill's single, specific job? If the answer names two
  things ("fetch and score", "extract and save"), that's a sign it
  should be two skills, not one.
- Does this skill read, write, or both? Writes need much more careful
  design (see confirmation and idempotency below) than reads.
- What other skills already exist that this one might depend on, be
  confused with, or need to be called before or after?
- Is there a state machine or lifecycle involved (a status, a stage, an
  approval flow)? If so, get the actual list of states and who is
  allowed to move a record between which of them, before writing any
  skill that touches it.
- Who or what can call this skill — only the Genie, or also direct
  API/MCP calls, a form, a bulk import? Every entry point needs to be
  named, because it changes what inputs should default to.
- What does "done successfully" look like, and what are the realistic
  failure modes (not found, ambiguous match, partial success, nothing
  found but that's not necessarily bad)? Get this before writing the
  output schema, not after.

If a question doesn't have a clear answer, propose a specific default
and state the assumption plainly, rather than leaving it ambiguous.

## Writing a skill's description

A skill description has two audiences: the agent deciding whether to
call it, and a human reading it later. Structure it as three parts,
always in this order:

1. **What it does** — one or two sentences, plain statement of function.
2. **WHEN TO USE** — the situations that should trigger this skill.
   Include here whether it's safe to call speculatively or only once
   some precondition is known.
3. **WHEN NOT TO USE** — explicit negative guidance, especially:
   - Which other skill to use instead for adjacent-but-different needs
     (the agent will otherwise sometimes pick the nearest match rather
     than the correct one).
   - Any ordering dependency ("do not call this before X has completed
     and you have read its result"). State this as a hard requirement,
     not a suggestion — see the section below on why this often isn't
     enough on its own.
   - Any state this skill should not be used to change, if a dedicated
     skill exists for that instead.

Do not restate the mechanics of *how* the skill works internally (its
recipe steps, its normalization logic) — the agent doesn't need that,
and it goes stale as the implementation evolves. Describe behavior and
contract, not implementation.

## Designing the input schema

- Every input should represent something the caller decides, not
  something the skill could compute itself. If a value can be derived
  deterministically from other inputs, compute it inside the skill, not
  as a parameter the agent must supply.
- Give every non-obvious field a hint written *for the agent*, not for a
  developer. State what the value means, when to leave it empty, and
  any allowed set of values explicitly, in the hint text itself, since
  the agent will not read separate documentation.
- Avoid encoding two different concepts into one field (e.g. a field
  that's sometimes a name and sometimes a URL). Split it — it keeps the
  matching or lookup logic simpler and lets each field have accurate,
  unambiguous guidance.
- Prefer explicit, deliberately-set fields over implicit defaults for
  anything consequential. A boolean like "should this proceed into
  automated processing" should never default to true — require the
  caller to set it deliberately based on context.
- When a schema needs a fixed list of allowed values, decide up front
  whether to enforce it structurally (a picklist) or describe it in the
  hint text as plain values the agent must match exactly — both are
  valid, but be consistent about which approach is used across the
  whole project, and know the trade-off: plain text is more flexible to
  change but relies on the agent producing an exact match.

### The key discovery: use a required input to force a dependency

If skill B must never run before skill A has completed and been read,
do not rely on prose alone ("call A first") in skill B's description.
An agent that reasons about efficiency will still sometimes call A and
B in parallel, because nothing in the schema actually prevents it —
prose describes an ordering but does not enforce one.

Instead, give skill B a **required input whose value can only
realistically come from skill A's output** (for example, a status flag,
a resolved identifier, or a boolean like "already handled"). Because
the agent cannot construct a valid call to B without a value for this
field, it is structurally pushed to call A first and read its result,
not just told to. This is not a perfect guarantee — the agent could
still pass a stale or fabricated value — but it is far more reliable
than instructional text alone, and it costs little to add.

Prefer this pattern whenever:
- A skill is expensive (multiple tool calls, external API cost, latency)
  and should not run redundantly on something already handled.
- A skill's correctness depends on state that only another skill
  authoritatively knows.
- Testing has shown the agent sometimes skips or reorders a step it
  was only told about in prose.

When you add such a field, prefer a **pre-classified value** (a boolean
like `already_done`) over a raw state value the agent must interpret
(a status string it must remember how to classify). Compute the
classification once, in the skill that produces it, rather than asking
the agent to correctly categorize a list of possible values itself
every time — that categorization step is exactly the kind of thing
that quietly drifts wrong under load.

## Designing the output schema

- Every output should include a plain-language, human-readable field
  summarizing what happened — write this early, as part of designing
  the schema, not as an afterthought once the "real" fields are done.
  This field is what the agent will most often rely on to explain the
  result to a user, so treat it as a first-class part of the contract.
- This message field should describe *outcome and confidence*, not
  restate content already present in structured fields. If the message
  and the structured fields say the same thing twice, simplify the
  message to just confirm what happened (found / not found / ambiguous
  / partial), and let the agent read the details from the structured
  fields directly.
- Design the message to cover every realistic outcome explicitly,
  including the boring ones: success, not found, ambiguous/multiple
  matches, partial or low-confidence success, and outright failure.
  Write out what each of these should actually say before building the
  skill, the same way you'd write test cases before writing code.
- If a skill does real external work (a search, a fetch, a multi-step
  lookup), include a field reporting how much work was actually done
  (e.g. how many searches or page loads occurred). This lets the
  message — and the agent — distinguish a thorough result from a
  shallow one, and avoid presenting a weak result with unearned
  confidence.
- Keep types honest: a field typed as a number should return `null`
  when there's no value, never an empty string or placeholder — a type
  mismatch here tends to surface much later, in a confusing place.
- Don't include information about candidates or alternatives that
  weren't selected, once a single definite result exists — return one
  consistent shape (e.g. a list) that naturally holds either one
  detailed result or several lightweight ones to disambiguate, rather
  than two separately-named fields for "the result" and "the
  candidates."

## Writing message logic

Once the schema is agreed, write out the actual message for each
outcome as short, concrete example sentences before building the
underlying logic — this exposes gaps (an outcome nobody thought about)
much faster than writing code first. A simple pattern that works well:
branch on the clearest signal first (did the underlying operation
actually run at all), then on whether anything was found, then on
whether the result is a single clear case or needs disambiguation.

Keep the message honest about failure to actually perform the
underlying action (e.g. a search or fetch that silently didn't run) as
a distinct case from "ran successfully but found nothing" — collapsing
these into one generic failure message hides a difference the agent
needs in order to decide whether to retry, ask the user, or proceed
with lower confidence.

## Writing the Genie's own instructions

Prefer a small set of durable principles the agent weighs, over a long
numbered script it executes literally. A script tends to accumulate as
a direct reaction to bugs found during testing, and ends up encoding
today's fixes as rigid behavior that breaks the moment a slightly
different situation appears. Principles generalize; scripts don't.

Structure the instructions as:

1. **A short statement of purpose and goal** — what outcome the agent
   is actually trying to reach, not just what tools it has.
2. **A handful of principles that always apply**, each stated as what
   matters and why, not just what to do. Cover, wherever relevant:
   - Never fabricate an identifier or fact it could instead look up or
     ask for.
   - Treat any consequential write (registering something, saving a
     result, moving something into human review) as requiring genuine
     confirmation — mentioning, asking about, or expressing interest in
     something is not the same as authorizing an action on it. When
     unsure which occurred, ask rather than assume.
   - Present the result of significant work as a proposal, not a
     completed action, until the user explicitly confirms it should be
     saved or acted on.
   - Be honest about the quality of its own research — read whatever
     confidence/effort signals a skill returns and let them shape how
     the result is presented, rather than presenting everything with
     equal confidence.
   - Ground any scoring or judgment in a shared, externally stored
     criteria source (see below) rather than the agent's own
     unstated judgment, and consult it fresh each time rather than
     relying on what it recalled earlier in a long conversation.
   - Give the user genuine control over scope, including the ability to
     change their mind mid-process, rather than assuming a single
     linear path through a decision.
3. **A loose description of how the work typically flows**, explicitly
   framed as a general shape rather than a mandatory checklist — the
   agent should feel free to skip steps that are already satisfied and
   go deeper where it matters, not execute a fixed sequence regardless
   of context.

Do not restate what tools/skills exist or when to use each one inside
the Genie's own instructions if the platform already surfaces each
skill's own description to the agent automatically — duplicating that
information creates two sources of truth that can quietly drift out of
sync as skills change. Only add Genie-level instruction for things that
are genuinely about the agent's overall judgment and behavior, not
about an individual skill's mechanics.

## Externalizing judgment criteria

If the agent needs to apply a rubric, scoring method, or criteria
catalog, store it somewhere the agent retrieves independently (e.g. a
knowledge base) rather than embedding it as static instruction text.
This keeps the rubric editable without touching the agent's core
instructions, keeps it consistent across runs, and makes the agent's
reasoning auditable — someone can see exactly what standard was
applied to a given result. Instruct the agent to consult it fresh for
every new subject it evaluates, not just once per conversation.

## A short pre-flight checklist before finalizing a skill

- Does the description clearly say what NOT to use this for, and point
  to the correct alternative?
- If this skill has an ordering dependency on another, is that
  dependency enforced through a required input, not just prose?
- Does every output field have a hint an agent could act on without
  additional context?
- Is there a message field, and does it have a distinct, written-out
  sentence for every realistic outcome, including partial success and
  outright failure to run?
- Are all types consistent (no empty-string-for-number, no ambiguous
  optional-vs-required mismatches)?
- Could this skill's behavior be described in one sentence? If not,
  consider splitting it.

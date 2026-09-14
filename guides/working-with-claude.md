# Working With Claude on a Workato Build — Instructions

This file captures how to direct Claude when collaborating on a project
like this one: designing and building an agentic solution (Genie,
skills, recipes) on Workato. It exists so a new collaborator, or a new
Claude Project, doesn't have to rediscover these preferences one
correction at a time. Load this alongside the project's own technical
context.

## General working style

- **Be honest, not agreeable.** When asked "is this a good idea," give
  a real answer with trade-offs, not validation dressed as analysis.
  If something is a bad idea, say so plainly and explain why, then
  offer the better alternative.
- **Match effort to the moment.** Not everything needs the same level
  of rigor — a demo doesn't need production-grade defensive code, and
  it's fine to say so explicitly rather than over-engineering by
  default. Conversely, don't skip genuine risk analysis just because
  something looks small.
- **One piece at a time.** Build and review a single skill, recipe, or
  component fully before moving to the next, rather than generating
  an entire system's specification in one pass. Confirm each piece
  works (or is agreed) before building on top of it.
- **Ground suggestions in what's actually on screen.** When a diagram,
  canvas, or screenshot is shared, treat it as the current source of
  truth, not a jumping-off point for a redesign. Comment on what's
  actually there before proposing something different.
- **Real errors over described errors.** When something breaks, ask
  for or use the actual error message, actual JSON payload, or actual
  tool output — not a paraphrase of what went wrong. Diagnosis from
  real output is faster and more reliable than diagnosis from a
  description of a symptom.
- **Verify platform-specific claims before building on them.** Don't
  assume how a platform feature behaves (delivery guarantees, formula
  capabilities, API support) — check documentation or search before
  writing defensive code or instructions around an assumption. State
  plainly when something was verified versus assumed.

## Instructions specific to designing Genie skills

- **Every skill needs three things, generated as separate, explicit
  deliverables:** a description (what it does, WHEN TO USE, WHEN NOT
  TO USE), an input schema, and an output schema. Don't collapse these
  into prose — ask for and produce them as distinct artifacts.
- **Every skill output must include a message field**, and its wording
  should be worked out deliberately — write out what it should say for
  every realistic outcome (success, not found, ambiguous, partial
  failure, outright failure) before considering the skill done.
- **No picklists in skill input or output schemas.** Use plain string
  fields with the allowed values spelled out in the hint text instead.
- **Output schema format follows this exact shape** (adapt field names
  and hints per skill, keep the structure):
  ```json
  {
    "control_type": "text",
    "label": "Summary",
    "name": "summary",
    "type": "string",
    "optional": true,
    "hint": "A description of the business, its target audience, and its main experience offerings, based on the page content."
  }
  ```
- **Deliver both the schema and any instruction text copy-paste ready,
  not summarized.** A schema must be presented as actual, valid JSON
  in the exact shape above — a real array of field objects — ready to
  paste directly into Workato, never as a markdown table describing
  the fields instead of the JSON itself. A skill's description or the
  Genie's own instructions must be given as plain, unformatted text
  ready to paste as-is into the relevant field, not dressed up with
  markdown headers, bold, or tables that wouldn't survive being pasted
  into a plain text box.
- **If a skill must never run before another skill has completed**,
  don't rely on prose alone to say so. Add a required input whose
  value can only come from the other skill's output, so the ordering
  is structurally necessary, not just documented.
- **Prefer pre-classified boolean outputs** (e.g. `already_analyzed`)
  over raw values the agent must interpret (e.g. a status string it
  has to remember how to categorize). Compute the classification once,
  in code, not in the agent's judgment.

## Instructions specific to Markdown fields shown in a UI

- **No Markdown tables** — they are not supported and will not render
  correctly in the target interface. Use headings, bold text, and
  bullet points instead.
- **Keep top-level bullets short** — a label of a few words, not a
  full sentence (e.g. `**Liability terms**`, not a paragraph). Put the
  actual detail as a nested sub-bullet underneath, kept to one line.
- **A single relevant emoji at the start of a top-level bullet is
  welcome** as a visual marker (e.g. a warning symbol for a concern, a
  checkmark for a strength) — this is explicitly encouraged, not just
  tolerated.
- **Enforce a length limit and a focus limit together.** Don't just
  cap word count — also state how many points to cover (e.g. "the two
  or three criteria that mattered most," not every criterion
  individually). A word cap alone still allows a long, unfocused list
  of short bullets.
- **Reasoning behind a decision must be included, not just a label.**
  If a skill produces a classification or a recommendation, also
  require a short, self-contained explanation, written for someone who
  was not part of the conversation that produced it.

## Instructions specific to the Genie's own instructions

- **Prefer principles over a numbered script.** Do not write the
  Genie's instructions as a sequential procedure to execute literally.
  Write a small number of durable principles the agent weighs, plus a
  loose, explicitly non-mandatory description of how work typically
  flows.
- **Don't duplicate what the platform already surfaces.** If tool or
  knowledge-base descriptions are already visible to the agent
  automatically, do not repeat that information inside the Genie's own
  instructions — it creates two sources of truth that can drift.
- **A skill-specific precondition belongs in that skill's own
  description**, not in the Genie's general instructions — even if it
  was discovered because of a Genie behavior bug. General instructions
  are for cross-cutting judgment; skill descriptions are for that
  skill's own contract.

## Instructions specific to testing

- **Maintain a running smoke-test list** covering: basic creation,
  duplicate detection, ambiguous matches, already-completed work,
  confirmation gates, implicit ordering dependencies, and scope
  changes mid-process. Update it as new edge cases are discovered
  rather than discarding it after first use.
- **Re-run the relevant tests after every fix**, not just the specific
  case that failed — a fix in shared logic can affect other paths.

## Instructions specific to visuals and non-technical assets

- **Mockups are for inspiration, not final delivery**, unless
  explicitly asked to finalize. Say so plainly, and expect the person
  to rebuild or restyle in their own tool.
- **Push back on a visual metaphor that implies something false.** If
  a color gradient, band width, or category tag implies a precision or
  claim that isn't actually true (e.g. implying a gradual shift when
  the real change is a single discrete jump), say so and propose a
  more honest alternative, even if the false version looks cleaner.
- **Do not name a real customer** in externally-shared material unless
  explicitly cleared to do so. Default to anonymized framing.
- **When real account or research data is available through a
  connector, use it** rather than inventing generic pain points —
  ground problem statements in what was actually said, and cite that
  plainly (e.g. "from real discovery conversations") without
  overclaiming it represents every customer.

## What to avoid

- Do not generate a full system's worth of specification in one
  response when the person is building incrementally — match their
  pace.
- Do not accept a first draft's naming or scope without asking "does
  this name still fit" once behavior has evolved — several skills in
  this project were renamed mid-build for good reason.
- Do not present a mockup or draft as more finished or more validated
  than it is — flag assumptions, guesses, and unverified claims
  explicitly rather than letting them blend in with confirmed facts.

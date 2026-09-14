# Playbook: Designing Agentic Solutions on Workato (Genie + Skills + Recipes)

This document captures general, reusable guidance for building an agentic
solution on Workato — a Genie backed by skills, recipes, data tables, and
human-in-the-loop steps. It is not specific to any one project. Apply it
whenever helping design or build a similar system: an AI agent that
researches, reasons, and takes action through a set of tools, with real
data and real consequences.

## 0. Kickoff checklist — before designing anything

Before scoping any of the patterns below, ask the person directly for
answers to these questions — as an explicit set of clarifying
questions at the start of the engagement, not something inferred
silently from context or carried over from a previous session.
Skipping this tends to surface as rework later, once assumptions
collide with what the person actually wanted.

- **Build mode.** Will this be built using the platform's own AI build
  agent (e.g. Workato's AIRO), a fully manual build, an AI assistant
  driving the builder tools directly, or a CLI/code-based workflow
  (e.g. the open-source Workato Labs toolkit — the `wk` CLI, its
  recipe linter, and the recipe visualizer — for authoring recipes as
  code in a local editor and pushing them to a workspace; experimental
  and community-supported, not an officially SLA'd product as of this
  writing)? This determines who does what for everything that follows.
- **The assistant's role for this engagement.** Active builder using
  tools directly, an advisor producing specs and schemas for manual
  entry, or a hybrid that drafts and waits for approval at each step?
  State this plainly rather than letting it default — and expect it to
  be re-confirmed per session, not assumed to carry over from a
  previous one.
- **Delivery format for advisory output.** When the assistant is an
  advisor drafting specs, schemas, or instructions rather than building
  directly, default to discussing and presenting them conversationally
  in the chat, not as a saved file — only create a file when the
  person explicitly asks to save, export, or share it as one. Don't
  infer a file is wanted just because the content is long or
  structured.
- **Read vs. write tool permission.** Can the assistant read the current
  state of the target environment for grounding, even if it won't
  create or change anything, or should it stay off the builder tools
  entirely?
- **Target environment.** Only relevant if the assistant has any tool
  access (read or write, per the item above) to the platform — in a
  fully manual build with no tool access, the specific project or
  workspace name doesn't change anything the assistant drafts, so skip
  this question. When it does apply, name the specific project,
  workspace, or folder up front, rather than discovering it mid-build.
- **Pace of delivery.** Confirm whether building one piece at a time
  (the default throughout this playbook) applies as-is, or whether the
  audience or format calls for larger, batched increments.
- **Reference material visibility.** Only relevant for a *new* project
  where a separate, similar system happens to already exist elsewhere
  — decide whether the assistant should treat it as an open reference
  throughout the build, or work from requirements alone, so the build
  isn't silently anchored to — or silently ignoring — prior art. This
  doesn't apply to a rebuild of the system itself: `CLAUDE.md`'s step 3
  already makes that system the authoritative source, so re-asking
  this during the section 0 checklist is redundant and confusing —
  skip it in that case.
- **External environment readiness.** Which connections, credentials,
  or integrations need to already be authorized in the target
  environment before build work, or any demo of it, can proceed without
  stalling.
- **Data and naming used during the build.** Confirm whether example
  data, company names, or scenarios are fictional or need anonymizing,
  particularly if any output will be shared outside the immediate team.
- **Scope and time-box.** Is the goal a complete end-to-end system in
  one sitting, or a defined subset / phased delivery?

If, after asking, a question still doesn't have a clear answer — the
person is unsure, or says it doesn't matter for this session — propose
a specific default and state the assumption plainly, rather than
leaving it ambiguous. This is the same principle `genie-skill-design.md`
applies to individual skill design, applied here at the project level.

## 1. Division of responsibility between skills and the agent

- Skills (recipes exposed as tools) fetch or persist data. They never
  reason, score, or judge. All qualitative judgment — scoring, weighing
  criteria, forming a recommendation — belongs to the agent, grounded in
  a shared, inspectable source (see section 7), not buried in a skill.
- Deterministic branching logic — state transitions, deduplication,
  normalization, default values — belongs in code inside the skill's
  recipe, not in the agent's instructions. If a decision can be expressed
  as a fixed rule, encode it once in the skill, not as a rule the agent
  must recall and apply correctly every time.
- Name a skill after its actual behavior, not its original intent.
  Revisit the name whenever behavior evolves — for example, a skill
  that started as "create" but grew into "find or create" needs a new
  name (one past build renamed it to "register") so its own name
  doesn't mislead the agent about what it does. Treat this as one
  illustration of the pattern, not the specific rename to apply.
- Give every skill an explicit WHEN TO USE and WHEN NOT TO USE section.
  Put ordering dependencies and preconditions here explicitly — an agent
  will parallelize calls that look independent unless told a dependency
  exists. A soft instruction ("check X first") is not reliably enough;
  where possible, make the dependency a required input the agent cannot
  supply without having made the prerequisite call.

## 2. Controlling entry into an automated pipeline

- When several different paths can all create the same kind of record
  — the actual set varies by project, but common examples are manual
  entry, self-service submission, agent-driven creation, and bulk
  import — funnel them through one shared "register" skill rather than
  separate paths per entry point.
- Give that skill an explicit boolean controlling whether the new record
  enters downstream automation, plus an optional override for the
  specific resulting state. Require the caller to set these deliberately
  — never default to "enters automation," since an accidental or
  exploratory call should not silently trigger real processing.
- Track *how* a record entered the system as its own field. This is
  useful for debugging and for demonstrating the system's behavior, and
  the taxonomy of entry sources will likely need at least one revision
  once real usage patterns emerge — don't treat the first list as final.

## 3. Synchronous vs. asynchronous agent work

- A small, explicitly confirmed unit of work can run synchronously,
  in-thread, with the result shown directly to the user.
- A larger or open-ended amount of work should be proposed as background
  processing, with its own explicit confirmation, rather than attempted
  synchronously.
- Do not skip confirmation just because a batch is small. A single
  action is still a real, consequential one if it writes data or
  triggers a process — confirmation should be about consequence, not
  about volume.
- Do not hardcode the sync/async boundary as a rigid, unexplained number
  the agent applies mechanically. State the reasoning behind the
  threshold explicitly, and let the agent apply judgment near the
  boundary rather than treating it as a strict cutoff.

## 4. Identifier normalization

- Distinguish two different needs that are easy to conflate:
  - Canonical identity for deduplication — collapsing superficial
    variation down to one base identity so the same real-world thing
    is never recorded twice. Examples: a URL's protocol, "www.", and
    subdomains; an email address's casing or plus-addressing; an
    external system's ID with or without a leading zero or prefix.
  - Superficial cleanup that preserves identity — normalizing
    formatting without collapsing anything actually distinguishing.
    Examples: stripping a URL's trailing slash, query string, or
    fragment while keeping a path that's part of what makes the record
    unique; trimming whitespace from a name or ID without changing
    what it refers to.
- Reusing a "collapse to base identity" normalizer on something that
  needs its distinguishing part preserved is a common and easy-to-miss
  bug. If two different kinds of identifiers exist in the same system
  — say, a dedup key and a display value — give them two different
  normalization functions, whatever the identifier type happens to be.

## 5. Skill output schema conventions

- Every skill output should include a plain-language status or message
  field, meant to be read directly by the agent (or relayed to the user)
  without further interpretation.
- That field should communicate *outcome and confidence*, not restate
  content that already exists in structured fields — avoid asking the
  agent to reconstruct a sentence from raw fields when a well-written
  message field can just say what happened.
- When using structured output support in an LLM API (e.g. JSON Schema
  response formats), remember that `required` typically must list every
  key present in `properties` — a very common source of otherwise
  confusing validation errors when a field is added or removed later.
- Keep types consistent end to end. A numeric field should return `null`
  when no value exists, not an empty string — type mismatches surface
  as silent, hard-to-trace bugs much later in a pipeline.
- Prefer boolean, pre-classified fields over asking the agent to
  classify a value itself (e.g. an `already_done` boolean rather than
  requiring the agent to remember which of several status values count
  as "done"). Compute the classification once, in code, and expose the
  result — don't make the agent re-derive a categorization it could get
  wrong under load.

## 6. Working with LLM APIs that support tool/browsing use

- Force actual tool use when a call must not be answerable from the
  model's own general knowledge — an "auto" tool choice setting may
  simply skip the tool for anything the model feels confident about.
- Reasoning effort settings materially change whether a model does
  shallow (single-page, single-query) or deep (multi-hop, multi-query)
  exploration. Set this deliberately per skill based on how thorough
  that skill actually needs to be, not uniformly across every skill.
- Have each research skill report how much real work was actually done
  (e.g. number of tool calls made). Expose this so the calling agent —
  and a human reviewing the result — can gauge confidence, rather than
  trusting a confident-sounding output at face value.
- Prefer structured JSON output over parsing free-form prose whenever a
  result needs to be split into distinct fields. Parsing prose after the
  fact is fragile and breaks as soon as phrasing varies.

## 7. A shared, inspectable source for judgment criteria

- When an agent must apply a rubric, criteria catalog, or scoring
  method, store that rubric somewhere it can retrieve and re-consult
  independently of its own instructions (e.g. a knowledge base), rather
  than embedding it as static prompt text.
- This keeps the rubric editable without touching the agent's
  instructions, keeps it consistent across many runs, and makes the
  system's reasoning auditable — a reviewer can see exactly what
  standard was applied.
- Instruct the agent to consult this source fresh for each new subject
  it evaluates, rather than relying on what it recalled earlier in a
  long conversation, so updates to the rubric take effect immediately.

## 8. Migrating a hand-rolled status field onto a platform-native mechanism

- If the platform offers a native lifecycle/workflow tracking mechanism,
  treat adopting it as a genuine migration, not an addition alongside
  what you already have. Running both a custom status field and a
  platform-native mechanism in parallel reintroduces the duplication
  you were trying to remove.
- Before migrating, identify every write path and every trigger
  condition that currently depends on the old field — a partial
  migration where some paths still write the old mechanism is worse
  than not migrating at all, since it silently breaks consistency.
- After migrating, retest the same scenarios that validated the old
  mechanism before removing the old field entirely.

## 9. Human-in-the-loop principles for an agent acting on real data

- Mentioning, asking about, or expressing interest in something is not
  the same as authorizing an action on it. Treat these as distinct, and
  when in doubt about which one occurred, ask rather than assume intent.
- Treat any consequential result — a finished analysis, a scored
  recommendation, a completed piece of research — as a proposal to the
  user until they explicitly confirm it should be saved or acted upon.
  Do not persist meaningful results automatically just because the
  agent's reasoning is finished.
- Never let an agent fabricate an identifier it does not actually have
  (a URL, an ID, a reference number). If it isn't known, look it up or
  search for it and confirm the result with the user, rather than
  guessing something plausible.
- Before repeating expensive work, check whether it has already been
  done. Make this check something the agent can rely on directly (a
  precomputed flag), not something it has to infer from a list of
  possible states it needs to remember correctly.
- Design for the user changing their mind mid-process (adding, removing,
  or reconsidering a selection) rather than assuming a single linear
  choice — a good agent should be able to adapt without restarting.

## 10. Instruction philosophy for the agent itself

- Prefer a small set of durable principles the agent weighs over a long,
  numbered script it executes literally. A step-by-step script tends to
  accumulate as a reaction to specific bugs found during testing, and
  ends up encoding today's fixes as rigid, un-agentic behavior that
  doesn't generalize to cases you didn't anticipate.
- Use step-by-step sequencing only to describe the *typical shape* of a
  task, explicitly framed as non-mandatory, not as a checklist the agent
  must always complete in order.
- State principles in terms of what matters and why (e.g. "don't act on
  unconfirmed intent, because it can trigger irreversible downstream
  effects"), not just what to do, so the agent can extend the reasoning
  to situations the instructions didn't explicitly cover.
- If the platform already exposes tool/knowledge-base descriptions to
  the agent automatically, don't duplicate that information in the
  agent's own instructions — it creates two sources of truth that can
  drift out of sync.

## 11. The most common bug class in this kind of build

- Field-name mismatches between what one step produces and what the
  next step reads — caused by renaming, recasing, or restructuring a
  field partway through a build without updating every place that
  consumes it. This was, empirically, the single most frequent source
  of bugs across an entire build of this kind.
- Whenever a field is renamed or a schema changes shape, deliberately
  search for every other place that field name appears — the skill's
  own schema, any recipe step reading it, any other skill that might
  reference the same concept — rather than assuming the rename is
  self-contained.

## 12. A reusable smoke-test mindset

When validating a system like this, deliberately test:
- Basic creation and duplicate detection.
- Ambiguous matches (multiple plausible results) and how the agent
  handles disambiguation.
- Already-completed work, and whether the agent avoids redoing it
  unprompted.
- Confirmation gates — does the agent proceed only after genuine
  confirmation, and does it correctly distinguish a mention or question
  from an explicit go-ahead?
- Implicit ordering dependencies — does the agent ever parallelize calls
  that should have run sequentially, especially around a status or
  precondition check?
- Scope changes mid-process — does the agent adapt gracefully if the
  user adds, removes, or reconsiders part of a request already in
  progress?

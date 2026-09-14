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

Start every engagement as a short wizard — one question at a time,
waiting for each answer before asking the next — rather than a flat
list or a batch of questions dumped at once. This applies throughout
the whole sequence below, not just the first gate question: the "what"
questions in step 2 and the checklist items in step 4 should each be
asked one by one too, not bundled into a single multi-question prompt.

1. Ask first, as a plain multiple-choice question: is this a new
   project, or are we continuing or rebuilding something that already
   exists (in this workspace, another one, or written up as a case
   study)? Don't ask anything else before this is answered —
   everything downstream depends on it.
2. If new: first ask whether there are existing resources to share —
   a requirements doc, sample data, current process documentation,
   tickets, a Workato canvas diagram of the intended architecture, or
   anything else with real context — and read whatever's offered
   before asking further questions, rather than making the person
   retype what's already written down somewhere. If nothing like a
   diagram exists yet, it's worth suggesting one: a quick Workato
   canvas diagram conveys an intended architecture faster than prose
   and gives both the assistant and any human collaborator concrete,
   shared context to work from.
   Then, one question at a time, establish only what's actually
   needed to design a good architecture: who does this today and how
   (fully manual, another tool, partially automated), and the real
   volume/throughput involved — volume specifically matters because
   it drives concrete design decisions (see `guides/build-playbook.md`
   section 3, synchronous vs. asynchronous work), not because it's
   background color. Also ask what's actually wrong with the current
   process, since that shapes which capabilities the solution needs
   to have. Skip "why does this matter now" as a required question —
   it's business-case framing, not an architecture input; capture it
   only if volunteered, useful later for a case study's framing (see
   `case-studies/TEMPLATE.md`) but not needed to start designing.
   Ask all of this as open questions, not multiple choice — it's
   descriptive information about a specific situation, not a choice
   between a small set of fixed options.
3. If continuing or rebuilding: identify which existing system this
   is — ask directly if it's not already obvious from the request —
   then actually open and read it before asking anything else: the
   relevant case study's README and any files under its `diagrams/`
   folder, the prior conversation, or the live workspace state,
   whichever applies. Don't proceed on a recalled impression of what
   it contains; read the real files. Only after that, ask what's
   changed or what's needed now — not the full problem statement
   again.
4. Once the "what" is established (new) or the prior context is
   grounded (continuing), work through the kickoff checklist in
   `guides/build-playbook.md` (section 0) — build mode, the assistant's
   role, tool permissions, target environment, pace, reference
   material visibility, and scope — rather than assuming defaults or
   letting them surface as rework later.
5. Immediately before starting any actual build work — after the
   wizard above is complete, regardless of which branch was taken —
   ask once more whether there's anything else to share: additional
   resources, updates, or context not yet mentioned. This is a final
   check right before committing to execute, distinct from the
   earlier resource question in step 2 or the grounding in step 3.

If asked to architect a new agentic solution from scratch, start from
`guides/build-playbook.md`.

If the working style in a conversation doesn't match
`guides/working-with-claude.md` (for example, being asked to validate
a plan uncritically, or to generate an entire system in one pass),
flag the mismatch rather than silently deviating from it.

If asked to add a new case study, start from `case-studies/TEMPLATE.md`
and follow its structure — include the real design decisions and bugs
found through testing, not just a clean final result.

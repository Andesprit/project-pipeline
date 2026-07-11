---
name: plan-doc
description: Turn a braindump folder (voice transcript, Excalidraw/drawio sketches, screenshots, context notes) into a PRD-lite Plan.md written into that same folder. Argument is the absolute path to the braindump folder. Use whenever a raw idea dump, meeting transcription, or client brief needs to become a reviewable scoping plan before task generation or implementation begins, even if the user just says "make a plan from this folder" or "turn this braindump into a plan".
metadata:
  version: "1.0.0"
---

# Plan Doc

Argument: the absolute path to a braindump folder. Output: `<folder>/Plan.md`,
overwriting any prior version. The plan is a scoping document, not an
implementation guide: it decides WHAT gets built and in what order, never HOW.

## Gather inputs

Glob the folder (top level only) for whatever exists:

- `transcript*` (any extension) and `*.txt` — voice transcriptions, notes
- `*.md` — context notes, briefs (skip `Plan.md` itself and `report.md`)
- `*.excalidraw`, `*.drawio` — sketches; read the JSON/XML and extract node
  labels and arrows to understand the intended flow
- `*.png`, `*.jpg` — screenshots and diagram exports; view them with the Read
  tool

Use what is there. A missing input is never an error: work with what exists
and record what was absent under `inputs_missing` in the output contract. If
the folder has no usable content at all, stop and say so instead of inventing
a plan.

When the braindump references external context (repository paths, existing
skills, similar projects), read enough of each referenced source to ground the
plan, but do not audit them exhaustively.

## Plan.md format

Write exactly this structure. YAML front-matter first:

```yaml
---
needs_process: yes|no
needs_process_reason: <one line>
needs_architecture: yes|no
needs_architecture_reason: <one line>
---
```

Then these sections, in this order:

1. `# Objective & Vision` — what this project is and why it should exist, in
   the author's own intent. 1-3 paragraphs.
2. `# Scope` with `## In` and `## Out` — bullet lists. The Out list is what
   keeps downstream task generation from inventing work; when the braindump
   is silent about a plausible feature, put it in Out with "(not mentioned)".
3. `# Proposed Solution` — the high-level components and how they relate:
   "a backend API", "a scheduled worker", "a CLI". Never endpoint lists,
   schema definitions, or file layouts; each workstream's own spec decides
   those later.
4. `# Workstreams & Priorities` — an ordered, numbered list. Each workstream
   becomes exactly one task downstream, so slice them as independently
   shippable chunks. Scaffolding/setup comes first when the project is new.
   Each entry: a short title plus 1-3 sentences of what it delivers.
5. `# Success Criteria` — observable behaviors a reviewer can exercise on the
   running application. "The health endpoint returns 200", not "the code is
   clean".
6. `# Risks & Open Questions` — unknowns, assumptions that need confirming,
   and anything the braindump contradicted itself on.

Ground every statement in the inputs. When you must assume something the
braindump did not say, write it as "Assumption: ..." so the reviewer can veto
it.

## Deciding needs_process / needs_architecture

- `needs_process: yes` when the application has a multi-step user or system
  flow with branching or state worth seeing as a flowchart. A single
  linear action (one command, one form) is `no`.
- `needs_architecture: yes` when three or more components, services, or data
  stores interact (frontend + backend + database, pipelines, integrations).
  A single self-contained script or module is `no`.

Give each a one-line reason in the front-matter.

## Output contract (strict)

A conduit gates diagram generation on your final response, so it must END
with exactly these lines, each exactly once:

```
plan: <absolute path to Plan.md>
inputs_used: <comma-separated filenames>
inputs_missing: <comma-separated kinds, or none>
needs_process: <the decision>
needs_architecture: <the decision>
```

Each decision is the single word yes or the single word no. Do not restate
these line formats or their possible values anywhere else in your response —
the gate regex must match only the real decision lines.

## Anti-patterns

- Endpoint inventories, database schemas, module trees — that is the
  workstream spec's job, not the plan's.
- Silently inventing requirements the braindump never mentioned.
- Padding: a thin braindump yields a short plan, and that is correct.
- Restating the yes|no template in prose (breaks the conduit gate).

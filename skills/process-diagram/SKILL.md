---
name: process-diagram
description: Create process.excalidraw (the application's process flow diagram) in a braindump folder from its Plan.md. Argument is the absolute path to the folder containing Plan.md. Use when a plan needs its process flow visualized as an Excalidraw flowchart, including when a conduit or user says "make the process diagram for this plan".
metadata:
  version: "1.1.0"
---

# Process Diagram

Argument: the absolute path to a folder containing `Plan.md`. Output:
`<folder>/process.excalidraw` plus an exported preview (`process.png` or
`process.svg`) next to it.

Scope: the APPLICATION's process flow - actors, steps, decisions, states, and
how a user or system moves through them. Not the deployment architecture and
not the codebase structure (that is architecture-diagram's job).

Inputs, in order of authority:

1. `<folder>/Plan.md` - primary. The flow must match `# Proposed Solution`
   and `# Scope`; label decision branches with the actual conditions the plan
   describes.
2. The raw braindump files in the folder (transcripts, sketches) - secondary
   reference for details the plan compressed away. Where they conflict with
   Plan.md, Plan.md wins (it survived human review).

## Delegate to the excalidraw plugin skill (required)

Invoke the `excalidraw:excalidraw-skill` skill (the 365-skills excalidraw
plugin) and follow its workflow fully - element construction, export, and its
verification steps. This skill only adds the input/output contract around it.

If that plugin skill is not available, stop immediately and end your response
with `process_diagram: blocked` plus a one-line reason. Never hand-roll the
diagram methodology from memory.

## Brief to carry into the excalidraw workflow

- Derive the flow from Plan.md's Proposed Solution and Scope-In; one lane of
  boxes per actor/system when more than one participates.
- Pick depth by plan size: a plan with 3 workstreams gets a simple flow; only
  go comprehensive when the plan describes branching or multi-actor behavior.
- Every decision diamond is labeled with the plan's real condition, not
  "yes/no?" placeholders.
- Save the diagram as `<folder>/process.excalidraw` (absolute path - the
  working directory is wherever the conduit was launched).

## Non-negotiables

- The output file must be valid Excalidraw JSON that opens clean in
  excalidraw.com.
- Export a preview (PNG or SVG, whichever the plugin's export path supports
  on this machine), look at it, and fix visual defects before finishing. Do
  not finish without having actually viewed at least one export.
- The reviewer at the HITL gate reads this diagram instead of the full plan
  text - legibility beats completeness.

## Output contract (strict)

End your response with exactly these lines:

```
process_diagram: written|blocked
path: <absolute path to process.excalidraw, or none>
render_iterations: <N>
```

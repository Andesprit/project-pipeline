---
name: architecture-diagram
description: Create architecture.drawio in a braindump folder from its Plan.md, via the drawio plugin skill and the native draw.io desktop CLI. Argument is the absolute path to the folder containing Plan.md. Use when a plan needs its architecture visualized as a draw.io diagram, including when a conduit or user says "make the architecture diagram for this plan".
metadata:
  version: "1.1.0"
---

# Architecture Diagram

Argument: the absolute path to a folder containing `Plan.md`. Outputs:
`<folder>/architecture.drawio` plus an exported preview
(`architecture.drawio.png`) next to it.

Input: `<folder>/Plan.md`, primarily `# Proposed Solution`. Raw braindump
files in the folder are secondary reference; Plan.md wins on conflict.

## Choose the abstraction level from the plan

State the choice in your output. Decision table:

| Plan describes | Abstraction | Diagram shows |
|---|---|---|
| Deployed services, infra, databases, third-party APIs | `deployment` | Services, stores, external systems, and the connections between them |
| One application with internal components | `modules` | The app's modules/layers and their dependencies |
| The product IS an API and the plan says so explicitly | `endpoints` | Resource groups and data flows (still no exhaustive route lists) |

Default away from `endpoints`; a plan that merely mentions "a backend" gets
`deployment` or `modules`, never a route inventory.

## Delegate to the drawio plugin skill (required)

Invoke the `drawio:drawio-skill` skill (the 365-skills drawio plugin) and
follow its workflow fully - shape selection, layout, structural validation
(`scripts/validate.py`), export via the native draw.io desktop CLI, and its
vision self-check. This skill only adds the input/output contract around it.

If that plugin skill is not available, stop immediately and end your response
with `architecture_diagram: blocked` plus a one-line reason. If the draw.io
desktop CLI is missing, still write the `.drawio` file, validate it
structurally, and note the skipped export in your output instead of blocking.

## Generate

- Save the diagram as `<folder>/architecture.drawio` (absolute path - the
  working directory is wherever the conduit was launched).
- Run the plugin's structural validation before finishing, and export a
  preview next to it when the CLI is available.
- Keep it high level: components and their relationships, matching the
  chosen abstraction. No exhaustive route lists or schema dumps.

## Output contract (strict)

End your response with exactly these lines:

```
architecture_diagram: written|blocked
path: <absolute path to architecture.drawio, or none>
abstraction: deployment|modules|endpoints
```

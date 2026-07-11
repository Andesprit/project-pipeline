# project-pipeline

**Sit down for an hour, brain-dump a project (voice transcript, sketches,
context), and let the pipeline take it the rest of the way: a reviewable
plan with diagrams, a prioritized task backlog, an implementation run where
every task is built by one agent and checked by another, and a final report
proving the application actually does what the plan promised.**

A [flow-atelier](https://github.com/LGuillermoAngaritaG/flow-atelier)
package. Two human gates keep you in control while the skills are being
tuned: you approve the plan before tasks exist, and you approve the tasks
before any code is written. As the pipeline earns trust, the goal is to
delete the gates - the golden path is braindump in, working app out.

```
braindump/ (transcript + sketches + context)
   |
   v
plan-autonomous-projects ........ Plan.md (+ process.excalidraw, architecture.drawio when needed)
   |
   v
[HITL] you review and edit the plan
   |
   v
implement-autonomous-projects ... project.md + raw tasks -> spec'd backlog cards, prioritized per plan
   |
   v
[HITL] you move approved cards from 00_backlog/ to 01_to-do/  (the move IS the approval)
   |
   v
run-improvement ................. work-one-todo drains the queue (build agent + review agent per task)
   |
   v
review-application .............. runs the real app against the plan's success criteria
   |
   v
report.md in the braindump folder - the final human review is a 2-minute read
```

The whole thing is also wired as one master conduit
(`implementation-conduit`) with the two gates inline.

## Prerequisites

1. **flow-atelier** and the **autonomous-projects package** (hard
   requirement - this package calls its conduits and scripts by name):

   ```bash
   uv tool install flow-atelier
   atelier add LGuillermoAngaritaG/autonomous-projects
   ```

2. **CLIs on PATH**: `claude` (Claude Code), `python3`, `curl`, `git`.

3. **Diagram plugin skills** (only needed when plans call for diagrams) -
   installed inside Claude Code:

   ```
   /plugin marketplace add Agents365-ai/365-skills
   /plugin install excalidraw@365-skills
   /plugin install drawio@365-skills
   /reload-plugins
   ```

   The drawio plugin exports via the native draw.io desktop CLI - install it
   with `brew install drawio` (macOS). Without it, architecture diagrams are
   still written, just not exported to PNG.

## Install

```bash
atelier add LGuillermoAngaritaG/project-pipeline
# or from a local checkout:
atelier add ./atelier-examples/project-pipeline
atelier list conduits   # the four conduits below should appear
```

This also installs the four skills (`plan-doc`, `process-diagram`,
`architecture-diagram`, `review-application`) into `~/.claude/skills/`, where
the harness steps invoke them as slash commands.

## Run

Create a braindump folder anywhere and drop your material in it: a
`transcript.md` (or any `transcript*`/`.txt` file), sketches
(`.excalidraw`/`.drawio`/`.png`), and an optional `context.md` with repo
paths and links. Then either run the whole pipeline:

```bash
atelier run implementation-conduit \
  --input braindump_dir=/abs/path/to/braindump \
  --input project_root=/abs/path/to/repo
```

...or run it phase by phase (useful while tuning - the "gate" is simply you,
between runs):

```bash
atelier run plan-autonomous-projects --input braindump_dir=/abs/path/to/braindump
# review/edit Plan.md and the diagrams

atelier run implement-autonomous-projects \
  --input braindump_dir=/abs/path/to/braindump --input project_root=/abs/path/to/repo
# review 00_backlog/, move approved cards to 01_to-do/

atelier run run-improvement \
  --input project_root=/abs/path/to/repo --input braindump_dir=/abs/path/to/braindump
# read report.md in the braindump folder
```

`project_root` may be a brand-new path: it is created, `git init`-ed and
scaffolded automatically. If it is an existing repo, the pipeline improves it
in place.

## The two gates

- **Plan gate** (`hitl_plan`): the pipeline pauses and asks `plan_confirm`.
  Review `Plan.md` (and diagrams) in the braindump folder and edit them
  freely first - the next phase reads them from disk. Type `yes` to
  continue; anything else stops the pipeline cleanly.
- **Task gate** (`hitl_tasks`): review the cards in
  `<project_root>/.atelier/project/00_backlog/` and MOVE the ones you
  approve into `01_to-do/` - the move is the approval. Cards left behind are
  not worked. Adjust `priority:` front-matter if you disagree with the order
  (lower runs sooner). Then type `yes`.

## Artifact conventions

- The braindump folder is the session workspace: inputs and outputs live
  together (`Plan.md`, `process.excalidraw`, `architecture.drawio`,
  `report.md`).
- `Plan.md` front-matter declares `needs_process` / `needs_architecture`;
  the plan conduit generates only the diagrams the plan calls for.
- Task priority equals plan workstream order (scaffolding first). The
  improver resets priorities, so the `prioritize` step restores them from
  the "This is workstream N of M" line embedded in each task description.

## Known ceilings

- `improve-all-tasks` iterates at most 20 times per invocation, so plans
  with more than 20 workstreams need a second `implement-autonomous-projects`
  run (realistic plans have 3-10).
- One braindump folder = one project. To run several projects, run the
  pipeline once per folder.

## First test drive

Use a real meeting as the fixture: create a folder, drop the meeting
transcription in as `transcript.md` (plus any whiteboard export), and run
`plan-autonomous-projects` against it. Passing means the person who was in
the meeting recognizes the plan. Then do one full `implementation-conduit`
run on a small throwaway app before pointing it at a real repo.

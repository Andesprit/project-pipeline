# Running the implementation conduit on a real project

Operator guide for taking a real braindump to a reviewed application with
`implementation-conduit`. The README covers installation; this covers how to
actually drive a run.

## 0. One-time checklist

- `atelier check` is green (installs verified: autonomous-projects first,
  then project-pipeline).
- `claude`, `python3`, `curl`, `git` on PATH.
- Diagram plugins installed in Claude Code: `excalidraw@365-skills` and
  `drawio@365-skills` (plus the draw.io desktop app for PNG exports).

## 1. Prepare the braindump folder

Create one folder per project, anywhere (for example
`~/braindumps/<project>/`). Drop in whatever exists - the plan skill uses
what is there and records what was missing:

- `transcript.md` (or any `transcript*` / `.txt`): the recording
  transcription of the meeting or your solo voice dump. Raw and messy is
  fine; do not clean it up.
- Sketches: `.excalidraw`, `.drawio`, `.png` exports of whatever was drawn.
- `context.md`: the highest-leverage file. Absolute paths to relevant repos,
  similar projects to imitate, links, and any hard constraints ("must use
  Postgres", "reuse the auth module from X"). One line per item.

What makes a good braindump: say what should exist and why, name what is
out of scope out loud, and mention concrete references. The plan is only as
grounded as this folder.

## 2. Decide the project_root

- **New project**: pass a path that does not exist yet, for example
  `~/projects/<name>`. The pipeline creates the repo and its project board.
  Include application setup in the first working path. Give the scaffold its
  own card only if the plan names someone who can use it independently.
- **Existing repo**: the pipeline edits it IN PLACE and commits per task.
  For anything you care about, point `project_root` at a fresh clone or a
  dedicated branch checkout (`git worktree add` works well), review the
  result, then merge like any other branch.
- One repo can host only one `.atelier/project/` board. Re-running the
  pipeline against the same repo reuses the existing board and adds new
  tasks to it.

## 3. Launch

```bash
atelier run implementation-conduit \
  --input braindump_dir=/abs/path/to/braindump \
  --input project_root=/abs/path/to/repo
```

Rules of thumb:

- Both inputs must be absolute paths.
- Run it from a terminal you can come back to - the two gates are prompts in
  that terminal.
- Do not start a second flow against the same braindump folder or repo while
  one is running; they will overwrite each other's artifacts.
- Expect the plan phase to take 10-20 minutes (plan plus diagrams), and each
  approved task a few minutes to tens of minutes depending on size.

Everything the run does is logged under `.atelier/flows/<flow_id>/` in the
directory you launched from (or `~/.atelier/flows/` when there is no local
`.atelier/`). `atelier stop <flow_id>` from that same directory stops a run
gracefully; `atelier run --resume <flow_id>` continues one that was stopped
or died.

## 4. Gate 1 - the plan

The run pauses with a `plan_confirm` prompt. Before typing anything:

1. Open `<braindump_dir>/Plan.md`. Check that Scope/Out reflects what you
   want to exclude. Read the workstreams in priority order. Each should
   describe one independently usable result that one agent can deliver and
   a reviewer can check. Each additional workstream should explain why it
   needs fresh context, parallel work, an independent test or review, or a
   different agent or permissions. Check that the Success Criteria would
   let you sign off on the application.
2. Glance at `process.png` / `architecture.drawio.png` - they exist to make
   this review fast.
3. Statements marked "Assumption:" are the model filling gaps - veto or
   confirm each one.

Edit the files directly; the next phase reads them from disk, so your edits
are authoritative. Then answer `yes` (continue) or anything else (stop
cleanly - fix the braindump or the plan and rerun).

## 5. Gate 2 - the tasks

The run pauses with a `tasks_confirm` prompt after the backlog is generated:

1. Open `<project_root>/.atelier/project/00_backlog/` - one `task_*.md`
   card per workstream, each with a Description, Spec, and Plan section and
   a `priority:` matching the plan's workstream order.
2. MOVE the cards you approve into `../01_to-do/`. The move is the approval;
   anything left in `00_backlog/` is not worked.
3. Edit `priority:` (lower runs first) or the card text if you disagree.
   Move rejected cards to `00_abandoned/` with a note if you want a record.

Then type `yes`. Only what is in `01_to-do/` gets built.

## 6. While it runs and after it finishes

Each task is picked by priority, built by one agent, and independently
reviewed by a second one (which re-runs the tests) until a DONE verdict;
finished tasks land in `03_to-review/`, stuck ones in `05_blocked/` with a
note. When the queue drains, the review skill launches the real application
against the plan's Success Criteria and writes
`<braindump_dir>/report.md`.

Your final pass:

1. Read `report.md` - verdict first, then the Gaps section. `partial` means
   something could not be verified; the report names the exact command to
   close each gap.
2. Skim the diffs/commits per task in `03_to-review/`; move accepted tasks
   to `04_done/` (only humans move cards there).
3. Anything in `05_blocked/` needs a decision: fix the blocker and move the
   card back to `01_to-do/`, or abandon it.

## 7. When something goes wrong

- **Guard fails at start**: the message names the missing dependency or
  input; nothing has run yet, fix and relaunch.
- **You answered no at a gate**: nothing downstream ran. Fix the artifact
  the gate was about, then either relaunch the master or run the remaining
  phases individually (see below).
- **The run died mid-flight** (laptop slept, network): `atelier run
  --resume <flow_id>` from the launch directory, or just run the remaining
  phase conduits by hand - every phase is idempotent-friendly (existing
  project boards are reused, Plan.md is only rewritten by the plan phase).
- **A task loops without reaching DONE**: it ends up in `05_blocked/` with
  the reviewer's reason; the run continues with the next task.
- **More than 20 workstreams**: the task-improvement loop caps at 20 per
  run; run `implement-autonomous-projects` a second time to drain the rest.

## 8. Phase-by-phase mode

The same pipeline, one conduit at a time - useful while tuning skills or
when you want the gates to be asynchronous instead of a waiting prompt:

```bash
atelier run plan-autonomous-projects --input braindump_dir=...
# review/edit Plan.md and diagrams at your leisure
atelier run implement-autonomous-projects --input braindump_dir=... --input project_root=...
# move approved cards to 01_to-do/ at your leisure
atelier run run-improvement --input project_root=... --input braindump_dir=...
# read report.md
```

`run-improvement` on an empty `01_to-do/` is safe: it reports the empty
queue and exits without touching anything.

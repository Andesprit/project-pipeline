---
name: review-application
description: Run the real application end-to-end against the Success Criteria in a Plan.md and write report.md next to the plan. Arguments are the project root and the absolute path to Plan.md. Never unit tests or type checks as evidence - the running app, observed. Use after an implementation phase finishes, or whenever someone asks "does the app actually do what the plan says".
metadata:
  version: "1.0.0"
---

# Review Application

Arguments: `<project_root>` `<plan_path>`. Output: `report.md` written into
the directory containing Plan.md (`dirname(plan_path)/report.md`).

## Core rule

The application's behavior must be OBSERVED by running the real thing
end-to-end. Unit tests, type checks, linters, and successful builds are not
verification: they prove properties of the code, not that the application
does what the plan promised. If you catch yourself reaching for
`pytest`/`jest`/`go test` as the evidence, stop and run the app instead.

Never claim `pass` for anything you could not actually run. Blocked is
`partial` or `fail`, with the gap named.

## Step 1 - Success criteria checklist

Read `<plan_path>` and turn `# Success Criteria` into a numbered checklist.
Every criterion ends the review in exactly one state: verified (with an
evidence run), failed (with an evidence run), or explicitly not-verified
(with the reason and the smallest command that would close the gap).

Also skim `# Scope` and `# Proposed Solution` so you recognize
out-of-scope behavior and do not fail the app for lacking it.

## Step 2 - Derive the run command (never guess)

Look in this order and take the first real answer:

1. `README.md` / `CONTRIBUTING.md` / `docs/`
2. `package.json` scripts, `Makefile`, `pyproject.toml [project.scripts]`,
   `justfile`
3. `Dockerfile` / `docker-compose.yml` / `.env.example`
4. CI config (`.github/workflows/*`, `.gitlab-ci.yml`)

State the intended command in your response BEFORE running it. If required
config or secrets are missing, do not fabricate them: record the blocker as
a finding and continue with whatever criteria remain runnable. A project
with no discoverable run instructions is itself a report finding.

## Step 3 - Scenario list

Before running anything, write down:

- At least one happy-path run per success criterion (grouping several
  criteria into one run is fine when one exercise genuinely covers them).
- At least one deliberate failure path (bad input, missing resource,
  wrong flag) - a healthy app fails loudly and safely.
- The blast radius: side effects to check afterwards (files written,
  database rows, log lines, queue messages).

## Step 4 - Run and observe

Pick the branch that matches the app:

- **HTTP API**: boot in the background, wait for the listening line, `curl -i`
  each changed path happy+failure, then verify the side effects (DB row, file,
  log), then stop the service.
- **CLI / script**: the real invocation with real arguments - not `--help`,
  not `--dry-run`. Capture stdout, stderr, and the exit code. Re-run once if
  idempotency is claimed.
- **Web UI**: drive a real browser (DevTools/Playwright MCP if available, or
  a short Playwright script); screenshot each state; exercise one unhappy
  path.
- **Library**: exercise through the nearest in-repo consumer or a small
  script importing it with realistic inputs; never mock the thing under
  review.

For every run, record what you actually saw - including output that
surprised you. A 200 response that did not write the row is a broken
endpoint.

## Step 5 - report.md (always required)

Write `dirname(plan_path)/report.md` in exactly this shape:

```markdown
# Verification report - <project name from Plan.md>

**Date**: <date>
**Scope**: <what was reviewed, one line>
**Verdict**: pass | partial | fail

## Success criteria

| # | Criterion | Met? | Evidence |
|---|-----------|------|----------|
| 1 | <criterion> | yes/no/not verified | run 1 |

## How the app runs

<exact boot/invocation commands and where you learned them>

## End-to-end runs

### Run 1 - <name>
- **Command**: <exact command>
- **Observed**: <what actually happened - output, status, timing>
- **Side effects checked**: <what you inspected and what you found>
- **Interpretation**: <what this proves or disproves about the criterion>

## Gaps / not verified

<each unverified criterion: why, and the smallest exact command the user
could run to close the gap. "none" if everything was verified>

## Artifacts

<paths to screenshots, saved responses, logs - or "none">
```

The Interpretation field is mandatory per run; "tested, works" is not a
report. Verdict rules: `pass` only when every criterion is verified;
`partial` when some are; `fail` when a criterion is contradicted by an
observed run or the app cannot start at all.

## When you genuinely cannot run it

Missing credentials, external hardware, paid services: never `pass`. Write
`partial` or `fail`, list exactly what was blocked in Gaps, and give the
user the smallest command to finish the verification themselves.

## Output contract (strict)

End your response with exactly these lines:

```
verdict: pass|partial|fail
report: <absolute path to report.md>
```

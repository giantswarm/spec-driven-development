---
name: to-issues
description: Break a finished spec into independently-grabbable GitHub issues — tracer-bullet vertical slices, each declaring its blocking edges — and publish them with whatever GitHub integration this environment has. Optional last step of the plan workflow. Use when the user says "to-issues", "break this into issues", "create the tickets for this plan".
---

# to-issues

Break a plan into a set of **issues**: tracer-bullet vertical slices, each declaring the
issues that **block** it.

**This step is optional.** Many teams run the plan workflow purely to produce a reviewed
Spec and let the implementing agent work straight from it. Only run this when the team
tracks work as issues.

## Publishing target

Issues go to **GitHub**, in the repo the code lives in — not this plans repo — so each
issue sits with the code it will be PR'd against. Use whichever GitHub integration this
environment offers, in this order of preference:

1. a GitHub MCP server, if one is connected;
2. the `gh` CLI (`gh issue create`, `gh issue edit`, `gh api`);
3. failing both, write the issues to `<plan-dir>/issues/<NN>-<slug>.md`, numbered from
   `01` in dependency order, and tell the user they were not published.

If your team publishes to a project board, a custom tracker or a non-GitHub system, this
is the one skill to replace. Nothing else in the workflow depends on how issues are
created.

## Process

### 1. Gather context

Work from whatever is already in the conversation. If the user passes a reference (a spec
path — for a plan in this repo that is `<plan-dir>/SPEC.md` — or an issue number or URL),
fetch it and read its full body and comments.

### 2. Explore the codebase (optional)

If you have not already explored the target codebase, do so. Issue titles and
descriptions should use the project's glossary vocabulary and respect ADRs in the area
you're touching. Look for opportunities to prefactor the code to make the implementation
easier: "make the change easy, then make the easy change."

### 3. Draft vertical slices

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests) — vertical, NOT a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window
- Any prefactoring should be done first

</vertical-slice-rules>

Give each issue its **blocking edges** — the other issues that must complete before it
can start. An issue with no blockers can start immediately.

**Wide refactors are the exception to vertical slicing.** A **wide refactor** is one
mechanical change — rename a column, retype a shared symbol — whose **blast radius** fans
across the whole codebase, so a single edit breaks thousands of call sites at once and no
vertical slice can land green. Don't force it into a tracer bullet; sequence it as
**expand–contract**. First expand: add the new form beside the old so nothing breaks. Then
migrate the call sites over in batches sized by blast radius (per package, per directory),
each batch its own issue blocked by the expand, keeping CI green batch to batch because
the old form still exists. Finally contract: delete the old form once no caller remains,
in an issue blocked by every migrate batch. When even the batches can't stay green alone,
keep the sequence but let them share an integration branch that all block a final
integrate-and-verify issue — green is promised only there.

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each issue, show:

- **Title**: short descriptive name
- **Blocked by**: which other issues (if any) must complete first
- **What it delivers**: the end-to-end behaviour this issue makes work

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the blocking edges correct — does each issue only depend on issues that genuinely gate it?
- Should any issues be merged or split further?

Iterate until the user approves the breakdown.

### 5. Publish

Publish one issue per approved slice, **in dependency order** (blockers first), so each
issue's "Blocked by" section can reference real issue numbers.

If the spec carries an `**Epic:**` line under its title, link every slice as a sub-issue of
that epic where the integration supports it (GitHub sub-issues, or a task list on the
epic). Do NOT close or modify the epic or any other parent issue.

<issue-template>

## Parent

Epic: `<owner>/<repo>#<n>` — one line on the epic's goal. Omit if the plan has no epic.
Plan: link the spec (`<plan-dir>/SPEC.md`) and the companion `index.html` if one exists.
State "Slice N of M".

## What to build

The end-to-end behaviour this issue makes work, from the user's perspective — not
layer-by-layer implementation.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- Each blocking issue by `#number` (real identifiers, since you publish in dependency
  order), or "None — can start immediately".

</issue-template>

GitHub has no native "blocked by" field, so the dependency chain lives in the **Blocked
by** section; a sub-issue link expresses *epic → slice* parentage, not blocking.

Avoid specific file paths or code snippets in the body — they go stale fast. Exception:
if a prototype produced a snippet that encodes a decision more precisely than prose can
(state machine, reducer, schema, type shape), inline it and note briefly that it came
from a prototype. Trim to the decision-rich parts.

After publishing, report the created issues as a table (slice → `#number` → blocked-by).

Work the **frontier** — any issue whose blockers are all done — one issue at a time,
clearing context between issues.

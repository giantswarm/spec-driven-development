---
name: plan
description: End-to-end plan creation, wrapping /grill-me → /to-spec → /ground-truth + /contrarian → /to-website. Starts from a GitHub issue or a rough idea, creates the plan directory, runs the grilling interview, produces the spec, fact-checks it against real upstream source, red-teams its necessity, generates the companion website, and optionally breaks it into issues. Use when the user wants to start a new plan, says "plan this properly", "run the plan process", "new plan from this issue".
---

# plan

Orchestrates the full plan-creation workflow of this repo: take a seed, create the plan
directory, grill the user on it, then — with confirmation at each step — generate the spec,
fact-check it against real source, red-team its necessity, and produce its companion
website. Each stage delegates to an existing skill; this skill only sequences them and
carries the seed context through.

## Workflow

### 1. Get the seed

A plan starts from one of:

- **a GitHub issue** (an epic, a feature request) — a full URL or `org/repo#123`;
- **a rough idea** the user pastes or describes;
- **a file** in this repo or another.

If the user didn't provide one, ask which it is.

### 2. Load it

For a GitHub issue, read its title, body, labels and comments — use a GitHub MCP server if
one is connected, otherwise:

```sh
gh issue view <url-or-number> --repo <org/repo> --json number,title,body,url,labels,comments
```

Keep the issue URL at hand; it goes in the spec. If the issue can't be fetched (bad URL, no
access), report the error and ask for a corrected reference — don't invent issue content.

### 3. Create the plan directory

Derive a short kebab-case name (2–4 words, e.g. `persistent-workspace`,
`token-exchange-teleport`) from the seed. Tell the user the name you chose and create it as
a top-level directory of this repo. One directory per plan: `SPEC.md` and `index.html` both
live there, alongside the `grill-sessions/`, `ground-truth/`, `contrarian/` and `research/`
artifacts produced along the way.

### 4. Grill the user

Invoke the **grill-me** skill, summarizing the seed (title, goal, key constraints) as the
plan under discussion. Run the session to completion — resolve every branch of the design
tree.

### 5. Offer the spec

When grilling is done, ask: *create the spec now with to-spec?*

- **Yes** → invoke **to-spec**. The spec is saved as `<plan-dir>/SPEC.md` — always that exact
  filename — and links the originating issue, if any, in a bold `**Epic:**` line under the
  title.
- **No** → reply: "When you're ready, run `/to-spec` in this conversation (or a session with
  this context). Save the spec as `<plan-dir>/SPEC.md`." Then stop.

### 6. Ground-truth the spec

Ask: *fact-check the plan against real source with ground-truth?* Run this **before** the
website, so any hallucinated APIs, flags or fields are fixed in the spec before it's
rendered.

- **Yes** → invoke **ground-truth** on `<plan-dir>/SPEC.md`. It extracts the plan's external
  claims, verifies them against shallow clones in `<plan-dir>/.agents_source/`, and applies
  the inline fixes the user approves. The spec may change here — that's expected.
- **No** → reply: "When you're ready, run `/ground-truth <plan-dir>/SPEC.md`." Then continue.

### 7. Contrarian gate

Ask: *run the contrarian review?* Strongly recommend yes — it attacks the plan's
**necessity** (is there an existing capability that already covers this?), which
ground-truth's claim-checking does not.

- **Yes** → invoke **contrarian** on the spec. A `SIMPLIFY` or `KILL` verdict blocks the
  workflow: return to grilling, or abandon the plan. Do not proceed to the website with an
  unresolved verdict.
- **No** → reply: "When you're ready, run `/contrarian <plan-dir>/SPEC.md` before rendering
  the website." Then continue.

### 8. Offer the website

Ask: *generate the companion website with to-website?*

- **Yes** → invoke **to-website** on the spec.
- **No** → reply: "When you're ready, run `/to-website <plan-dir>/SPEC.md`." Then stop.

### 9. Wrap up

Commit the plan directory on a branch and open a pull request — the PR is what humans
review. If `main` is protected, say so; if the repo allows direct commits, still prefer a
PR, since reviewing the plan is the point of this repo.

### 10. Offer the issue breakdown (optional)

Ask: *break this plan into issues with to-issues?* Many teams stop at the reviewed spec and
let the implementing agent work from it — only run this if the team tracks work as issues.

### 11. Offer to link the plan on the issue

The **final** action, only if the plan came from an issue and only if the user agrees.

Ask: *post a comment on the issue with a direct link to this plan?*

- **No** → skip silently and stop.
- **Yes** →
  1. Resolve this repo's slug: `gh repo view --json nameWithOwner -q .nameWithOwner`
  2. Build the blob URL to the spec, pointing at `main` — the plan lands there when its PR
     merges, so the link resolves afterwards even though it's posted from the branch:
     `https://github.com/<this-repo>/blob/main/<plan-dir>/SPEC.md`
  3. Comment on the issue:

     ```sh
     gh issue comment <issue-url-or-number> --repo <org/repo> \
       --body "📋 Plan drafted: [<plan-name>](<blob-url>)"
     ```

  Confirm which issue was commented and show the link posted.

## Notes

- Never skip a confirmation: steps 5, 6, 7, 8, 10 and 11 always ask before acting.
- Step 11 is optional and always last — never post the comment mid-workflow, and never
  without an explicit "yes".
- Ground-truth (step 6) runs on the spec and may edit it, always before the website is
  rendered, so `index.html` reflects the corrected plan.
- The spec markdown is the source of truth; `index.html` is always generated from it via
  to-website, never hand-written.

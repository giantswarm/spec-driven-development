# AGENTS.md

Agent-facing notes for working in this repo. Human-facing docs (what this is, the
workflow, the skills) live in [README.md](./README.md) — read that for the big picture.

This repo stores **plans** in a versioned manner: each plan is its own directory holding a
`SPEC.md` and a companion `index.html` website.

## Working in this repo

- **One directory per plan.** A new plan gets its own top-level directory containing the
  spec markdown — always named `SPEC.md`, the source of truth — and a companion `index.html`
  rendered from it. Don't scatter a plan's files across the repo, and don't name the spec
  after the plan slug.

- **Grill logs.** Each `/grill-me` session is logged as its own markdown file in
  `<plan-dir>/grill-sessions/`, capturing every question and answer as it happens.

- **Shared glossary.** `context/glossary.md` is the repo-wide canonical terminology for all
  plans. `/grill-me` reads it to challenge a plan's language and updates it inline as terms
  are resolved. Keep terminology consistent with it.

- **Ground-truth reports.** Each `/ground-truth` run writes a verdict report to
  `<plan-dir>/ground-truth/<YYYY-MM-DD>-<slug>.md`, listing every checked claim, its status,
  and the `file:line` evidence.

- **Research notes.** Each `/research` run saves its cited findings to
  `<plan-dir>/research/<YYYY-MM-DD>-<slug>.md`, sourcing claims from primary sources (and
  the plan's `.agents_source/` clones where code is involved).

- **Contrarian verdicts.** Each `/contrarian` run writes its verdict
  (`BUILD` / `SIMPLIFY` / `KILL`) to `<plan-dir>/contrarian/<YYYY-MM-DD>-<slug>.md`.
  `SIMPLIFY` and `KILL` send the plan back to grilling before any website or issues work.

- **Plans land via pull request.** Commit a plan directory on a branch and open a PR. The
  PR is the review surface — that is the point of this repo.

## Agent skills

The plan-creation workflow is codified in `/plan`, which sequences
`/grill-me` → `/to-spec` → `/ground-truth` + `/contrarian` → `/to-website`, and optionally
`/to-issues`. Skills live in `.agents/skills/`, symlinked into `.claude/skills/` and
`.cursor/skills/`.

Four of them are customized copies from mattpocock/skills. Never reinstall them with the
`skills` CLI or the Claude Code plugin — see
[.agents/skills/UPSTREAM.md](./.agents/skills/UPSTREAM.md) for what was changed and how to
take upstream updates.

## Source code

When a plan needs the source of a related project, shallow-clone it into the plan's
`.agents_source/` folder (gitignored). `/ground-truth` uses those same clones to fact-check
the spec's claims against real code.

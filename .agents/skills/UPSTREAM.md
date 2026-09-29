# Upstream skill provenance — mattpocock/skills

Four skills in this repo started life in
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT, see
[THIRD-PARTY-NOTICES.md](../../THIRD-PARTY-NOTICES.md)) and were customized for this
plan workflow. This file records what came from upstream, what was changed, and how to
take future upstream releases without losing those changes.

> **Don't run `npx skills add mattpocock/skills` in this repo,** and don't install the
> `mattpocock-skills` Claude Code plugin alongside it. Either would give you a second,
> conflicting copy of these skills. Updates are taken by diffing upstream manually and
> porting what's wanted — record every port in the sync log at the bottom.

## Which skills are which

| Skill | Origin | Status |
|---|---|---|
| `grill-me` | upstream `productivity/grilling` | **customized** — see below |
| `to-spec` | upstream `engineering/to-spec` | **customized** — see below |
| `to-issues` | upstream `engineering/to-tickets` | **customized** — see below |
| `research` | upstream `engineering/research` | **customized** — see below |
| `plan` | this repo | native, no upstream |
| `ground-truth` | this repo | native, no upstream |
| `contrarian` | this repo | native, no upstream |
| `to-website` | this repo | native, no upstream |

Synced against upstream **v1.2.3**.

## Local modifications per skill

### grill-me

Upstream splits this in two: `grill-me` is a thin user-invoked wrapper that says "run a
`/grilling` session", and `grilling` holds the interview prompt. We **vendor the
`grilling` prompt directly into `grill-me`** and drop the wrapper, so the repo ships one
skill per job. On top of that:

- **No `disable-model-invocation`**, so the model (and `/plan`) can invoke it.
- **Glossary awareness**: reads `context/glossary.md` at session start, challenges the
  plan's terms against it, updates it inline as terms resolve. `GLOSSARY-FORMAT.md`
  defines the entry format.
- **Per-session Q&A logging**: every session appends live to
  `<plan-dir>/grill-sessions/<YYYY-MM-DD>-<slug>.md`, never batched at the end.
- **Version-currency guard**: never assert a library's current/latest version from
  memory; verify against a primary source or stay version-agnostic, and keep unverified
  versions out of the grill log.
- Description rewritten with concrete trigger phrases for model invocation.

Deliberately **not** adopted: upstream's necessity/capability-inventory questions. The
necessity gate lives entirely in `/contrarian`, sequenced by `/plan`. Don't re-add them
here.

### to-spec

Upstream `to-spec`, same name. Changes:

- **`disable-model-invocation` removed.**
- **Canonical filename**: the spec is always saved as `<plan-dir>/SPEC.md`, never a
  slug-named file. Upstream saves no file at all (it publishes to the issue tracker).
- **Epic link**: a bold `**Epic:**` line under the title when the plan came from an
  issue, which `/to-issues` parses later.
- **Version-currency guard**, matching `grill-me`.
- Dropped the `/setup-matt-pocock-skills` tracker-configuration references — this repo
  has no tracker config; `/to-issues` decides publishing on its own.

### to-issues

Upstream `to-tickets`, kept under the name `to-issues` and narrowed to GitHub. Changes:

- **Issue vocabulary** throughout, and GitHub as the target (MCP server → `gh` CLI →
  local files as a fallback), instead of upstream's configurable tracker.
- **Explicitly optional**: many teams stop at the reviewed spec.
- Dropped the local `.scratch/` tickets mode, the `ready-for-agent` triage label (no
  triage vocabulary in this repo), and the closing `/implement` pointer.
- Kept upstream's substance: tracer-bullet vertical slices, blocking edges per issue,
  slices sized to a single fresh context window, the wide-refactor expand–contract
  exception, and the breakdown quiz.

### research

Upstream `engineering/research`. Changes:

- Findings are saved to `<plan-dir>/research/<YYYY-MM-DD>-<slug>.md`, matching this
  repo's per-plan artifact convention.
- When the question is about a project's code, the agent prefers (or creates) the plan's
  `.agents_source/` shallow clones and cites `file:line` there, so `/ground-truth` can
  verify against the same source later.

## How to take an upstream update

1. Shallow-clone upstream and read its `CHANGELOG.md` for the releases since the last
   sync entry below.
2. Diff each upstream skill against our copy. Watch the renames: our `grill-me` diffs
   against upstream **`grilling`**, `to-spec` against **`to-spec`**, `to-issues` against
   **`to-tickets`**.
3. Port what's wanted by hand, preserving the local modifications listed above.
4. Append an entry to the sync log — including what you chose *not* to adopt, and why.

The easiest way to do all of this: ask the agent to "check for a new mattpocock/skills
release and sync per UPSTREAM.md".

## Sync log

- **2026-09-22** — initial vendoring for this template, against upstream **v1.2.3**.
  `grill-me` carries the current `grilling` prompt (rounds + frontier model, sub-agent
  fact-finding). `to-spec` carries `to-spec` v1.2.3. `to-issues` carries the `to-tickets`
  content, re-scoped to GitHub. `research` carries the current upstream skill. Not
  adopted: the `setup-matt-pocock-skills` configuration flow, and the `implement`,
  `tdd`, `code-review`, `wayfinder`, `prototype`, `diagnosing-bugs`,
  `improve-codebase-architecture` and `grill-with-docs` skills — they serve
  implementation repos rather than this planning repo, or overlap with `/plan`.
  Naming follows upstream where it can: `grill-me`, `to-spec` and `research` match his
  names, and the plan artifact is `SPEC.md` to match. `to-issues` keeps the older name
  because it is re-scoped to GitHub issues rather than upstream's configurable tracker.

---
name: ground-truth
description: Fact-check a finished spec against the real upstream source code, catching hallucinated APIs, features, flags, fields, and integration points before any work starts. Reads a plan/spec, extracts every claim that asserts something about existing external code, verifies each in parallel against shallow clones in the plan's .agents_source/, then reports a per-claim verdict with file:line evidence and offers to fix the spec inline. Use after the spec is created (after /to-spec, before /to-website and /to-issues), or when the user says "ground-truth this", "did we hallucinate anything", "check this against the real code", or wants to verify a plan is grounded in reality.
---

# ground-truth

Run **after the spec is created** (after `/to-spec`), before rendering the website or
turning the plan into issues or code. The job is
to catch the hallucinations: every place the plan assumes an API, function, flag,
config field, CRD property, feature, or integration point *that already exists in
some upstream project* — and to confirm it actually does, in the real source.

This is a contrarian check. Assume each claim is wrong until the source proves it
right. A plan that reads plausibly is exactly the plan that hides a hallucinated
method call.

## Workflow

### 1. Locate the plan

Take the plan directory or spec markdown the user names; otherwise default to the
plan under discussion. The spec is `<plan-dir>/SPEC.md` — the source of truth — read
it fully.

### 2. Extract verifiable claims

Walk the plan and pull out **only claims that assert something about existing
external code or behavior** — the kind that can be true or false against a
repository. Record each as `{claim, upstream project, what to check, plan location (file:line)}`.

Claims worth checking:

- **API / function / method** — "we call `Foo.Bar()`", "the client exposes `X`"
- **Flag / CLI option / env var** — "pass `--enable-Y`", "set `Z=true`"
- **Config / values field** — "set `spec.foo.bar` in the CR / `values.yaml`"
- **CRD / type / field** — "the `Cluster` CRD has a `status.ready` field"
- **Feature / capability** — "project P supports webhooks / hooks / plugins"
- **Extension / integration point** — "we hook into P's reconcile loop / plugin API"
- **Version / behavior** — "as of vX, P does Q"
- **Version currency** — "the latest / current version of P is N", "P is on vN"
  (for any library, tool, or component). These are moving-target facts LLMs routinely
  get wrong from stale memory, so treat every one as suspect. Note: a currency claim
  is **not** verifiable against a shallow
  clone (the clone is pinned to one SHA and says nothing about what the newest release
  is) — verify it against a **primary source**: the package registry
  (npm/PyPI/crates/etc.) or the project's GitHub releases/tags. A plan that pins a
  version it doesn't need should instead be version-agnostic ("latest stable at build
  time"); flag the pin even if the number happens to be current today.

**Do NOT check** the plan's own design decisions, new code we intend to write, or
aspirational goals — those aren't verifiable against upstream and aren't
hallucinations. Only claims about what *already exists* elsewhere.

### 3. Ensure the sources are present

For every upstream project referenced, ensure a **shallow clone** exists at
`<plan-dir>/.agents_source/<project>/` (repo convention). Clone what's missing:

```sh
git clone --depth 1 <repo-url> <plan-dir>/.agents_source/<project>
```

Record each clone's ref and commit SHA — the verdict is only valid against a known
revision, and it defuses "outdated" ambiguity. If a claim names a specific version,
check out that ref for that clone.

### 4. Fan out — one verifier per claim

Spawn verification subagents **concurrently** (Agent tool, multiple calls in one
message). One subagent per claim keeps each verdict independent; claims against the
same repo simply share the clone on disk.

**Version-currency claims** have no clone to check against — dispatch each to a
subagent that verifies it against a primary source (the package registry or the
project's GitHub releases/tags) using WebFetch/WebSearch, and cite the source URL and
the date observed as its evidence. All other claims get the clone-based brief below.

Give each subagent: the claim, the plan location, the clone path + commit SHA, and
this contrarian brief:

> Try to **disprove** this claim against the source at `<path>`. Search the real
> code — do not rely on prior knowledge. Return a verdict with concrete evidence:
> - **VERIFIED** — found it. Cite `file:line` for the definition/usage.
> - **NOT-FOUND** — no such API/flag/field/feature exists in this revision. Say
>   what you searched and what the nearest real thing is, if any.
> - **MISMATCH** — something similar exists but differs (wrong name, signature,
>   type, location, semantics). Cite the real one at `file:line`.
> - **UNCERTAIN** — can't confirm either way; state what's blocking (missing repo,
>   generated code, needs runtime). Never guess VERIFIED.

### 5. Collate the report

Write the report to `<plan-dir>/ground-truth/<YYYY-MM-DD>-<short-slug>.md`:
a header (date, plan, clones + commit SHAs checked), then one row per claim —
verdict, the claim, plan location, and the evidence `file:line` (or the gap).
Lead the summary with the count of NOT-FOUND / MISMATCH — those are the
hallucinations.

### 6. Offer inline fixes

Present the verdicts. For every **NOT-FOUND / MISMATCH / UNCERTAIN**, offer to
correct the plan in place (AskUserQuestion) — pick per claim:

- **Fix** — replace the hallucinated claim with the real API/flag/field (with its
  `file:line`).
- **Caveat** — mark it as an assumption to validate, if it can't be resolved now.
- **Drop** — remove the claim / dependent decision.
- **Keep** — the user asserts it's correct despite the source (note why).

Apply the chosen edits to the spec markdown, then note in the report which claims
were fixed. Leave VERIFIED claims untouched.

## Notes

- Keep terminology aligned with `context/glossary.md`.
- Never mark a claim VERIFIED on memory alone — evidence is `file:line` in a clone,
  or it doesn't count.
- If no upstream is named for a claim, that itself is a finding: an external
  assertion with nothing to check it against.

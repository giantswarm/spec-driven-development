---
name: contrarian
description: Adversarial review of a finished spec — argue the plan is unnecessary, find the simplest path using only existing capabilities, and demand a written justification for every new subsystem. Runs after /to-spec, alongside /ground-truth, before /to-website. Use when the user says "contrarian", "red-team this plan", "try to kill this plan", or as the plan gate.
---

# contrarian

Ground-truth checks whether the plan's claims are *true*. This skill checks
whether the plan is *necessary*. A plan can be 100% verified and still wrong —
every claim accurate, every diagram correct, all of it building machinery that
an existing capability already provides. The reviewer who kills that plan in
two review comments is this skill's role model; run it before a human has to.

## Workflow

### 1. Locate the plan

Take the plan directory or spec markdown the user names; otherwise the plan
under discussion. The spec is `<plan-dir>/SPEC.md`. Read the spec fully, plus the
grill log and glossary for context.

### 2. Build the kill brief

Work through these attacks in order, as an adversary whose stated goal is
that the plan should not be built:

1. **Necessity.** For the core problem, search the code in the plan's
   `.agents_source/` clones for existing mechanisms that already solve it —
   re-derive the answer independently rather than trusting the spec's version.
2. **Simplest path.** Construct the cheapest solution that uses only existing
   capabilities plus configuration. Compare it honestly against the spec's
   proposal: what does the spec buy that the simple path does not, and is that
   delta actually required by the epic?
3. **Complexity budget.** Enumerate every **new subsystem** the plan
   introduces — a new trust class, a new store, a new protocol leg, a new
   service, a new CRD/API surface. For each, the spec must contain (or you
   demand) a written justification for why nothing existing suffices. New
   subsystems without such a justification are findings, not style points.
4. **Reviewer simulation.** Name the objection the most knowledgeable human
   reviewer of the touched components would raise, in one or two sentences,
   as they would phrase it.

### 3. Write the verdict

Write to `<plan-dir>/contrarian/<YYYY-MM-DD>-<short-slug>.md`:

- **Verdict**: `BUILD` (necessity holds), `SIMPLIFY` (a cheaper existing-path
  variant should replace part of the design — say which part), or `KILL`
  (an existing mechanism covers the problem — cite it at `file:line`).
- The simplest-path comparison (short table: Spec proposal vs existing-path
  alternative, with the delta and whether the epic requires it).
- The new-subsystem list with per-subsystem justification status.
- The simulated reviewer objection.

### 4. Present and gate

Present the verdict. `SIMPLIFY` and `KILL` block the workflow: the spec goes
back to grilling (or is abandoned) before `/to-website` and `/to-issues` run.
Do not soften a `KILL` into a `SIMPLIFY` to keep the plan alive — the
cheapest failure is here, on paper.

## Notes

- Independence matters: do not reuse the spec's own framing or the grill log's
  conclusions as evidence — re-derive from the sources.
- Keep terminology aligned with `context/glossary.md`.
- This skill does not verify factual claims (`/ground-truth` does); it only
  attacks necessity and proportionality. Run both; they catch disjoint
  failure modes.

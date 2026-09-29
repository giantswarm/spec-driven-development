# context/glossary.md Format

A single, repo-wide glossary shared across all plans in this repo. One file, at
`context/glossary.md`.

## Structure

```md
# Glossary

The canonical terminology for this repo's plans. Add a term when it is resolved
during a grilling session.

## Language

**Plan**:
A single unit of planned work, living in its own top-level directory (spec markdown + companion website).
_Avoid_: Project, spec

**Spec**:
The markdown document that is the source of truth for a plan.
_Avoid_: Doc, writeup

**Epic**:
The GitHub issue a plan is created from and linked back to.
_Avoid_: Ticket, story

## Relationships

- A **Plan** is derived from exactly one **Epic**
- A **Plan** contains one **Spec** and one companion website

## Flagged ambiguities

- "plan" was used to mean both the **Spec** document and the whole plan directory — resolved: these are distinct.
```

## Rules

- **Be opinionated.** When multiple words exist for the same concept, pick the best one and list the others as aliases to avoid.
- **Flag conflicts explicitly.** If a term is used ambiguously, call it out in "Flagged ambiguities" with a clear resolution.
- **Keep definitions tight.** One sentence max. Define what it IS, not what it does.
- **Show relationships.** Use bold term names and express cardinality where obvious.
- **Only include terms specific to this project's domain.** General programming or planning concepts don't belong even if used often. Before adding a term, ask: is this unique to this repo's plans, or a general concept? Only the former belongs.
- **Group terms under subheadings** when natural clusters emerge. A flat list is fine when all terms belong to one cohesive area.

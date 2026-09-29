---
name: grill-me
description: Grill the user relentlessly about a plan or design, walking every branch of the design tree, challenging the plan's terms against the shared glossary, and logging every question and answer to a per-session file. Use when the user wants to stress-test a plan before building, or uses any 'grill' trigger phrase.
---

# grill-me

<what-to-do>

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask *now* without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a *later* round, not this one.

Finding *facts* is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, the web), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The *decisions* are the user's: put each to them and wait.

Never state a library's or component's *current* or *latest* version from memory. Your training is stale, and moving-target facts like "the latest major is N" are exactly what you get wrong. If a version is relevant, either verify it against a primary source (the registry — npm/PyPI/crates.io — or the project's own releases) before asserting it, or keep it version-agnostic ("the latest stable release at build time"). Do not let an unverified version number enter the grill log.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on the plan until the user confirms you have reached a shared understanding.

</what-to-do>

<supporting-info>

## Glossary awareness

This repo keeps a single, shared glossary at `context/glossary.md` — the canonical
terminology for every plan in it. Read it at the start of a grilling session so you
can hold the plan to the language already established. If it is missing, recreate it
the first time a term is resolved.

## Log the session

Every grilling session is recorded as its own markdown file inside a
`grill-sessions/` folder within the plan being grilled:
`<plan-dir>/grill-sessions/<YYYY-MM-DD>-<short-topic-slug>.md`.

At the start of the session, create the file with a short header (date, plan, and the
topic being grilled). Then append each question and its answer **as they happen** — one
entry per exchange — so the log survives even if the session is interrupted. Never
batch the whole transcript at the end.

Use this entry format:

```md
# Grill session — <topic>

- **Date:** <YYYY-MM-DD>
- **Plan:** <plan-dir>

## Q1: <the question>

<the user's answer, and any resolution or decision reached>

## Q2: <the question>

<the user's answer…>
```

Record the question as asked and the user's actual answer, not a paraphrase that loses
the decision. When a question resolves a glossary term or a trade-off, note the
resolution in that entry.

## During the session

### Challenge against the glossary

When the user uses a term that conflicts with an existing definition in
`context/glossary.md`, call it out immediately. "The glossary defines 'cluster' as X,
but you seem to mean Y — which is it?"

### Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term.
"You're saying 'plan' — do you mean the spec document or the whole plan directory?
Those are different things."

### Discuss concrete scenarios

When relationships between concepts are being discussed, stress-test them with specific
scenarios that probe edge cases and force the user to be precise about the boundaries
between concepts.

### Update the glossary inline

When a term is resolved, update `context/glossary.md` right there. Don't batch these up
— capture them as they happen. Use the format in [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md).

Only include terms meaningful to someone reasoning about the plan's domain. Don't couple
the glossary to implementation details, and skip general concepts that aren't specific to
this project.

</supporting-info>

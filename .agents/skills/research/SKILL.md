---
name: research
description: Investigate a question against high-trust primary sources and capture the findings as a cited Markdown file in the plan directory. Use when the user wants a topic researched for a plan, docs or API facts gathered, or reading legwork delegated to a background agent.
---

# research

Spin up a **background agent** to do the research, so you keep working while it reads.

Its job:

1. Investigate the question against **primary sources** (official docs, source code,
   specs, first-party APIs), not a secondary write-up of them. Follow every claim back to
   the source that owns it.
2. Write the findings to a single Markdown file, citing each claim's source.
3. Save it to `<plan-dir>/research/<YYYY-MM-DD>-<slug>.md`, matching this repo's per-plan
   artifact convention (`grill-sessions/`, `ground-truth/`, `contrarian/`, `research/`).
   If the question isn't tied to a plan, put it somewhere sensible and say where.

When the question is about a project's code, prefer the plan's `.agents_source/` shallow
clones — create one if it isn't there yet — and cite `file:line` against them, so
`/ground-truth` can verify against the same source later.

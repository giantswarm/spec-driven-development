# Component reference

The catalog of building blocks in `base.html`. Pick the component that matches the *shape* of
the content, not just its topic. All components are already styled — you only write the markup.

## Design tokens (CSS variables)

Defined twice in `:root`: light values, then overridden under `html[data-theme="dark"]`. Never
hardcode colors in markup — reference these so dark mode keeps working.

| Token | Role |
|-------|------|
| `--ink` | Headings, strongest text |
| `--body` | Body text |
| `--muted` / `--faint` | Secondary / tertiary text, captions, meta |
| `--paper` / `--paper-2` | Page bg / raised surfaces (tables, callouts, diagrams) |
| `--line` / `--line-2` | Hairlines / stronger borders |
| `--teal` / `--teal-bg` | Primary accent: links, durable/positive, active toggle |
| `--amber` / `--amber-bg` | Warning accent: ephemeral/loss, out-of-scope, notes |
| `--code-bg` / `--code-fg` | Code & pathbox foreground/background (dark in both themes) |
| `--measure` | `70ch` — max line length for readable prose |

The accent semantics are a convention worth keeping: **teal = durable/kept/in-scope**,
**amber = ephemeral/lost/out-of-scope/caveat**.

## Document structure

- **`header`** — one per page. `.kicker` (optional uppercase mono eyebrow), `<h1>`, and a
  `.meta` row of `<span><b>Label</b> value</span>` pairs. The meta row holds **only** the `Plan`
  link and the `Tracking` issue — no other fields. Fold scope/status/size into a section or the
  abstract, not the meta.
- **`.abstract`** — optional teal-bordered framing block right after the header, before the
  first section. One or two paragraphs. Delete if the first section already frames things.
- **`section`** — each numbered chapter. Always `<section><div class="wrap">…`. Heading is
  `<h2><span class="no">N</span> Title</h2>`. Renumber consecutively when adding/removing.
- **`.lead`** — a one-line muted summary directly under an `<h2>`.
- **`footer`** — one per page, mono, faint. Holds a one-line summary + issue link, and a
  `Generated <date> <hour>` line (the `GENERATED_AT` token) stamped at generation time.

## Content components

### Prose
Plain `<p>` (capped at `--measure`). Use `<code>` for identifiers, paths, fields, env vars.

### `.pathbox`
A single highlighted monospace line on a dark background — for a key path, command, or fact.
Spans: `.tmp` (teal, e.g. a variable), `.sid` (amber, e.g. an id placeholder), `.cm` (grey comment).
```html
<div class="pathbox"><span class="tmp">$VAR</span>/path/<span class="sid">{id}</span><span class="cm">  # note</span></div>
```

### `.deftable` (in `.tablescroll`)
The workhorse table for **decisions, fields, options, comparisons**. First column is the key
(add `class="key"` for mono+teal). Wrap each cell's prose in `<p>` so spacing is consistent.
Always wrap the table in `<div class="tablescroll">` so it scrolls on mobile instead of
breaking the layout.
```html
<div class="tablescroll">
  <table class="deftable">
    <thead><tr><th>Decision</th><th>Rationale</th></tr></thead>
    <tbody>
      <tr><td class="key">choice</td><td><p>why</p></td></tr>
    </tbody>
  </table>
</div>
```

### Code block (`figure > .codewrap > pre`)
For manifests / source. `.codebar` is the filename label. Inside `<pre>`, hand-apply highlight
spans (no syntax highlighter runs): `.c` comment, `.k` key, `.v` value, `.hi` highlight, `.d`
divider (`---`). Keep `<pre>` content flush-left — its whitespace is literal.

### Mermaid diagram (`figure > .codewrap > .mermaid-host > .mermaid-src`)
See SKILL.md → "Diagrams". `flowchart` for structure, `sequenceDiagram` for swimlanes. The
source lives in a hidden `.mermaid-src`; the page script renders + themes it. Optional
`<figcaption>` below.

### `.scope` (two-column in/out)
A grid with an `.in` column (teal heading "In scope") and `.out` column (amber heading "Out of
scope"). Each is a `<ul>` of `<li><b>Item</b> — detail.</li>`. Collapses to one column on mobile.

### `ol.steps`
Numbered, circled steps for procedures / acceptance criteria. Bold the key outcome of each step.

### `.note`
Amber callout for a caveat, gotcha, or important aside. Lead with `<b>Label.</b>`.

### `ul.plain`
A simple bulleted list (operational notes, misc). `<li><b>Topic.</b> Detail.</li>`.

## Behavior (already wired in `base.html`)

- **Theme toggle** — floating `.themebar` bottom-right with ☀/☾ buttons. Defaults to **light**,
  persists choice in `localStorage`, sets `data-theme="dark"` on `<html>`, and re-renders
  Mermaid on switch. The active button shows `aria-pressed="true"` (teal highlight).
- **Mermaid auto-render** — discovers all `.mermaid-src`, dedents, renders into `.mermaid-host`.

## Layout invariants — keep these true

- Every section's content sits inside `<div class="wrap">` (880px max, responsive padding).
- Wide content (tables, code, diagrams) lives in an `overflow-x: auto` container so the page
  body never scrolls horizontally.
- Prose stays within `--measure`. Don't widen `<p>`.
- Colors come from tokens, so both themes stay correct. Verify in both before finishing.

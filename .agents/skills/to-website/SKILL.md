---
name: to-website
description: Generate a single-file HTML website for a plan/spec in this repo — numbered sections, light/dark toggle, and themed Mermaid diagrams. Use when a plan needs an index.html, when asked to "make a website/page for this plan", "render this spec as a site", or to add/fix diagrams on an existing plan page.
---

# to-website

Turns a plan or spec markdown file into a polished, self-contained `index.html` that sits
next to it in the plan's directory. One HTML file, no build step, no external assets except
the Mermaid library (loaded from a CDN). Reference implementation: `example-plan/index.html`.

## When to use

- A plan directory has a `SPEC.md` and needs a companion `index.html`.
- Asked to "make a website / page / site for this plan", "render this as a webpage".
- Adding or fixing Mermaid diagrams, or the dark/light toggle, on an existing plan page.

## Workflow

1. **Read the plan markdown** in the target directory (`<plan-dir>/SPEC.md`). Identify its
   sections, key decisions, code/manifests, scope, verification steps, and caveats.
2. **Copy the template** to the plan directory as `index.html`:
   `cp .agents/skills/to-website/base.html <plan-dir>/index.html`
   (resolve `base.html` relative to this skill's directory).
3. **Fill the placeholders** — replace every ALL-CAPS token:
   - `PLAN TITLE` → the plan's title (appears in `<title>`, `<h1>`, and footer).
   - `PLAN_MARKDOWN_URL` / `PLAN_FILE.md` → the GitHub blob URL of the source markdown and
     its filename. The spec filename is always `SPEC.md`, so `PLAN_FILE.md` renders as
     `SPEC.md`. Resolve this repo's slug rather than hard-coding it:

     ```sh
     gh repo view --json nameWithOwner -q .nameWithOwner   # e.g. acme/team-plans
     ```

     URL form: `https://github.com/<this-repo>/blob/main/<plan-dir>/SPEC.md`.
   - `ISSUE_URL` / `org#NNNNN` → the originating issue (omit the `Tracking` span and footer
     clause if there is no issue).
   - `GENERATED_AT` → the generation date:hour. Run `date "+%Y-%m-%d %H:%M %Z"` and paste
     the result. Re-stamp it whenever you regenerate the page.

   The header `.meta` row holds **only** the `Plan` link and the `Tracking` issue — nothing
   else. Do not add bespoke meta fields (scope, status, size, …); fold that content into a
   section or the abstract instead.
4. **Build the sections from the plan markdown.** The template ships **one** example
   `<section>` showing the structure only — replicate that pattern, one `<section>` per
   chapter, and number them consecutively with `<span class="no">N</span>`. Drive the
   section list from the source markdown's own headings (see "Page structure" below), not
   from the template. Pick the right component per content type — see `reference.md` for the
   full catalog.
5. **Add diagrams where they earn their place** (see below). They are **not** required in
   any particular section. Reach for one when a section has **many interacting components**
   (a flowchart of the architecture/data flow) or describes a **multi-step or cross-actor
   process** (a `sequenceDiagram` swimlane). Skip diagrams for prose, tables, or single-fact
   sections — a gratuitous diagram is noise. A page may have several diagrams or none.
6. **Verify** it opens in a browser. Toggle dark/light, confirm every diagram renders in
   both themes and the page never scrolls horizontally on a phone width.

Do not invent facts. Everything on the page must be grounded in the plan markdown; if the
markdown is thin, ask before embellishing.

## Page structure

The section list comes from the **plan markdown's own headings**, not from a fixed mould —
mirror its structure and wording. Most specs map onto the skeleton below; use it as a
checklist for what's usually present, skip any the source doesn't cover, and merge or split
to match the source. Each section is a numbered `<section>`; the component column is the
*default* fit, not an obligation.

| Typical section | Default component(s) |
|-----------------|----------------------|
| Problem / context | prose, optional `.pathbox` for a key path or fact |
| Solution / approach | prose + a `.deftable` of the moving parts |
| Key decisions | `.deftable` (decision → rationale) |
| Reference manifest / code | code block (`figure > .codewrap > pre`) |
| Alternatives | prose + `.deftable`, optional `.note` for caveats |
| Scope | `.scope` two-column in/out |
| Verification / acceptance | `ol.steps`, optional `.note` |
| Operational notes | `ul.plain` |

Keep prose lean and let the component carry the structure. A section that is a plain list of
facts does not need a diagram, a table, *and* a callout — pick the one that fits.

## Diagrams (Mermaid)

Diagrams are **data-driven from the HTML** — you never touch the script. To add one, drop a
`figure` whose `.mermaid-host` contains a hidden `.mermaid-src` with the Mermaid source:

```html
<figure>
  <div class="codewrap">
    <div class="codebar">diagram title</div>
    <div class="mermaid-host">
      <script type="text/plain" class="mermaid-src">
flowchart TD
  a["Start"] --> b["End"]
      </script>
    </div>
  </div>
  <figcaption>Caption.</figcaption>
</figure>
```

The bottom-of-page script auto-discovers every `.mermaid-src`, strips common indentation, and
renders it into its host. It re-renders on theme toggle using Mermaid's `neutral` theme (light)
and `dark` theme (dark). Rules:

- **When to add one** — only when it earns its place: `flowchart TD/LR` when a section has many
  interacting components (architecture/data flow); `sequenceDiagram` when it describes a
  multi-step or cross-actor process (swimlane). For plain prose, tables, or single facts, leave
  the diagram out.
- Put the source in `<script type="text/plain" class="mermaid-src">` — never a bare
  `<div class="mermaid">` (that bypasses theming and the re-render).
- Wrap node labels containing spaces/slashes/braces in `"..."`; use `<br/>` for line breaks.
- Keep labels short — long labels overflow on mobile.

## CSP caveat

The template loads Mermaid from `cdn.jsdelivr.net`. This works when the file is opened in a
browser and when it is served from GitHub Pages. If the page must be published somewhere with
a strict Content-Security-Policy that blocks the CDN, either inline the Mermaid library or
pre-render each diagram to a static inline SVG and drop the `<script type="module">` block.
Default to the CDN approach.

## Files

- `base.html` — the template to copy. Self-contained: CSS design system, light/dark toggle
  (defaults to light, persisted in `localStorage`), the Mermaid auto-render machinery, and a
  single example `<section>` showing structure only — it deliberately does **not** pre-build a
  section list, so it won't bias the render. Build sections per "Page structure" above.
- `reference.md` — catalog of every CSS component (when to use each, with snippets) and the
  design tokens. Read it when choosing how to render a piece of content.

The design system in `base.html` is deliberately brand-neutral. To put your own brand on plan
pages, change the design tokens at the top of its `<style>` block — that is the only file to
touch.

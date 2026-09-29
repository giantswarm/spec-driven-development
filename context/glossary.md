# Glossary

The canonical terminology for the plans in this repo, shared across every plan. Add or
sharpen a term whenever it is resolved during a `/grill-me` session. Format and rules:
`.agents/skills/grill-me/GLOSSARY-FORMAT.md`.

Seed this with the three or four terms your team argues about most, then let `/grill-me`
grow it.

## Language

### Plan

A single unit of planned work, living in its own top-level directory: a `SPEC.md`, a
companion `index.html`, and the dated artifacts produced while making it.

### Spec

The plan's source of truth, always `<plan-dir>/SPEC.md`. Everything else in the plan
directory is either an input to it or generated from it.

### Website

The `index.html` rendered from a spec by `/to-website`. Generated, never hand-written.

## Example plan terms

Added by the grilling in `example-plan/`. Delete this section together with `example-plan/`.

### Install package

The one versioned chart an operator installs to get the whole product, producing exactly
one Helm release.
_Avoid_: bundle, umbrella, distribution

### Runtime prerequisite

Something that must already be running in the cluster for the product to work after an
install that succeeded without it; hidden when the install neither provides, checks nor
documents it.
_Avoid_: dependency (reserved for Helm chart dependencies), requirement

# Contrarian verdict: one install package for Pintail

- **Date:** 2026-09-22 (first pass), 2026-09-23 (after rework)
- **Plan:** example-plan
- **Verdict (first pass):** SIMPLIFY
- **Verdict (after rework):** BUILD
- **Stance:** the plan should not be built. Re-derived from the clones in `.agents_source/`,
  not from the spec's framing or the grill log.
- **Sources: illustrative.** Pintail and its repositories are fictional; the `file:line`
  references are consistent with the ground-truth report in this example but point at
  nothing real.

## First pass (2026-09-22): SIMPLIFY

The first draft proposed an umbrella chart `pintail` over the three existing charts, a
pre-install hook that orders and upgrades the CRDs, a `pintailctl install` wrapper with
preflight checks, and a self-signed certificate mode in the server. It justified them with
five problems. Two of those five no longer exist in the current code, and a third only
exists because of them.

### 1. Necessity: what already covers the problem

| Problem the draft states | Existing mechanism | Evidence |
|---|---|---|
| Three charts must be installed in a fixed order | Only true because of the next two rows | follows from rows 2 and 3 |
| CRDs need a manual `kubectl apply` before every upgrade | **Already solved.** The server applies its embedded CRDs at startup, on by default. The `pintail-crds` README that the grill relied on is stale. | `pintail-server/cmd/pintail-server/flags.go:42`; `pintail-server/internal/crds/manage.go:30-58`; stale doc at `pintail-crds/README.md:14-22` |
| The console is a separate chart with its own Service and route | **Already solved.** The console is embedded in the server and served behind one chart value. The console chart's own README says so. | `pintail-server/internal/console/embed.go:12`; `pintail-server/charts/pintail-server/values.yaml:88`; `pintail-console/README.md:3` |
| The webhook silently depends on cert-manager | **Nothing covers it.** The chart offers `cert-manager` or a user-supplied Secret, and skips the `Certificate` silently when the API is absent. | `pintail-server/charts/pintail-server/values.yaml:131-140`; `templates/certificate.yaml:1` |
| The install guide does not say what to install | **Nothing covers it.** The guide lists three charts and never names cert-manager. | `pintail-server/docs/install.md:10-24` |

Two of five problems are real. The plan should build for those two and nothing else.

### 2. Simplest path using only existing capabilities

**The simplest path is documentation plus one small component:** install `pintail-server`
alone with `console.enabled: true`, let the server manage its CRDs as it already does, and
add the one thing that does not exist, a self-signed source for the webhook certificate.

| Spec proposal (draft) | Existing-path alternative | Delta the spec buys | Epic requires it? |
|---|---|---|---|
| Umbrella chart `pintail` over three charts | `pintail-server` is already a complete chart once the console is on | A third release shape, a version matrix across four charts, a new release pipeline | **No.** The epic asks for one install, which one existing chart already is. |
| CRDs in the umbrella's `crds/` plus a pre-install hook to upgrade them | `--manage-crds`, on by default | A second CRD writer racing the server's own apply | **No.** Worse than nothing: two field managers on one CRD. |
| `pintailctl install` wrapper with preflight checks | `helm install`; a render-time `fail` for the one real precondition | A second install surface to version, document and support | **No.** The epic says "one command", and the wrapper is a second one. |
| Self-signed certificate mode in the server | None exists | Removes the hidden runtime prerequisite | **Yes.** The epic's acceptance criterion is a working first `FlagSet` on a bare cluster. |
| Install guide rewrite | The guide exists but is wrong | Operators stop hitting the stale path | **Yes.** |

### 3. Complexity budget: new subsystems

| New thing proposed | Justification in the draft | Status |
|---|---|---|
| Umbrella chart `pintail` | "One package" (grill Q7) | **Unjustified.** Drop; `pintail-server` is the install package. |
| CRD pre-install hook | Manual CRD step (grill Q6) | **Unjustified.** The problem it solves is already solved in the server. Drop. |
| `pintailctl install` wrapper | Preflight for missing prerequisites | **Unjustified.** The only missing prerequisite disappears with the certificate mode; a render-time `fail` covers the opt-in `cert-manager` source. Drop. |
| Self-signed certificate module in the server | Hidden runtime prerequisite (grill Q3, Q4) | **Justified.** Nothing existing issues a certificate without cert-manager, and bundling cert-manager was rejected with reasons. |
| Retiring `pintail-crds` and `pintail-console` | Not addressed in the draft | **Justification demanded.** Once the server is the install package, what happens to existing installs of the other two? `pintail-crds` renders the CRD from `templates/` (`pintail-crds/charts/pintail-crds/templates/flagset-crd.yaml:1`): uninstalling it deletes every `FlagSet`. |

### 4. Simulated reviewer objection

The maintainer of the server: *"You are building an umbrella around a console chart I
deprecated in my own README and a CRD chart my server made redundant. Turn the console on
and write the certificate code; the rest is a migration note."*

### Disposition

SIMPLIFY blocks the workflow. Back to grilling with: drop the umbrella, the CRD hook and the
wrapper; make `pintail-server` the install package; answer the retirement question for
the two old charts.

## After rework (2026-09-23): BUILD

Grill session `2026-09-23-rework-after-contrarian.md` adopted the simplification (Q8),
answered the retirement question with a migration that keeps every `FlagSet` (Q9), and
fixed the test gate at one seam (Q10). Re-reviewed against the same clones.

### Simplest path, second pass

The reworked spec **is** the simplest path from the first pass, plus the migration it was
missing. There is no cheaper existing-path variant left: doing nothing leaves the hidden
runtime prerequisite in place, and every remaining piece is either configuration or the one
component that does not exist.

| Spec proposal (reworked) | Existing-path alternative | Delta | Epic requires it? |
|---|---|---|---|
| `pintail-server` as install package, `console.enabled` default `true` | Same chart, value set by hand | One default flip, so the one command needs no values | Yes (one command) |
| `webhook.certificate.source: self-signed`, new default | None | Removes the hidden runtime prerequisite | Yes |
| Loud render failure for `source: cert-manager` without the API | Silent skip today | Turns a runtime failure into an install failure | Yes (a green install means a working product) |
| Final releases of `pintail-crds` (keep policy) and `pintail-console` (notice) | "Never uninstall the CRD chart" in the docs | Migration without data loss, no zombie release | Yes (existing users are in the epic) |

### New things and their justification

| New thing | Status |
|---|---|
| Self-signed certificate module (generate once into a Secret, create-if-absent across replicas, renew with a resource-version-conditioned update, inject the CA into the chart's own webhook configuration by name) | **Kept, justified.** The one missing capability. RBAC is scoped to one Secret and one webhook configuration by name. |
| `console.enabled` default flip | **Kept.** Configuration only, no new component. |
| Render-time guard for the `cert-manager` source | **Kept, justified.** Cheap, render-only, no runtime component. |
| Final releases of the two retiring charts | **Kept, justified.** Without the keep policy, the documented migration deletes every `FlagSet`. |
| Three acceptance scenarios in the existing end-to-end suite | **Kept, justified.** They are the definition of done and they extend an existing harness. |
| Umbrella chart, CRD hook, `pintailctl install` | **Dropped** (first pass). |
| **Not introduced:** a new chart, controller, CRD, CLI or trust class | The self-signed CA is trusted only by the one webhook configuration it is injected into. |

### Simulated reviewer objection, second pass

The security reviewer: *"A server that patches webhook configurations is a server that can
disable validation cluster-wide."* Answered in the spec: the ClusterRole is scoped by
`resourceNames` to the chart's own configuration, and it is not rendered at all in the
`cert-manager` and `secret` modes.

### Verdict: BUILD

Necessity holds for everything that remains. The plan may proceed to `/to-website`.

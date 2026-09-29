# One install package for Pintail

**Epic:** [acme/pintail#412](https://github.com/acme/pintail/issues/412)

> **This is a worked example shipped with the template.** Pintail, its repositories, its
> issues and every file path cited in this directory are fictional, invented to show the
> workflow on a realistic problem. Nothing here refers to a real product. Read the
> artifacts in order: `grill-sessions/`, the contrarian verdict (first pass SIMPLIFY), the
> second grill session, the ground-truth report, then this spec. Delete the directory once
> you have real plans.

## Problem Statement

Pintail is a self-hosted feature-flag service for Kubernetes. Today an operator who wants
the whole product installs three Helm charts in a fixed order (`pintail-crds`, then
`pintail-server`, then `pintail-console`) and follows a troubleshooting page for anything
that goes wrong.

The install reports success even when the product does not work. The server's admission
webhook validates every `FlagSet` and fails closed, and its TLS certificate comes from
cert-manager. On a cluster without cert-manager the chart silently skips the certificate,
the webhook listener never starts, and the first `kubectl apply` of a `FlagSet` fails with
a webhook connection error. cert-manager is a **hidden runtime prerequisite**: the install
neither provides it nor checks for it, and the install guide does not name it.

The operator's experience: three charts, one undocumented prerequisite, and a product that
breaks on its first real use rather than at install time.

## Solution

`pintail-server` becomes the **install package**: the one chart an operator installs to get
the whole product. One `helm install` on a cluster that has nothing but Kubernetes yields a
running server, a working admission webhook and the web console.

- The webhook gets its certificate without cert-manager. The server generates a
  self-signed certificate on first start, stores it in a Secret and injects the CA into its
  own webhook configuration. cert-manager stays supported as an opt-in for operators who
  already run it.
- The console, already embedded in the server binary, is switched on by default.
- The server already applies its own CRDs at startup, so the separate `pintail-crds` chart
  and the manual CRD step are retired.
- `pintail-crds` and `pintail-console` get one final release each that makes leaving them
  safe, then are deprecated.

No new chart, no new CLI, no new controller.

## User Stories

1. As an operator on a fresh cluster, I want one `helm install` to give me the whole
   product, so that I do not need to learn three charts and their order.
2. As an operator on a cluster without cert-manager, I want the admission webhook to work
   out of the box, so that my first `FlagSet` is accepted instead of failing with a
   connection error.
3. As an operator who already runs cert-manager, I want to keep issuing the webhook
   certificate through it, so that all certificates in my cluster follow one policy.
4. As an operator, I want the install to fail loudly or not at all, so that a green
   `helm install` means the product works.
5. As an operator, I want the server to keep serving flag evaluations even while the
   webhook certificate is being rotated, so that rotation is never an outage.
6. As an operator running more than one server replica, I want every replica to serve the
   same webhook certificate, so that validation does not fail on whichever pod the API
   server happens to reach.
7. As an operator upgrading Pintail, I want CRD changes to arrive with the server upgrade,
   so that I never run a separate `kubectl apply` again.
8. As an operator, I want the console reachable after install without a second chart, so
   that my team can manage flags from day one.
9. As an operator who does not want the console, I want to switch it off with one value,
   so that I expose only the API.
10. As an operator on the old three-chart layout, I want a documented migration that keeps
    every existing `FlagSet`, so that moving to the install package is not a data-loss
    event.
11. As an operator on the old layout, I want the deprecated charts to tell me what to do
    when I upgrade them, so that I learn about the migration from the tool I already use.
12. As a flag author, I want `FlagSet` validation to behave exactly as before, so that
    nothing I write today is accepted or rejected differently after the change.
13. As a security reviewer, I want the self-signed CA to be scoped to the webhook's Service
    name only and never trusted cluster-wide, so that the new mode widens no trust.
14. As a security reviewer, I want the server's permission to patch webhook configurations
    limited to its own configuration by name, so that the new mode cannot tamper with
    other webhooks.
15. As a Pintail maintainer, I want one chart, one release process and one install guide,
    so that the docs stop drifting from the code.
16. As a Pintail maintainer, I want an acceptance test that installs the product on a bare
    cluster, so that a hidden runtime prerequisite cannot come back unnoticed.

## Implementation Decisions

- **The install package is the `pintail-server` chart.** No umbrella chart is introduced.
  The chart already carries the server, its Service, its webhook configuration and the
  console switch; it gains only the certificate mode below and a changed default.
- **Webhook certificate: a third source.** The chart's existing
  `webhook.certificate.source` value (today `cert-manager` or `secret`) gains
  `self-signed`, which becomes the default. `cert-manager` and `secret` keep their current
  behaviour exactly.
- **Self-signed mode, in the server.** A small certificate module in the server:
  - on start, reads the certificate Secret named in the chart; if it is absent, generates
    a CA and a serving certificate for the webhook Service's DNS names and creates the
    Secret. A create that fails with "already exists" means another replica won; the
    replica reads the winner's Secret instead. Every replica therefore serves the same
    certificate.
  - patches the CA into the `caBundle` of the chart's own `ValidatingWebhookConfiguration`,
    addressed by name.
  - renews the serving certificate when less than a third of its lifetime remains, using an
    update conditioned on the Secret's resource version, so concurrent renewals resolve to
    one winner.
  - starts the webhook listener only once a certificate is loaded. Flag evaluation and the
    API do not wait for it.
- **The webhook keeps `failurePolicy: Fail`.** Validation is the point of the webhook;
  failing open would let invalid `FlagSet`s in. The server's readiness probe includes the
  webhook listener, so a pod that cannot serve validation is not ready and receives no
  webhook traffic.
- **RBAC for self-signed mode.** The server's Role gains get/create/update on the one
  certificate Secret, and a ClusterRole scoped by `resourceNames` to the chart's own
  webhook configuration gains get/patch. Neither is rendered in `cert-manager` or
  `secret` mode.
- **The cert-manager guard becomes loud.** When `webhook.certificate.source` is
  `cert-manager` and the cluster does not serve the cert-manager API, the chart fails the
  render with a message naming the value to change, instead of silently rendering nothing.
- **CRDs.** The server's existing startup CRD management stays on by default and is the
  only CRD path. The install guide drops the manual CRD step.
- **Console on by default.** The chart's `console.enabled` default flips from `false` to
  `true`. The console is already embedded in the server; no new component ships.
- **Retiring `pintail-crds`.** Its final release adds `helm.sh/resource-policy: keep` to
  the `FlagSet` CRD and a NOTES message pointing at the migration guide. Because the chart
  renders the CRD from `templates/`, uninstalling any earlier release deletes the CRD and
  every `FlagSet` with it; the final release exists so that uninstall becomes safe.
- **Retiring `pintail-console`.** Its final release marks the chart deprecated and prints
  the same pointer. It holds no state, so uninstalling it is safe at any time.
- **Migration order** for the three-chart layout, documented and tested: upgrade
  `pintail-crds` to its final release, upgrade `pintail-server` (console on), uninstall
  `pintail-console`, uninstall `pintail-crds`.
- **Docs.** The install guide shows one command. A short "Certificates" page explains the
  three sources and when to pick each. The troubleshooting entry for the webhook
  connection error is rewritten to point at the loud guard.

## Testing Decisions

**What "done" means.** The plan is done when the three acceptance scenarios below pass in
CI on the `pintail-server` repository's main branch. They are the test gate: an
implementation that passes them and nothing else is complete; one that fails any of them is
not, whatever else it ships.

**The seam: one.** All three scenarios run through the product's public surface and
nothing else: `helm install`/`upgrade`/`uninstall` with values, `kubectl` against the
`FlagSet` API, and HTTP against the server's evaluation API and console route. They assert
cluster state and responses, never template internals or log lines. This is the highest
seam available, and it is the one where the hidden runtime prerequisite actually hurt
users.

1. **Bare cluster.** A kind cluster with no cert-manager. `helm install pintail-server` with
   no values. Then: a valid `FlagSet` is accepted, an invalid one is rejected by the
   webhook with its validation message, a flag evaluates over the API, and the console
   route answers 200. With two replicas, ten consecutive `FlagSet` applies all succeed
   (every replica serves the same certificate).
2. **cert-manager present.** A kind cluster with cert-manager installed first,
   `webhook.certificate.source: cert-manager`. The same assertions as scenario 1, plus: the
   webhook's CA matches the cert-manager-issued certificate, and no self-signed Secret
   exists. A second run with cert-manager absent and the same value set asserts that the
   install fails with the guard's message.
3. **Migration.** A kind cluster with the old three-chart layout and five `FlagSet`s. Run
   the documented migration order. Then: all five `FlagSet`s still exist and evaluate
   unchanged, `helm list` shows exactly one Pintail release, and the console route answers
   200.

**One exception below the seam.** Certificate renewal depends on time, which a cluster test
cannot fast-forward cheaply. The certificate module gets unit tests with an injected clock
for exactly two behaviours: renewal happens once the renewal threshold is crossed, and two
concurrent renewals yield one certificate. Nothing else is unit-tested for this plan.

**What makes a good test here.** It fails if a user would notice, and only then. A test
that breaks when a template is refactored but the cluster ends in the same state is a bad
test.

**Prior art.** The server repository's existing end-to-end suite already creates a kind
cluster, installs the chart and applies `FlagSet`s. The three scenarios extend it; they do
not start a new harness.

## Out of Scope

- A Pintail operator or any new controller.
- An install CLI or wrapper script. `helm` is the install surface.
- Bundling cert-manager, in any form (see grill session 1, Q4).
- Changing `FlagSet` validation rules or the flag evaluation API.
- Installing Pintail through a GitOps tool. Nothing in this plan prevents it; nothing in
  this plan tests it.
- Archiving the `pintail-crds` and `pintail-console` repositories. That follows once their
  final releases have been out for two minor releases of the server.

## Further Notes

- **The first draft was bigger.** It proposed an umbrella chart over the three existing
  charts, a pre-install hook to order the CRDs, and a `pintailctl install` wrapper with
  preflight checks. The contrarian pass returned **SIMPLIFY**: two of the five stated
  problems (the manual CRD step and the separate console) no longer exist in the current
  code, so the only missing piece is the certificate. The plan went back to grilling
  (session 2) and returned in its current shape; the second contrarian pass is **BUILD**.
  See `contrarian/`.
- **Ground truth caught one hallucination.** The first draft named a
  `webhook.certificate.certManager.enabled` value that does not exist; the real value is
  `webhook.certificate.source`. Fixed inline before the page was rendered. See
  `ground-truth/`.
- **Why the grill missed the two stale problems.** The grill looked the facts up in the
  `pintail-crds` README, which still documents the manual CRD step. The contrarian
  re-derived them from the server's source. Stale docs are exactly why the contrarian does
  not trust the spec's framing.

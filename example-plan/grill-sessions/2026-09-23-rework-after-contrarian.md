# Grill session: rework after contrarian SIMPLIFY

- **Date:** 2026-09-23
- **Plan:** example-plan
- **Topic:** the contrarian's first-pass SIMPLIFY (`contrarian/2026-09-22-one-install-package.md`)
- **Note:** fictional worked example. Pintail and every path below are invented for the template.
- **Supersedes:** Q6 and Q7 of session 2026-09-22. Q1 to Q5 stand.

## Q8: The contrarian found the CRD step and the separate console already solved. Which chart is the install package now?

Facts re-checked against the server source, not the docs: the server applies its embedded
CRDs at startup (`--manage-crds`, default true), and the console is embedded in the server
binary behind the chart's `console.enabled` value (default false). Nothing is left for an
umbrella chart to aggregate except a stateless console chart and a CRD chart the server
makes redundant.

Recommended: `pintail-server` is the install package. Flip `console.enabled` to `true`,
add the self-signed certificate source, drop the umbrella, the CRD hook and the wrapper.

**Answer:** Yes. Drop the wrapper as well; if preflight matters, the chart should fail the
render, which Q4 already gives us for the one case that exists.

**Resolution:** install package is `pintail-server`. Umbrella chart, CRD hook and
`pintailctl install` are dropped. Glossary unchanged: **Install package** still fits.

## Q9: What happens to `pintail-crds` and `pintail-console`, and to installs that use them?

Found: `pintail-crds` renders the `FlagSet` CRD from `templates/`, so `helm uninstall
pintail-crds` deletes the CRD and, with it, every `FlagSet` in the cluster. An operator
following a naive "uninstall the old charts" instruction loses all flags.

Options: (1) document "never uninstall `pintail-crds`, just leave it"; (2) one final release
of `pintail-crds` that adds `helm.sh/resource-policy: keep` to the CRD, then a documented
order: upgrade to that release, upgrade the server, uninstall the console, uninstall the
CRD chart. Recommended: (2). Option (1) leaves a zombie release forever and a trap for the
next person who cleans up.

**Answer:** Option 2. And `pintail-console` gets a final release too, only to print the
pointer, because people will upgrade it before they read any guide.

**Resolution:** final releases for both charts (keep policy on the CRD; deprecation notice
on both); migration order documented and covered by an acceptance scenario. Archive the
repositories two server minor releases later.

## Q10: What proves the plan is done, and at which seam?

Recommended: one seam, the public install surface. Three kind-cluster scenarios in the
server's existing end-to-end suite: bare cluster (no cert-manager), cert-manager present
(including the loud-failure case), and migration from the three-chart layout with existing
`FlagSet`s. Two replicas in the bare scenario, to catch per-pod certificates.

**Answer:** Yes, and write it down as the definition of done, not as "tests we will add". One
addition: renewal cannot be seen in a cluster run without waiting months. Unit-test that
one module with a fake clock, and nothing else below the seam.

**Resolution:** the three scenarios are the test gate; the certificate module's renewal and
concurrent-renewal behaviour are unit-tested with an injected clock; no other unit tests
are required by this plan. Frontier empty; shared understanding confirmed.

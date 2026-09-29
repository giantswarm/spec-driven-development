# Ground-truth report: one install package for Pintail

- **Date:** 2026-09-23
- **Plan:** example-plan (`SPEC.md` after grill session 2026-09-23)
- **Sources: illustrative.** Pintail and its three repositories are fictional, invented for
  this template's worked example. The clone paths, commit SHAs and `file:line` references
  below are consistent across this example directory but point at nothing real. In a real
  plan every row cites a real shallow clone at a real SHA. The one exception is claim 8,
  which is about Helm itself and cites Helm's public documentation.
- **Method:** each claim about existing code was handed to its own verifier with the
  contrarian brief ("try to disprove this against the clone; cite `file:line` or it does
  not count"). No verifier edited any file.

## Summary

**8 claims checked: 7 VERIFIED, 1 MISMATCH, 0 NOT-FOUND, 0 UNCERTAIN.** The mismatch was a
hallucinated chart value in the Implementation Decisions; it was fixed inline in `SPEC.md`
before the website was rendered. No version-currency claims: the spec pins no versions.

## Clones checked

| Clone (`.agents_source/`) | Repository | Commit (illustrative) |
|---|---|---|
| `pintail-server` | acme/pintail-server | `4f2c9e1` (main, 2026-09-22) |
| `pintail-crds` | acme/pintail-crds | `b71a03d` (main, 2026-08-30) |
| `pintail-console` | acme/pintail-console | `9d0e5a7` (main, 2026-07-12) |

## Claims

| # | Verdict | Claim | Plan location | Evidence |
|---|---|---|---|---|
| 1 | VERIFIED | The server applies its embedded CRDs at startup, controlled by `--manage-crds` (default true) | `SPEC.md:40-41`, `SPEC.md:119-120` | `pintail-server/cmd/pintail-server/flags.go:42` (flag, default `true`); `pintail-server/internal/crds/manage.go:30-58` (server-side apply of the embedded CRDs before the manager starts) |
| 2 | VERIFIED | The console is embedded in the server binary and served when the chart's `console.enabled` is true; the default is `false` | `SPEC.md:39`, `SPEC.md:121-122` | `pintail-server/internal/console/embed.go:12` (`//go:embed dist`); `pintail-server/internal/httpserver/routes.go:77` (mounted under `/console` when enabled); `pintail-server/charts/pintail-server/values.yaml:88` (`enabled: false`) |
| 3 | VERIFIED | On a cluster without the cert-manager API the chart renders no `Certificate` and does not fail | `SPEC.md:21`, `SPEC.md:116-118` | `pintail-server/charts/pintail-server/templates/certificate.yaml:1` (whole template wrapped in `if .Capabilities.APIVersions.Has "cert-manager.io/v1"`, no `else`) |
| 4 | VERIFIED | The webhook fails closed for `FlagSet` create and update | `SPEC.md:20`, `SPEC.md:108` | `pintail-server/charts/pintail-server/templates/webhook.yaml:24-31` (rules `CREATE`, `UPDATE`; `failurePolicy: Fail`) |
| 5 | VERIFIED | Without certificate files the server keeps serving the API but never starts the webhook listener | `SPEC.md:22`, `SPEC.md:106-107` | `pintail-server/internal/webhook/server.go:57-71` (missing `tls.crt` in `--webhook-cert-dir` logs an error and returns; the HTTP API is started separately in `cmd/pintail-server/main.go:88`) |
| 6 | **MISMATCH** | The chart has a `webhook.certificate.certManager.enabled` value (default true) that the plan extends with a self-signed alternative | `SPEC.md:91-93` (as first written) | No `certManager` key exists anywhere in the chart. The real value is `webhook.certificate.source`, an enum of `cert-manager` or `secret`, default `cert-manager`, at `pintail-server/charts/pintail-server/values.yaml:131-140`, validated by `values.schema.json:212-218`. |
| 7 | VERIFIED | `pintail-crds` renders the `FlagSet` CRD from `templates/`, so uninstalling it deletes the CRD and every `FlagSet` | `SPEC.md:123-126` | `pintail-crds/charts/pintail-crds/templates/flagset-crd.yaml:1`; the chart has no `crds/` directory; no `helm.sh/resource-policy` annotation on the CRD |
| 8 | VERIFIED | Helm leaves a resource in place on uninstall when it carries `helm.sh/resource-policy: keep` | `SPEC.md:123-124` | Helm documentation, "Chart Development Tips and Tricks", section "Tell Helm Not To Uninstall a Resource" (primary source, not a clone) |

## Fixes applied to the plan

- **Claim 6: Fix.** The Implementation Decisions now extend the real value:
  `webhook.certificate.source` gains `self-signed` as a third option and the new default;
  `cert-manager` and `secret` keep their behaviour. The loud guard (`SPEC.md:116-118`) was
  reworded to trigger on `source: cert-manager`. Had this gone unchecked, the implementing
  agent would have added a second, conflicting switch next to the existing enum, and the
  schema at `values.schema.json:212-218` would have rejected any values file that set it.

All VERIFIED claims were left untouched.

## Not verifiable against a clone

The central user-facing fact (a green `helm install` followed by a failing first `FlagSet`
on a cluster without cert-manager) is runtime behaviour. It is reproduced by acceptance
scenario 1 of the spec, which fails on the current `main` of `pintail-server` and must pass
for the plan to be done.

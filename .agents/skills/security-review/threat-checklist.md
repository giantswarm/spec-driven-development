# Threat checklist

Questions for `security-review`, by area. Apply each to every trust-boundary
crossing and new component. They are prompts for thinking, not a form to fill: record
only what applies, and one line for each area that doesn't. Examples lean on Kubernetes;
translate them to your platform's equivalents.

## STRIDE, per crossing

| Category | Ask |
| --- | --- |
| Spoofing | Can a caller claim to be someone else here? What proves identity, and who issued it? |
| Tampering | Can the request, a stored object or a config value be changed on the way or at rest by someone who shouldn't? |
| Repudiation | Can an actor deny having done a sensitive action? Is it logged with the real principal? |
| Information disclosure | What leaks to logs, status fields, error messages, other tenants, an LLM's context? |
| Denial of service | Can one tenant or one input exhaust a shared resource (quota, queue, token budget)? |
| Elevation of privilege | Can the component be made to act with more rights than its caller has? |

## Identity and acting-as-caller

- Does each hop act as the human or agent who asked, or as a shared service identity? A
  shared identity that can do more than the caller is a confused deputy.
- Are tokens passed through to a downstream service that wasn't their audience? Token
  exchange with a narrowed audience beats passthrough.
- Where is authorization decided — at the edge only, or at the resource too? A check only
  at the edge is gone once anything else reaches the resource.
- Who can create or edit the objects (CRs, configs) that grant access? That is the real
  permission.
- Who is actually in each subject the plan binds? Broad groups are wider than they read:
  `system:authenticated` includes every ServiceAccount, so every pod on the cluster.
- When two controls disagree (an allow-list and a protection, a default and an override),
  which one wins? The weaker one usually does unless the plan says otherwise.

## Credentials and secrets

- Where does each credential live, who can read it (RBAC on the Secret, the pod, the
  node), and how long is it valid?
- Is it scoped to what this component needs, or a broad one reused for convenience?
- Rotation and revocation: what happens to a leaked one, and how fast?
- Can it end up in logs, CR status, events, traces, an agent transcript or a model prompt?

## Tenant and workload isolation

- What separates customers, teams and environments: namespace, cluster, RBAC, network
  policy? Which of those does the plan rely on, and does it hold for every new component?
- Shared caches, stores, queues or controllers: is every read and write keyed and
  checked by tenant?
- Can a less-trusted cluster or environment reach a more-trusted one (a workload cluster its
  control plane, one customer another) through the new path?

## Network and egress

- Which new outbound connections does the plan add, and to where? Is egress
  allow-listed or open?
- Inbound: which new endpoints are exposed, to whom, with what authentication?
- Could the new path be used for server-side request forgery (the component fetches a URL
  the caller supplies)?

## Untrusted input, agents and prompt injection

- Which inputs come from outside the trust boundary: user text, issue and PR bodies, Slack
  messages, web pages, tool results, file contents?
- Can such input reach an LLM that can then call tools that write, send or delete? That
  is the prompt-injection path: content steering an agent into an action its user didn't
  ask for.
- Which tool calls need human approval, and can the approval be bypassed or pre-granted
  by content?
- Classic injection still applies: shell, template, query and YAML built from input.

## Supply chain

- New dependencies, images, charts, MCP servers or forks: who maintains them, are they
  pinned by digest or commit, and who can push to what is pinned?
- Does the plan run third-party code with access to credentials or cluster rights?

## Data handling

- Does the plan touch personal or customer data? Where is it stored, for how long, and
  who can export it?
- Does data cross a region or leave your organisation's control (an external API, a model
  provider)?

## Audit and detection

- Is every sensitive action (granting access, using a credential, an agent's write)
  logged with the real principal, and is the log out of reach of the actor?
- Would an operator notice abuse — an alert, a dashboard, a rate?

## Failure behaviour

- When a dependency (identity provider, policy engine, gateway) is down, does the path
  fail closed or open?
- What does a partial failure leave behind: orphaned credentials, half-granted access,
  stale caches that still authorize?

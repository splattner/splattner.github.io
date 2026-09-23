---
title: "Three Small Kubernetes Tools Born From Real Annoyances"
date: 2026-09-21
tags: [Kubernetes, Open Source]
---

Not every project needs a grand plan. These three came out of specific problems I
ran into — some in my homelab, some during my day job as a Platform Architect —
and each one just solves that problem and stops. A lot of AI assistance went into
all three, they're shipped as-is with no roadmap for continuous development, and
I'm fully open to contributions if someone finds them useful and wants to take
them further. I'll keep contributing myself whenever I have time or hit a need for
a new feature.

## node-providerid

[node-providerid](https://github.com/splattner/node-providerid) is a Go CLI that
reads or changes a Kubernetes Node's `spec.providerID` directly through the etcd
v3 API, targeting embedded etcd on a k3s server by default.

`providerID` links a Node object to its machine in the cloud provider, and
Kubernetes treats it as immutable once set — the API server rejects any attempt
to change a non-empty value, and it can't be cleared through `kubectl edit` or
`kubectl patch`. In practice it still ends up wrong sometimes: a Node registered
before the cloud provider was configured, a migration between providers, or a bad
`--provider-id` baked in at first boot. The usual fix is to drain, delete and
re-register the Node. When that isn't practical, the only place left to correct
the value is the datastore itself — which is exactly what this tool does, as
narrowly as possible: it patches only the `providerID` field, guards the write
with an etcd compare-and-swap on the Node's revision, and writes a per-key backup
first.

Worth saying plainly: `set` writes directly to your cluster's backing store and
bypasses every Kubernetes safety layer — schema validation, admission control,
authorization, the lot. `get` is read-only and safe; `set` is a deliberate,
last-resort operation, and the README says so at length before you get anywhere
near running it.

## burrow

[burrow](https://github.com/splattner/burrow) exposes a TCP service running
outside a Kubernetes cluster to workloads running inside it, without opening
inbound firewall ports or configuring static routes.

The client runs wherever the service actually lives — laptop, edge node, private
server — and dials *outbound* to a WebSocket endpoint on a server component
running inside the cluster. Because the client always initiates the connection,
it works through NAT, firewalls and most corporate networks. Traffic from pods
reaches the local service through that persistent tunnel; the server can
optionally manage a Kubernetes `Service` object per connected client, so pods
reach tunnelled services by a stable DNS name rather than tracking ports by hand.
It's the kind of thing that's handy for reaching a homelab machine from an
in-cluster job, or bridging a service that will never itself run in Kubernetes.

## alertmanager-label-enricher

[alertmanager-label-enricher](https://github.com/splattner/alertmanager-label-enricher)
is an inline proxy that sits between Prometheus and Alertmanager and enriches
alert labels from external lookups — Kubernetes objects, HTTP endpoints, or a
static file — before forwarding alerts on.

Prometheus's own `alert_relabel_configs` already covers static and conditional
label rewriting, but nothing for a lookup against something external — for
example, adding a `team` label read off the Kubernetes object for the alert's
namespace, or off a CMDB via HTTP. Because enrichment happens *before*
Alertmanager sees the alert, the new labels participate fully in routing,
grouping, inhibition and silences, exactly like any label Prometheus set itself.

Rules run in declared order, all matching rules apply, and later rules see labels
earlier ones added. Overwriting or dropping an existing label changes the alert's
fingerprint in Alertmanager — which can invalidate existing silences — so both
require an explicit opt-in, while annotations (no fingerprint risk) don't.
`alertname` gets extra protection since it's Alertmanager's primary identifying
label. A rule can be marked `required`, failing the whole batch closed so
Prometheus retries if its lookup fails; everything else fails open rather than
block delivery.

---

None of these three are trying to be a platform or a product — they each fix one
specific thing that was in my way. If one of them is in your way too, they're on
GitHub and I'll happily look at a PR.

# `katastroma`

Open GitOps platform for Kubernetes. Breaks the GitOps problem into its
fundamental primitives and defines the interfaces between them.

## Architecture

GitOps asks two questions:

1. **What resources should exist?** Given a source (a git repo, a branch, a
   path, and credentials), determine the resource inventory.
2. **How do we make them exist?** Given a resource inventory, apply it to a
   cluster.

Most GitOps systems collapse watching, resolving, and provisioning into a single
monolith. Katastroma separates them at every natural boundary.

### The Pipeline

External event → [epibathra](#epibathra) receives intent → [pedalion](#pedalion)
reconciles → [resolver](#keleustēs) determines what →
[provisioner](#katartismos) executes how.

No polling. Event-driven end to end.

### Components

| Component                   | Role                                          |
| --------------------------- | --------------------------------------------- |
| [tropis](#tropis)           | CRD type definitions — the backbone           |
| [epibathra](#epibathra)     | Tenant manager — receives external intent     |
| [pedalion](#pedalion)       | Reconciler — steers and orchestrates          |
| [keleustēs](#keleustēs)     | Resolver interface — what should exist?       |
| [katartismos](#katartismos) | Provisioner interface — how do we apply them? |
| [orpheus](#orpheus)         | Reference resolver implementation             |
| [histia](#histia)           | Reference provisioner implementation          |

## Tropis

Kubernetes CRD type definitions and generated API machinery.

[katastroma/tropis](https://github.com/katastroma/tropis)

## Epibathra

Platform API for tenant management and event entrypoint. Provisions tenants and
their namespaces, creates Application and RepoCredential resources on their
behalf, and exposes an API for tenant self-service. Receives webhooks from
tenant repos — this is the external signal that enters the platform.

[katastroma/epibathra](https://github.com/katastroma/epibathra)

## Pedalion

Reconciler and orchestrator. Watches Application resources in the cluster and
drives the reconcile loop. Does not resolve sources or provision resources
itself — it delegates to a resolver to determine what should exist and a
provisioner to execute.

[katastroma/pedalion](https://github.com/katastroma/pedalion)

## Keleustēs

Resolver interface. Defines the contract for answering "what resources should
exist?" One of two fundamental GitOps primitives. Designed for ecosystem
adoption — any GitOps system can implement it.

[katastroma/keleustes](https://github.com/katastroma/keleustes)

## Katartismos

Provisioner interface. Defines the contract for answering "how do we make them
exist?" The second fundamental GitOps primitive. Designed for ecosystem adoption
— any GitOps system can implement it.

[katastroma/katartismos](https://github.com/katastroma/katartismos)

## Orpheus

Katastroma's reference resolver. Implements the keleustēs interface. Given a
source repository and credentials, clones, renders, and produces the resource
inventory.

[katastroma/orpheus](https://github.com/katastroma/orpheus)

## Histia

Katastroma's reference provisioner. Implements the katartismos interface.
Applies resources directly to the Kubernetes API using server-side apply. No
external dependencies, no running services — just the cluster.

[katastroma/histia](https://github.com/katastroma/histia)

## Swappability

The resolver and provisioner are independently swappable:

- Katastroma's resolver with a Flux-based provisioner
- An ArgoCD-backed provisioner with a custom resolver
- Both reference implementations for a fully self-contained stack

The interfaces are the product. The implementations are reference points.

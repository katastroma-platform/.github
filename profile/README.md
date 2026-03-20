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

External event → [grammateus](#grammateus) receives intent →
[pedalion](#pedalion) reconciles → [adapter](#zeugma) retrieves, renders, and
provisions.

No polling. Event-driven end to end.

### Components

| Component                   | Role                                          |
| --------------------------- | --------------------------------------------- |
| [oiax](#oiax)               | Platform bootstrap and teardown CLI           |
| [tropis](#tropis)           | CRD type definitions — the backbone           |
| [epibathra](#epibathra)     | Tenant management chart — composes components |
| [grammateus](#grammateus)   | API server — receives intent, manages tenants |
| [prora](#prora)             | Self-service frontend                         |
| [pedalion](#pedalion)       | Reconciler — watches and delegates            |
| [zeugma](#zeugma)           | Adapter service contract (gRPC)               |
| [ergata](#ergata)           | Adapter implementations                       |
| [keleustēs](#keleustēs)     | Resolution interfaces — what should exist?    |
| [katartismos](#katartismos) | Provisioner interface — how do we apply them? |
| [orpheus](#orpheus)         | Reference resolution implementation           |
| [histia](#histia)           | Reference provisioner implementation          |

## Oiax

Platform bootstrap and teardown CLI. Installs CRDs, waits for them to establish,
creates the root Application, and deploys pedalion. Reverses all of that on
teardown. Runs locally using the user's kubeconfig.

[katastroma/oiax](https://github.com/katastroma/oiax)

## Tropis

Kubernetes CRD type definitions and generated API machinery.

[katastroma/tropis](https://github.com/katastroma/tropis)

## Epibathra

Helm chart that composes the tenant management stack. Deploys grammateus, prora,
and off-the-shelf components (auth/IdP, gatekeeper) as dependencies. No source
code of its own.

[katastroma/phortion/epibathra](https://github.com/katastroma/phortion/tree/default/epibathra)

## Grammateus

API server. Manages tenant namespaces and attaches Applications and
RepoCredentials to them. Receives webhooks from tenant repositories. This is the
external signal that enters the platform.

[katastroma/grammateus](https://github.com/katastroma/grammateus)

## Prora

Tenant self-service frontend. Consumes the grammateus API.

[katastroma/prora](https://github.com/katastroma/prora)

## Pedalion

Application operator. Watches Application resources in the cluster and drives
the reconcile loop. Delegates to an adapter service for resolution and
provisioning.

[katastroma/pedalion](https://github.com/katastroma/pedalion)

## Zeugma

Adapter service contract. Defines the gRPC API between pedalion and whichever
adapter implementation powers it. Pedalion imports the generated client. Adapter
implementations import the generated server.

[katastroma/zeugma](https://github.com/katastroma/zeugma)

## Ergata

Adapter implementations. Each adapter implements the zeugma gRPC service as a
standalone image. Includes the native adapter (orpheus + histia) and an ArgoCD
adapter.

[katastroma/ergata](https://github.com/katastroma/ergata)

## Keleustēs

Resolution interfaces. Defines the retriever and renderer contracts for
answering "what resources should exist?" One of two fundamental GitOps
primitives. Designed for ecosystem adoption.

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

The adapter is a separate deployment, selected by configuration. Swap the image,
swap the backend:

- The native adapter (orpheus + histia) for a self-contained stack
- An ArgoCD adapter that delegates to a running ArgoCD instance
- Any custom adapter that implements the zeugma gRPC contract

The interfaces are the product. The implementations are reference points.

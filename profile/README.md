# Katastroma

Multi-tenant GitOps platform for Kubernetes. Webhook-driven, event-based, no
polling. Tenant isolation through Kubernetes-native RBAC, impersonation, and
admission control.

## Architecture

GitOps has four operations:

1. **Fetch** — given a repo URL, revision, path, and credentials, retrieve the
   source
2. **Render** — given source content, produce Kubernetes manifests
3. **Order** — given unordered manifests, sort them into a safe apply order
4. **Provision** — given ordered manifests, apply them to the cluster

Each operation is a separate service with its own interface, implementation, and
deployment. A webhook server orchestrates the pipeline.

### The Pipeline

Git push → [pharos](#pharos) receives webhook → [phortizo](#phortizo) fetches
source → [orpheus](#orpheus) renders manifests → orderer sorts manifests →
[histia](#histia) provisions to cluster via impersonation.

### Components

| Component                   | Role                                 |
| --------------------------- | ------------------------------------ |
| [grammateus](#grammateus)   | Tenant management API server         |
| [prora](#prora)             | Tenant self-service frontend         |
| [pharos](#pharos)           | Webhook server — receives git events |
| [naukleros](#naukleros)     | Retriever interface (gRPC proto)     |
| [phortizo](#phortizo)       | Retriever implementation             |
| [keleustēs](#keleustēs)     | Renderer interface (gRPC proto)      |
| [orpheus](#orpheus)         | Renderer implementation              |
| TBD                         | Orderer interface (gRPC proto)       |
| TBD                         | Orderer implementation               |
| [katartismos](#katartismos) | Provisioner interface (gRPC proto)   |
| [histia](#histia)           | Provisioner implementation           |

## Grammateus

Tenant management API server. Manages tenant namespaces, hierarchy, service
accounts, credentials, and resource queries. Handles onboarding, offboarding,
and webhook signature verification. See
[grammateus README](https://github.com/katastroma/grammateus) for the full
multi-tenancy architecture.

[katastroma/grammateus](https://github.com/katastroma/grammateus)

## Prora

Tenant self-service frontend. Consumes the grammateus API.

[katastroma/prora](https://github.com/katastroma/prora)

## Pharos

Webhook server. Receives git push events, verifies HMAC signatures through
grammateus, determines if the push affects the tenant's registered revision and
path, and orchestrates the fetch → render → provision pipeline.

[katastroma/pharos](https://github.com/katastroma/pharos)

## Naukleros

Retriever interface. gRPC proto defining the contract for fetching source
content given a repo URL, revision, path, and credentials.

[katastroma/naukleros](https://github.com/katastroma/naukleros)

## Phortizo

Retriever implementation. Implements the naukleros interface. Fetches source
using git libraries.

[katastroma/phortizo](https://github.com/katastroma/phortizo)

## Keleustēs

Renderer interface. gRPC proto defining the contract for rendering Kubernetes
manifests from source content.

[katastroma/keleustes](https://github.com/katastroma/keleustes)

## Orpheus

Renderer implementation. Implements the keleustēs interface. Renders manifests
from source content using helm, kustomize, or raw YAML.

[katastroma/orpheus](https://github.com/katastroma/orpheus)

## Orderer (TBD)

Ordering is a distinct pipeline stage between rendering and provisioning.
Different render backends produce manifests in different orders — Helm sorts by
its hardcoded `InstallOrder`, Kustomize's SDK returns accumulation order by
default, and raw YAML has no ordering at all. The Kubernetes API applies one
resource at a time, and apply order matters: a Namespace must exist before
resources can be created in it, CRDs must exist before custom resources, RBAC
before workloads that depend on service accounts, etc.

The orderer service accepts manifests from the renderer and ensures they are in
a safe apply order. A per-source ordering strategy could be explored in the
future.

Interface and implementation repo names TBD.

## Katartismos

Provisioner interface. gRPC proto defining the contract for applying manifests
to a cluster and pruning resources no longer in the rendered output.

[katastroma/katartismos](https://github.com/katastroma/katartismos)

## Histia

Provisioner implementation. Implements the katartismos interface. Applies
manifests to the cluster using server-side apply with Kubernetes impersonation.
Prunes resources by label.

[katastroma/histia](https://github.com/katastroma/histia)

## Deployment

The platform is deployed as Helm charts via
[phortion](https://github.com/katastroma/phortion):

- **Epibathra** — tenant management stack (grammateus, prora, gatekeeper,
  auth/IdP)
- **Prymna** — GitOps engine (pharos, phortizo, orpheus, histia)

## Swappability

Each service implements a gRPC interface. Swap the implementation by deploying a
different image. The interfaces are the contracts — the implementations are
reference points.

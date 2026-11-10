---
title: Kubernetes Operator
description: Run DocumentDB on Kubernetes with the DocumentDB Kubernetes Operator. Learn when to choose it, what your cluster needs, and where to find the operator's documentation.
---

# Kubernetes Operator

The [DocumentDB Kubernetes Operator](https://github.com/documentdb/documentdb-kubernetes-operator) runs and manages DocumentDB clusters on Kubernetes. You describe a cluster as a `DocumentDB` resource, and the operator provisions it and handles high availability, failover, backups, and upgrades, building on [CloudNativePG](https://cloudnative-pg.io/docs/) for the PostgreSQL layer. Every instance pairs a PostgreSQL container with a DocumentDB gateway sidecar, so applications connect using MongoDB-compatible drivers exactly as they would to any other DocumentDB deployment.

The operator is a separate project, with its own repository, release cadence, and documentation. This page covers when to reach for it and what your cluster needs; the operator's own documentation covers everything else.

## Versions

Operator **1.1.1** runs DocumentDB engine **1.0-0**. Pin the chart version for repeatable installs (cert-manager must be installed first; see the quickstarts below):

```bash
helm install documentdb-operator oci://ghcr.io/documentdb/documentdb-operator \
  --version 1.1.1 \
  --namespace documentdb-operator \
  --create-namespace \
  --wait
```

## When to use it

| Option | Best for |
|---|---|
| [DocumentDB Local](../documentdb-local/index.md) | A single container for development, prototyping, and integration tests. |
| [Pre-built packages](../getting-started/prebuilt-packages.md) | Adding DocumentDB to a PostgreSQL server you already run and operate yourself. |
| Kubernetes Operator | Running DocumentDB as a replicated service, with automatic failover, rolling upgrades, backup and restore, and multi-region deployments. |

## What your cluster needs

The operator needs Kubernetes 1.32 or later. Check one more thing before you plan a deployment: how the DocumentDB extension reaches PostgreSQL pods. By default the operator mounts it with the Kubernetes [ImageVolume](https://kubernetes.io/docs/concepts/storage/volumes/#image) feature when the cluster supports it:

- **Kubernetes 1.35 or later.** ImageVolume is GA from 1.36. In 1.35 it is beta and on by default, so check that your cluster hasn't turned the `ImageVolume` feature gate off. On 1.33 and 1.34 it is off by default, and works only if you enable the gate on the API server and the kubelets.
- **containerd 2.1 or later, or CRI-O 1.33 or later** as the container runtime. The Docker runtime does not support ImageVolume.

Otherwise the operator uses a combined image that ships PostgreSQL 18 and the extension together. It decides when a `DocumentDB` cluster is created, and the cluster keeps that layout. Managed Kubernetes offerings vary in the versions and runtimes they expose, and local tools often need an explicit flag to select a supported runtime, so confirm both up front if you want ImageVolume. [Before You Start](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/getting-started/before-you-start/) has the full prerequisite list, including supported tooling versions and resource sizing.

## Operator documentation

The operator's documentation is versioned and published alongside its releases:

- [Overview](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/) — what the operator installs and how the pieces fit together
- [Quickstart: Kind](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/getting-started/quickstart-kind/) and [Quickstart: K3s](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/getting-started/quickstart-k3s/) — deploy a cluster locally
- [Connecting to DocumentDB](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/getting-started/connecting-to-documentdb/) — `mongosh` and driver examples for Python, Node.js, Go, and Java
- [Architecture overview](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/architecture/overview/) — how the operator, CloudNativePG, and the gateway interact
- Configuration: [networking](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/configuration/networking/), [storage](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/configuration/storage/), and [TLS](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/configuration/tls/)
- [High availability](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/high-availability/overview/) — replicas, failover behavior, and recovery objectives
- Operations: [backup and restore](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/operations/backup-and-restore/) and [upgrades](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/operations/upgrades/)
- [Monitoring](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/monitoring/overview/) and the [kubectl plugin](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/kubectl-plugin/)
- [API reference](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/api-reference/) — the `DocumentDB`, `Backup`, and `ScheduledBackup` resources
- [FAQ](https://documentdb.io/documentdb-kubernetes-operator/latest/preview/faq/)

Report operator issues in the [operator repository](https://github.com/documentdb/documentdb-kubernetes-operator/issues) rather than this one.

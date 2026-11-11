---
title: documentdb_distributed
description: Extend DocumentDB with distribution, colocation, and rebalancing.
type: commands
category: documentdb_distributed
---

# documentdb_distributed

`documentdb_distributed` is an optional PostgreSQL extension that adds distributed behavior to DocumentDB. It implements distribution hooks used by existing DocumentDB commands and adds SQL APIs for cluster administration, collection placement, and rebalancing.

These features require **`documentdb_distributed` to be installed and loaded**. **[Citus](https://github.com/citusdata/citus), a PostgreSQL extension for distributing tables across nodes, is a prerequisite**, together with `documentdb_core` and `documentdb`; the core DocumentDB extension alone does not provide this distributed surface.

## Prerequisites and installation

Before installing prerequisites, review their license terms and any usage, distribution, or deployment restrictions that apply to the selected packages and versions.

Install compatible Citus and DocumentDB extension packages on the coordinator and workers. Configure a Citus cluster with its coordinator and worker nodes before initializing DocumentDB's distributed metadata.

**Citus 14 is the recommended Citus version.** Use a build compatible with your PostgreSQL major version and platform. Do not assume that an arbitrary newer Citus package, or the same Citus build across different PostgreSQL major versions, is compatible.

Build or obtain `documentdb_core`, `documentdb`, and `documentdb_distributed` from the same DocumentDB release for the same PostgreSQL major version and platform, and use a consistent package set on the coordinator and workers. For provider-managed packages, use the provider's supported dependency combination instead of substituting independently selected versions. The installed-version query below reports what is installed; it does not certify compatibility.

The `pg_documentdb_distributed` library must be loaded through `shared_preload_libraries` on the coordinator and workers, after `citus` and `pg_documentdb`. Preserve the other libraries required by your deployment and restart PostgreSQL after changing this setting.

Once `citus`, `documentdb_core`, and `documentdb` have been created in the target PostgreSQL database, an administrator can create the extension:

```sql
CREATE EXTENSION IF NOT EXISTS documentdb_distributed;
```

On the configured coordinator, initialize the distributed metadata:

```sql
SELECT documentdb_api_distributed.initialize_cluster();
```

Creating the extension does not itself register Citus worker nodes. Worker registration and connectivity are part of the Citus cluster setup.

Check installed dependencies in the target database:

```sql
SELECT extname, extversion
FROM pg_extension
WHERE extname IN ('citus', 'documentdb_core', 'documentdb', 'documentdb_distributed')
ORDER BY extname;
```

## Command extensions

| API | Distributed behavior |
| --- | --- |
| `shard_collection` / `shardCollection` | [Distribute a collection](sharding.md#shard-a-collection) using its hashed shard key and distributed backing tables. |
| `reshard_collection` / `reshardCollection` | [Redistribute existing data](sharding.md#reshard-a-collection) using a different shard key or requested distribution. |
| `unshard_collection` / `unshardCollection` | [Return a collection to an unsharded layout](sharding.md#unshard-a-collection), while retaining distributed backing storage. |
| `coll_mod` / `collMod` | [Colocate collections](colocation.md) through the `colocation` extension option; propagate supported index modifications to distributed shard indexes. |
| `move_collection` / `moveCollection` | [Move an unsharded collection](moveCollection.md) to a selected node group, with blocking or logical-replication transfer. |
| `balancerStart`, `balancerStatus`, `balancerStop` | [Start, inspect, and stop shard rebalancing](rebalancer.md), subject to provider gateway integration and separate rebalancer enablement. |
| `getShardMap` | [Discover node groups and per-node metadata](shard-discovery.md#getshardmap), when the gateway integrates the distributed implementation. |
| `listShards` | [List node groups and their active members](listShards.md), when the gateway integrates the distributed implementation. |
| `listCollections` | [Add shard-count and colocation metadata](colocation.md#listcollections) when requesting `addDistributedMetadata`. |

The snake-case names above are PostgreSQL SQL APIs. The camel-case names identify corresponding wire commands where available. Existing shell helpers remain described in the [sharding reference](../../sharding/sharding-commands.md).

Installing a PostgreSQL extension does not change which commands or fields a gateway forwards. For example, the default gateway's legacy sharding adapter does not forward `numInitialChunks`; the SQL command-document APIs can pass that option directly. Rebalancer wire commands depend on the provider gateway plumbing them to the distributed implementation.

## Cluster lifecycle APIs

The extension also exports SQL lifecycle APIs used by deployment administration:

| API | Purpose |
| --- | --- |
| `documentdb_api_distributed.initialize_cluster()` | Initialize distributed metadata for the configured cluster, as shown above. An already-initialized cluster is left unchanged. |
| `documentdb_api_distributed.complete_upgrade()` | Apply the distributed setup actions needed for the installed extension version. Returns a boolean indicating whether upgrade work was applied. This is part of the deployment's extension-upgrade workflow, not a replacement for installing or upgrading extension packages. |

Worker index-update, version-cache, and catalog-repair helpers support the implementation and its maintenance workflows; they are not additional application wire commands.

## Implementation

The [distributed extension source tree](https://github.com/documentdb/documentdb/tree/main/internal/pg_documentdb_distributed) links to current `main`. Its `documentdb_distributed.control` file declares the Citus and DocumentDB dependencies, and `src/documentdb_distributed.c` registers distributed hooks during PostgreSQL startup. For the revision used to audit the command behavior in this section, see [wire protocol and common fields](../../diagnostic/wire-protocol.md).

Source: `internal/pg_documentdb_distributed/src/distribution/distributed_hooks.c`, `InitializeDocumentDBDistributedHooks`; `src/colocation/shard_colocation.c`, `UpdateColocationHooks`; lifecycle APIs: `src/distribution/cluster_operations.c`, `command_initialize_cluster` and `command_complete_upgrade`.

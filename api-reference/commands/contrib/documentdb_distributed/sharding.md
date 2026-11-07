---
title: Distributed sharding and placement
description: Shard, reshard, unshard, and move DocumentDB collections with documentdb_distributed.
type: commands
category: documentdb_distributed
---

# Distributed sharding and placement

These operations require [documentdb_distributed and its prerequisites](overview.md#prerequisites-and-installation). The shell examples use the [airline dataset](../../diagnostic/wire-protocol.md#airline-example-dataset).

Availability of these wire APIs and their options is subject to provider gateway integration.

## Shard a collection

Distribute `bookings.flights` by hashed route number, requesting four initial chunks:

```javascript
sh.shardCollection("bookings.flights", { route_no: "hashed" }, { numInitialChunks: 4 })
```

The extension distributes the backing table. Sharding an existing collection redistributes its data and recreates its indexes; it is not just a metadata update.

| Parameter | Description |
| --- | --- |
| **`shardCollection`** | (Required) A fully qualified namespace in `database.collection` form. |
| **`key`** | (Required) A supported hashed shard-key document. |
| **`numInitialChunks`** | (Optional) A positive integer requesting the initial chunk count, up to the configured sharding limit. The distributed hook uses this count when distributing the backing table. |

The provider gateway must forward `numInitialChunks` to honor the requested count. See [sharding commands](../../sharding/sharding-commands.md#shardcollection-and-reshardcollection) for the shared command options.

## Reshard a collection

After the preceding example, redistribute the flights by departure airport:

```javascript
sh.reshardCollection("bookings.flights", { departure_airport: "hashed" }, { numInitialChunks: 4 })
```

This rebuilds the collection's distribution using the new key. It can move data and recreate indexes, so allow for the work involved in rewriting an existing collection.

| Parameter | Description |
| --- | --- |
| **`reshardCollection`** | (Required) A fully qualified namespace. |
| **`key`** | (Required) The new supported hashed shard-key document. |
| **`numInitialChunks`** | (Optional) A positive integer specifying the requested chunk count, subject to the configured limit. |
| **`forceRedistribution`** | (Optional) A boolean-like value that forces redistribution even when the shard key matches the existing key. Without it, a matching key can cause the operation to be skipped. |

## Unshard a collection

Return the flights to an unsharded layout:

```javascript
db.adminCommand({ unshardCollection: "bookings.flights" })
```

| Parameter | Description |
| --- | --- |
| **`unshardCollection`** | (Required) A fully qualified namespace. |

Unsharding removes the collection's logical shard key and redistributes its data into the unsharded layout. It does not uninstall the distributed extension. This API does not select a destination node; use `moveCollection` separately when a placement change is needed. The `sh.unshardCollection()` helper requires a `toShard` argument, so the example sends only the namespace with `db.adminCommand`.

## Move an unsharded collection

After unsharding, use [`moveCollection`](moveCollection.md) to relocate the collection to a selected node group. Its reference covers destination discovery, `useLogicalReplication`, transaction restrictions, and the effect on colocation.

## Implementation

See [wire protocol and common fields](../../diagnostic/wire-protocol.md) for the source revision.

The `shardCollection`, `reshardCollection`, and `unshardCollection` wire APIs map to the SQL UDFs `documentdb_api.shard_collection`, `documentdb_api.reshard_collection`, and `documentdb_api.unshard_collection`, respectively. The command-document overloads accept BSON argument documents.

The SQL [sharding entry points](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/pg_documentdb/sql/udfs/schema_mgmt/shard_collection--latest.sql) call the core sharding implementation. `DistributePostgresTableCore` in `internal/pg_documentdb_distributed/src/distribution/distributed_hooks.c` distributes the backing table through `create_distributed_table`.

Source: `pg_documentdb/src/sharding/sharding.c`, `ParseShardCollectionRequest`, `ParseReshardCollectionRequest`, and `ParseUnshardCollectionRequest`; placement: `internal/pg_documentdb_distributed/src/colocation/shard_colocation.c`, `documentdb_command_move_collection`.

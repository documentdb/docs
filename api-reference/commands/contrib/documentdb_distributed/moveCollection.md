---
title: moveCollection
description: Move an unsharded DocumentDB collection to another node group with documentdb_distributed.
type: commands
category: documentdb_distributed
---

# moveCollection

Moves an existing unsharded collection to a selected node group. Requires [documentdb_distributed and its prerequisites](overview.md#prerequisites-and-installation), plus the deployment's move-collection capability.

The `moveCollection` wire API is subject to provider gateway availability.

## Parameters

| Parameter | Description |
| --- | --- |
| **`moveCollection`** | (Required) A string specifying the fully qualified namespace of an existing unsharded collection. |
| **`toShard`** | (Required) A string in `shard_<groupid>` form. The group must have an active primary node in the cluster metadata. This is a node-group identifier, not a collection's physical shard ID. |
| **`useLogicalReplication`** | (Optional) A boolean-like value that, when `true`, forces transfer via logical replication between shards without blocking writes during the data copy. Defaults to `false`, which blocks writes during transfer. Requires the deployment to be configured for logical shard transfer. |

## Choose a destination

Call [`listShards`](listShards.md) through a gateway integrated with the distributed implementation:

```javascript
db.adminCommand({ listShards: 1 })
```

Use an entry's `_id` as `toShard`; for example, `_id: "shard_1"` corresponds to `toShard: "shard_1"`. Confirm with [`getShardMap`](shard-discovery.md#getshardmap) that the group has an active primary member, and choose a different destination from the collection's current placement in [`config.chunks`](colocation.md#configchunks). Do not assume that the illustrative group exists in every deployment.

## Move with the default transfer mode

Using the [airline dataset](../../diagnostic/wire-protocol.md#airline-example-dataset), assume that `bookings.flights` is unsharded and group 1 is an eligible destination:

```javascript
db.adminCommand({
  moveCollection: "bookings.flights",
  toShard: "shard_1"
})
```

By default, the transfer blocks writes while the backing shard moves. Run the command outside an explicit transaction. The command returns `{ ok: 1 }` after the placement move completes; it does not enqueue a background rebalance.

## Use logical replication

On a deployment configured for logical shard transfer, the following is an alternative to the preceding move, starting from the same source placement:

```javascript
db.adminCommand({
  moveCollection: "bookings.flights",
  toShard: "shard_1",
  useLogicalReplication: true
})
```

This selects `force_logical` instead of `block_writes`. It does not change the destination or make a sharded collection eligible, and it is not a guarantee that the operation will never block.

## Colocation and distribution

The move detaches the collection from its previous colocation group before transferring its backing shard. Do not assume that another collection previously colocated with it will move as well.

`moveCollection` does not change the document structure or logical shard key. The source must already be unsharded. Use [unsharding](sharding.md#unshard-a-collection) to remove a shard key, [resharding](sharding.md#reshard-a-collection) to change it, and the [rebalancer](rebalancer.md) to balance shard placements across the cluster.

## Implementation

See [wire protocol and common fields](../../diagnostic/wire-protocol.md) for the source revision.

The `moveCollection` wire API maps to the SQL UDF `documentdb_api_distributed.move_collection`. Its [SQL entry point](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/internal/pg_documentdb_distributed/sql/udfs/operations/move_collection--latest.sql) calls `documentdb_command_move_collection` in `internal/pg_documentdb_distributed/src/colocation/shard_colocation.c`. The implementation validates the namespace and destination, breaks colocation, moves the backing shard placement, and invalidates cached placement information.

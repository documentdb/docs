---
title: Distributed collMod colocation
description: Colocate DocumentDB collections using the collMod extension provided by documentdb_distributed.
type: commands
category: documentdb_distributed
---

# Distributed collMod colocation

With [documentdb_distributed and its prerequisites installed and loaded](overview.md#prerequisites-and-installation), `collMod.colocation` changes how unsharded collections are colocated. Colocated collections share a colocation group so their placement can be managed together.

This is an extension to the existing [`collMod`](../../administration/collMod.md) command, not a replacement for its index, validation, or statistics options.

## Colocate two collections

The example uses the [airline dataset](../../diagnostic/wire-protocol.md#airline-example-dataset). Both `tickets` and `flights` must exist in `bookings` and be unsharded; run this independently of the sharding examples:

```javascript
db.runCommand({
  collMod: "tickets",
  colocation: { collection: "flights" }
})
```

The source collection is `tickets`. The target name is resolved in the same logical database. The target must be a different unsharded collection with a single backing shard and a compatible colocation layout.

| Parameter | Description |
| --- | --- |
| **`collMod`** | (Required) The source collection name. |
| **`colocation`** | (Required for this operation) A document specifying the colocation change. |
| **`colocation.collection`** | (Required) A non-empty target collection name in the same database, or `null` to detach the source from its current colocation group. |

Colocation can involve redistributing backing tables. It does not merge the collections or change their document structure.

## Detach a collection

To stop colocating tickets with the target collection:

```javascript
db.runCommand({
  collMod: "tickets",
  colocation: { collection: null }
})
```

The `null` form is also available for detaching an already-sharded collection from its current group. A sharded collection cannot use the named-target form to join another collection's group.

Without the distributed hook, the default single-node implementation does not apply the colocation change.

## Distributed collection and placement metadata

The extension supplies metadata query hooks as well as colocation operations. Use `listCollections` to inspect a collection's colocation group and shard count, `config.chunks` for backing-shard placements, and the shard metadata query for node groups. These are metadata projections, not application collections to update directly.

### listCollections

Set `addDistributedMetadata: true` to include distributed metadata in each returned collection document. The following request uses the extension-specific option and filters the [airline dataset](../../diagnostic/wire-protocol.md#airline-example-dataset) to the two collections used above:

```javascript
db.runCommand({
  listCollections: 1,
  filter: { name: { $in: ["flights", "tickets"] } },
  addDistributedMetadata: true,
  cursor: { batchSize: 20 }
})
```

The command returns the usual `listCollections` cursor. These additional fields are at the top level of each collection document, not inside `info`:

| Field | Description |
| --- | --- |
| **`colocationId`** | The numeric colocation-group identifier for the collection's backing table. Collections in the same group have the same value. It is not a node-group name or a physical shard identifier. |
| **`shardCount`** | The backing shard count recorded for that colocation group. An unsharded collection can still have one backing shard; this is not the number of worker nodes or document count. |

After colocating `tickets` with `flights`, the relevant fields might look like this. The group identifier is illustrative:

```json
[
  { "name": "flights", "type": "collection", "colocationId": 42, "shardCount": 1 },
  { "name": "tickets", "type": "collection", "colocationId": 42, "shardCount": 1 }
]
```

The option defaults to `false`. With it enabled, the distributed query joins collections to their backing-table and colocation metadata, so views without backing tables are not included. `nameOnly: true` can be combined with this option: the distributed fields remain alongside `name` and `type`. Without the distributed hook, no distributed metadata is added. See [`listCollections`](../../administration/listCollections.md) for the other options.

### config.chunks

The distributed hook replaces the base chunk metadata with backing-shard and placement information:

```javascript
db.getSiblingDB("config").chunks.find({ ns: "bookings.flights" }).toArray()
```

| Field | Description |
| --- | --- |
| **`_id`** | The numeric physical shard identifier. |
| **`ns`** | The logical collection namespace, such as `bookings.flights`. |
| **`shard`** | The placement's node-group name, formatted as `shard_<groupid>`. This identifies a group, not the physical shard in `_id`. |
| **`chunkSize`** | The shard size in bytes reported by the backing storage, not a document count. |
| **`min`**, **`max`** | Shard-bound metadata projected by the distributed query, not ranges of the original BSON shard-key values. |

The query joins collection backing tables with shard, size, and placement metadata, excluding views. Unsharded collections with distributed backing storage can also appear. Each matching placement contributes a row, so do not interpret the row count as a count of distinct shards when a shard has multiple placements.

Use `ns` to relate these rows to a collection, and `shard` to relate placements to the node groups described below. Colocated collections retain distinct backing shards and `_id` values; sharing a `colocationId` does not merge their chunk records.

### config.shards

The extension's config-shard hook exposes the same group entries as [`listShards`](shard-discovery.md#listshards), replacing the base single-node placeholder. At the audited source revision, the backend routes this metadata through **`config._shards`**, not `config.shards`; do not assume that installing the extension creates a `config.shards` alias.

```javascript
db.getSiblingDB("config").getCollection("_shards").find({}).toArray()
```

| Field | Description |
| --- | --- |
| **`_id`** | The node-group name, formatted as `shard_<groupid>`, matching `config.chunks.shard`. |
| **`nodes`** | A string containing the group name, `/`, and comma-separated active member-node names, such as `shard_1/node_default_2`. This is not a connection string in a `host` field. |

The group list is built from nodes marked `shouldhaveshards` in the cluster metadata; active members are included in `nodes`. For per-node role, activity, and cluster information, see [`getShardMap`](shard-discovery.md#getshardmap).

### config.collections

`config.collections` describes logical collections with a configured shard key:

```javascript
db.getSiblingDB("config").collections.find({ _id: "bookings.flights" }).toArray()
```

| Field | Description |
| --- | --- |
| **`_id`** | The logical collection namespace, such as `bookings.flights`, matching `config.chunks.ns`. |
| **`key`** | The collection's configured BSON shard-key document. |
| **`noBalance`** | Always emitted as `true` by this metadata projection. It is not the distributed rebalancer's current status. |

At the audited source revision, the extension does not add a separate `config.collections` rewrite or distributed-only fields here. In particular, this query does not include `colocationId` or `shardCount`; request them with `listCollections.addDistributedMetadata` instead. The two unsharded collections in the colocation example do not appear in `config.collections` merely because the extension manages their storage. Use the [rebalancer APIs](rebalancer.md) to inspect actual rebalancing activity.

## Distributed index modifications

The extension also supplies the shard-index update hook used by supported `collMod` index operations. For example, [index visibility changes](../../administration/collMod.md#hide-and-unhide-an-index) update the backing shard indexes rather than only the coordinator's index metadata. The restrictions in the main `collMod` reference still apply.

## Implementation

See [wire protocol and common fields](../../diagnostic/wire-protocol.md) for the source revision.

Source: [distributed colocation](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/internal/pg_documentdb_distributed/src/colocation/shard_colocation.c), `HandleDistributedColocation`; shard indexes: `internal/pg_documentdb_distributed/src/distribution/distributed_index_operations.c`, `UpdateDistributedPostgresIndex`.

Metadata: `UpdateColocationHooks`, `RewriteListCollectionsQueryForDistribution`, `RewriteConfigChunksQueryForDistribution`, and `RewriteConfigShardsQueryForDistribution` in the distributed colocation source; base config queries: `pg_documentdb/src/aggregation/bson_aggregation_config_queries.c`, `GenerateConfigDatabaseQuery`, `GenerateCollectionsQuery`; list projection: `pg_documentdb/src/aggregation/bson_aggregation_metadata_queries.c`, `BuildListCollectionsProjection`.

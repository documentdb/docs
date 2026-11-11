---
title: Distributed shard discovery
description: Describe getShardMap and listShards responses implemented by documentdb_distributed.
type: commands
category: documentdb_distributed
---

# Distributed shard discovery

The [documentdb_distributed extension](overview.md) implements `getShardMap` and `listShards` responses from cluster node metadata. These describe node groups and their members, not the distribution of every document or collection.

Availability of these wire APIs is subject to provider gateway integration.

## getShardMap

```javascript
db.adminCommand({ getShardMap: 1 })
```

The command implementation has no command-specific argument parsing. It returns `ok` and, when eligible node groups exist, the following fields:

| Field | Description |
| --- | --- |
| **`map`** | A document mapping `shard_<groupid>` names to strings containing the group name and its active member-node names. |
| **`hosts`** | A document mapping each active member-node name to its shard-group name. |
| **`nodes`** | A document keyed by member-node name, with each node's `role`, `active`, and `cluster` values. Inactive members can appear here even though they are excluded from the active host mapping. |

For illustration, a deployment with one eligible group and one active primary member could return:

```json
{
  "ok": 1,
  "map": { "shard_1": "shard_1/node_default_2" },
  "hosts": { "node_default_2": "shard_1" },
  "nodes": {
    "node_default_2": { "role": "primary", "active": true, "cluster": "default" }
  }
}
```

These are illustrative identifiers, not required node names. A `shard_1` group name can identify a destination for [`moveCollection`](moveCollection.md), but confirm that the group has an active primary node before using it.

## listShards

See the dedicated [`listShards` command reference](listShards.md) for command syntax, response fields, an illustrative result, and gateway availability. Its group identifiers can be used to choose a `moveCollection` destination.

Both implementations select nodes marked `shouldhaveshards` in the cluster metadata. If no eligible nodes exist, the implementation can return only `{ ok: 1 }` without the mapping fields or array.

## Other distributed metadata

The extension's query hooks also add distributed collection metadata. `listCollections` with `addDistributedMetadata` includes `shardCount` and `colocationId`. The config-shard and config-chunk query hooks expose node-group and placement information for the distributed layout.

These hooks do not add a new wire command for each metadata field. See [distributed collection and placement metadata](colocation.md#distributed-collection-and-placement-metadata) for the fields, namespace availability, and `config.collections` behavior, and the [collection metadata reference](../../administration/listCollections.md) for existing command options.

## Implementation

See [wire protocol and common fields](../../diagnostic/wire-protocol.md) for the source revision.

The extension provides `command_get_shard_map` and `command_list_shards`. Its SQL installation scripts do not create standalone `get_shard_map` or `list_shards` functions; the provider gateway must integrate the distributed implementations.

Source: [distributed node and colocation code](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/internal/pg_documentdb_distributed/src/colocation/shard_colocation.c), `command_get_shard_map`, `command_list_shards`, `GetShardMapNodes`, `WriteShardMap`, `WriteShardList`, and `UpdateColocationHooks`.

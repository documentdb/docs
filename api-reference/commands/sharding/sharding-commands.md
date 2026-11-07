---
title: Sharding commands
description: Supported shardCollection, reshardCollection, unshardCollection, and getShardMap wire fields in DocumentDB.
type: commands
category: sharding
---

# Sharding commands

Sharding behavior and option support depend on the installed implementation.

For additional behaviors, see the [contrib section](../contrib/).

## shardCollection and reshardCollection

```javascript
sh.shardCollection("bookings.flights", { route_no: "hashed" })
sh.reshardCollection("bookings.flights", { departure_airport: "hashed" })
```

| Parameter | Description |
| --- | --- |
| **`shardCollection`** | (Required) A string specifying the fully qualified namespace, `database.collection`. |
| **`reshardCollection`** | (Required) A string specifying the fully qualified namespace, `database.collection`. |
| **`key`** | (Required) A document specifying the shard-key specification; the backend validates supported hashed shard-key forms. |
| **`numInitialChunks`** | (Optional, implementation-dependent) A positive integer requesting the initial number of chunks. Support and the maximum permitted count depend on the deployment's sharding implementation. |

On an implementation that supports `numInitialChunks`, either shell helper can pass the option:

```javascript
sh.shardCollection("bookings.flights", { route_no: "hashed" }, { numInitialChunks: 4 })
sh.reshardCollection("bookings.flights", { departure_airport: "hashed" }, { numInitialChunks: 4 })
```

Support for `numInitialChunks` depends on the installed sharding implementation, regardless of which client helper submits the request.

## unshardCollection

```javascript
db.adminCommand({ unshardCollection: "bookings.flights" })
```

Use a raw request to send only the namespace. The shell's `sh.unshardCollection()` helper requires an additional `toShard` argument.

| Parameter | Description |
| --- | --- |
| **`unshardCollection`** | (Required) A string specifying the fully qualified namespace. |

## getShardMap

```javascript
db.adminCommand({ getShardMap: 1 })
```

Returns deployment-provided shard metadata. The default wire adapter passes no command-specific options to its backend query.

## Other sharding commands

`enableSharding` is a no-op returning success; it does not change database state.

`balancerStart`, `balancerStatus`, `balancerStop`, `listShards`, and `moveCollection` can be implemented by some providers.

See [wire protocol and common fields](../diagnostic/wire-protocol.md) for the full dispatch support inventory.

## Implementation

The backend command-document entry points parse `numInitialChunks` for both `shardCollection` and `reshardCollection` and apply it when creating the distributed collection. The count must be positive and no greater than the configured sharding limit.

The default open-source gateway adapter uses a legacy backend call that passes only the namespace, `key`, and whether the request is a reshard. It does not forward `numInitialChunks`; deployments must use an implementation that forwards the full command document to honor this option. That adapter also does not forward `unique`, `presplitHashedZones`, `collation`, `forceRedistribution`, or `shardDistribution`.

Source: gateway `processor/data_description.rs`, `postgres/documentdb_data_client.rs`, and backend `pg_documentdb/src/sharding/sharding.c`, including `ParseShardCollectionRequest`, `ParseReshardCollectionRequest`, and `ParseUnshardCollectionRequest`.

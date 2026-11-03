---
title: Wire-command support and common fields
description: Source-correlated inventory of DocumentDB wire commands, shared options, compatibility responses, and provider-dependent commands.
type: commands
category: diagnostic
---

# Wire-command support and common fields

This reference describes the default open-source DocumentDB gateway and backend. Features guarded by server capabilities or cluster-version checks are not necessarily enabled in a released package or your deployment. Custom gateway/backend implementations can expose different behavior.

## Shell and wire syntax

```javascript
db.flights.find({ status: "Delayed" }).batchSize(20).maxTimeMS(5000)
```

Examples prefer mongosh helpers when an equivalent is available. The option descriptions cover the underlying wire command, not the helper's argument names or return shape. Some helpers use aggregation rather than the similarly named command. Raw `runCommand` or `adminCommand` requests are used for operations without an equivalent helper or for DocumentDB-specific options.

In raw requests, place the command name first. Drivers add `$db`; shell helpers translate arguments into command documents and can add session metadata automatically. A shell method such as `createIndex()` is not necessarily a wire command with that spelling.

## Airline example dataset

The examples use a small document-oriented adaptation of the [Postgres Pro demonstration database](https://postgrespro.com/education/demodb), whose publisher distributes it under the MIT license. The September 1, 2025 schema provides the airline, flight, route, airport, and ticket-segment concepts. The documents and values below are original illustrative fixtures, not rows copied from the downloadable database or live flight information.

The `bookings` database contains a `flights` collection. Each document combines a flight with its route information and embeds its ticket segments, so the examples do not require relational joins:

| Document field | Mapping |
| --- | --- |
| `_id` | Flight identifier, corresponding to `flights.flight_id`. |
| `route_no`, `departure_airport`, `arrival_airport` | Route information stored with the flight, following the source's `timetable` view. |
| `scheduled_departure`, `scheduled_arrival` | BSON dates created with `ISODate()`, rather than date strings. |
| `status` | A flight status such as `Scheduled` or `Delayed`. |
| `segments` | Embedded ticket segments with `ticket_no`, `fare_conditions`, and `price`. The parent document identifies their flight. |
| `delay_minutes`, `seat_capacity`, `route_vector` | Example-specific additions for numeric updates, aggregation, and a gated vector index; these are not columns of the source's `flights` table. |

The airport insertion examples use an `airports` collection, with each source `airport_code` mapped to the document's `_id`.

Run this setup once in an empty `bookings.flights` collection:

```javascript
db = db.getSiblingDB("bookings")
db.flights.insertMany(
  [
    {
      _id: 101, route_no: "PG0101", departure_airport: "SVO", arrival_airport: "LED",
      scheduled_departure: ISODate("2025-09-01T08:00:00Z"),
      scheduled_arrival: ISODate("2025-09-01T09:30:00Z"),
      seat_capacity: 160, delay_minutes: 0, status: "Scheduled",
      segments: [
        { ticket_no: "0005432000101", fare_conditions: "Economy", price: 12000 },
        { ticket_no: "0005432000102", fare_conditions: "Economy", price: 8000 }
      ],
      route_vector: [1, 0, 0]
    },
    {
      _id: 102, route_no: "PG0102", departure_airport: "SVO", arrival_airport: "KZN",
      scheduled_departure: ISODate("2025-09-01T09:00:00Z"),
      scheduled_arrival: ISODate("2025-09-01T10:30:00Z"),
      seat_capacity: 120, delay_minutes: 12, status: "Delayed",
      segments: [{ ticket_no: "0005432000103", fare_conditions: "Business", price: 18000 }],
      route_vector: [0, 1, 0]
    },
    {
      _id: 103, route_no: "PG0103", departure_airport: "LED", arrival_airport: "KZN",
      scheduled_departure: ISODate("2025-09-01T10:00:00Z"),
      scheduled_arrival: ISODate("2025-09-01T12:00:00Z"),
      seat_capacity: 100, delay_minutes: 0, status: "Scheduled",
      segments: [{ ticket_no: "0005432000104", fare_conditions: "Business", price: 10000 }],
      route_vector: [0, 0, 1]
    },
    {
      _id: 104, route_no: "PG0104", departure_airport: "SVO", arrival_airport: "LED",
      scheduled_departure: ISODate("2025-09-01T18:00:00Z"),
      scheduled_arrival: ISODate("2025-09-01T19:30:00Z"),
      seat_capacity: 160, delay_minutes: 5, status: "Delayed",
      segments: [{ ticket_no: "0005432000105", fare_conditions: "Economy", price: 12000 }],
      route_vector: [1, 0, 0]
    }
  ],
  { ordered: true }
)
```

Select `bookings` as the current database for collection examples. Each independent update or deletion starts from these four documents. Additional airport, index, role, or temporary-collection setup is identified on the relevant page. Diagnostic and authentication commands that do not access collections have no dataset prerequisite. No SQL import is needed.

## Common request fields

| Parameter | Description |
| --- | --- |
| **`$db`** | (Required) A string specifying the target logical database; supplied by the driver. |
| **`maxTimeMS`** | (Optional) A number specifying the request execution budget in milliseconds. `0` does not set an explicit timeout. Whether the backend applies a statement timeout depends on the command execution path; it is not a universal control over compatibility handlers. |
| **`lsid`** | (Optional) A document specifying the logical session ID, normally `{ id: <BSON UUID> }`. Used for cursor/session ownership and transaction resolution. |
| **`txnNumber`** | (Optional) A BSON int64 specifying the transaction/retry number. Use the driver's BSON int64 type, not a 32-bit integer or double. |
| **`autocommit`** | (Optional) A boolean that identifies a multi-command transaction when `false`. |
| **`startTransaction`** | (Optional) A boolean that starts a transaction on its first operation when `true`. |
| **`readConcern`** | (Optional) A document or null specifying the read isolation request; subject to the constraints below. |
| **`comment`** | (Optional) A string containing a JSON object with `traceparent` and optional `tracestate`. When a tracing provider is configured and the context is valid, the gateway uses it as the request's trace parent. |

Common metadata is handled before command execution. Individual commands can impose additional restrictions; consult their parameter descriptions.

A multi-command transaction requires `lsid` and a BSON int64 `txnNumber`, with `autocommit: false`. Set `startTransaction: true` only on its first operation. Subsequent operations must use matching session and transaction identifiers.

### Read concern

With read-concern validation enabled, supported `level` strings are `local`, `available`, `majority`, `linearizable`, and `snapshot`. `snapshot` selects PostgreSQL repeatable-read isolation for the transaction; other recognized read concerns are mapped to PostgreSQL's Read Committed snapshot, which provides a consistent view of committed data for each statement.

| Parameter | Description |
| --- | --- |
| **`level`** | (Optional) A string naming a recognized level, or `null` for an unspecified level. Unknown or differently cased strings are rejected. |

Unknown read-concern subfields and invalid types fail. Legacy configurations with validation disabled parse more permissively.

## Routed command inventory

The following inventory covers public commands with explicit dispatcher branches. Authentication commands are handled before ordinary command execution.

| Commands | Reference and default behavior |
| --- | --- |
| `find` | [Query and cursor options](../query-and-write/find.md). |
| `insert` | [Insert batch options](../query-and-write/insert.md). |
| `update` | [Command and per-update options](../query-and-write/update.md). |
| `delete` | [Command and per-delete options](../query-and-write/delete.md). |
| `findAndModify` | [Atomic modification options](../query-and-write/findandmodify.md). |
| `getMore` | [Cursor continuation](../query-and-write/getMore.md). |
| `killCursors` | [Cursor termination](../query-and-write/killCursors.md). |
| `aggregate` | [Pipeline command options](../aggregation/aggregate.md). |
| `count` | [Count options](../aggregation/count.md). |
| `distinct` | [Distinct options](../aggregation/distinct.md). |
| `create` | [Collection and view creation](../administration/create.md). |
| `collMod` | [Collection, view, and index modifications](../administration/collMod.md). |
| `drop`, `dropDatabase` | [Collection and database removal](../administration/drop.md). |
| `renameCollection` | [Same-database renaming](../administration/renameCollection.md). |
| `listCollections` | [Collection metadata cursor](../administration/listCollections.md). |
| `listDatabases` | [Database listing](../administration/listDatabases.md). |
| `createIndex`, `createIndexes` | [Index creation](../indexing/createIndexes.md). |
| `listIndexes` | [Index metadata cursor](../indexing/listIndexes.md). |
| `dropIndexes` | [Index removal](../indexing/dropIndexes.md). |
| `reIndex` | Can be implemented by some providers. To rebuild indexes, use [`collMod.index.reindex`](../administration/collMod.md). |
| `createSearchIndexes` | Can be implemented by some providers. For vector indexes, use [`createIndexes`](../indexing/createIndexes.md). |
| `explain` | [Supported reads and verbosity](explain.md). |
| `collStats` | [Collection statistics](collStats.md). |
| `dbStats` | [Database statistics](dbStats.md). |
| `currentOp` | [Operation inspection](currentOp.md). |
| `killOp` | [Operation cancellation](killOp.md). |
| `compact` | [Maintenance modes and dry runs](compact.md). |
| `validate` | [Collection validation flags](validate.md). |
| `createUser`, `updateUser`, `dropUser`, `usersInfo`, `grantRolesToUser`, `revokeRolesFromUser` | [Gated user operations](../security/user-commands.md). |
| `createRole`, `dropRole`, `rolesInfo`, `grantRolesToRole`, `revokeRolesFromRole` | [Gated role operations](../security/role-commands.md). |
| `updateRole`, `grantPrivilegesToRole`, `revokePrivilegesFromRole` | [Provider-dependent role mutations](../security/role-commands.md#provider-dependent-role-mutations). |
| `killSessions` | [Session termination](../sessions/session-commands.md). |
| `commitTransaction`, `abortTransaction` | [Transaction commands](../sessions/transaction-commands.md). |
| `shardCollection`, `reshardCollection`, `unshardCollection`, `getShardMap` | [Sharding fields](../sharding/sharding-commands.md). |
| `enableSharding` | [No-op database sharding marker](../sharding/sharding-commands.md#other-sharding-commands). |
| `balancerStart`, `balancerStatus`, `balancerStop`, `listShards`, `moveCollection` | [Can be implemented by some providers](../sharding/sharding-commands.md#other-sharding-commands). |

Wire aliases explicitly recognized by the name parser are `buildinfo` for `buildInfo`, `dbstats` for `dbStats`, `findandmodify` for `findAndModify`, `ismaster` for `isMaster`, `reindex` for `reIndex`, and `dbhash` for `dbHash`. Backend-only spellings do not automatically become working gateway aliases.

## SDK and driver commands

These commands primarily support SDK and driver handshakes, compatibility checks, and session cleanup.

| Commands | Reference and default behavior |
| --- | --- |
| `hello`, `isMaster` | [Handshake fields](hello.md). |
| `ping`, `buildInfo`, `getCmdLineOpts`, `getLog`, `hostInfo`, `isdbgrid`, `whatsmyuri`, `getDefaultRWConcern` | [Compatibility responses and their limitations](compatibility.md). |
| `endSessions` | [Session cleanup](../sessions/session-commands.md). |

## Additional provider-dependent commands

The following 127 additional command names are recognized by the request-type parser. Their implementations and option support depend on the provider. A recognized or advertised name does not by itself guarantee command availability.

### Administrative, query, and legacy commands

`appendOplogNote`, `applyOps`, `authSchemaUpgrade`, `authenticate`, `availableQueryOptions`, `checkShardingIndex`, `cleanupOrphaned`, `clone`, `cloneCollection`, `cloneCollectionAsCapped`, `connPoolStats`, `connPoolSync`, `convertToCapped`, `copydb`, `copydbgetnonce`, `copydbsaslstart`, `createUniqueOrCompoundIndexOnNestedPath`, `currentOpCtx`, `customAction`, `dataSize`, `dbHash`, `diagLogging`, `driverOIDTest`, `dropAllRolesFromDatabase`, `dropAllUsersFromDatabase`, `eval`, `features`, `filemd5`, `forceerror`, `fsync`, `fsyncUnlock`, `geoNear`, `geoSearch`, `getDatabaseVersion`, `getDiagnosticData`, `getFreeMonitoringStatus`, `getLastError`, `getLastRequestStatistics`, `getNonce`, `getPrevError`, `getShardVersion`, `group`, `handshake`, `invalidateUserCache`, `lockInfo`, `logRotate`, `mapReduce`, `mapreduce.shardedfinish`, `mergeChunks`, `moveChunk`, `parallelCollectionScan`, `planCacheListPlans`, `planCacheListQueryShapes`, `profile`, `reSync`, `refreshSessionsInternal`, `repairCursor`, `repairDatabase`, `resetError`, `setFeatureCompatibilityVersion`, `setFreeMonitoring`, `setParameter`, `setShardVersion`, `shardConnPoolStats`, `shardingState`, `shutdown`, `splitChunk`, `splitVector`, `startSession`, `top`, `touch`, `unsetSharding`, and `writebacklisten`.

`getnonce` is advertised by `listCommands`, but the name parser recognizes `getNonce`. Check the provider's command spelling and availability.

### Replica-set commands

`replSetAbortPrimaryCatchUp`, `replSetDeclareElectionWinner`, `replSetElect`, `replSetFreeze`, `replSetFresh`, `replSetGetConfig`, `replSetGetRBID`, `replSetGetStatus`, `replSetHeartbeat`, `replSetInitiate`, `replSetMaintenance`, `replSetReconfig`, `replSetRequestVotes`, `replSetResizeOplog`, `replSetStepDown`, `replSetStepUp`, `replSetSyncFrom`, and `replSetUpdatePosition`.

### Internal protocol commands

`_cloneCatalogData`, `_cloneCollectionOptionsFromPrimaryShard`, `_configsvrAddShard`, `_configsvrAddShardToZone`, `_configsvrBalancerStart`, `_configsvrBalancerStatus`, `_configsvrBalancerStop`, `_configsvrCommitChunkMerge`, `_configsvrCommitChunkMigration`, `_configsvrCommitChunkSplit`, `_configsvrCommitMovePrimary`, `_configsvrCreateCollection`, `_configsvrCreateDatabase`, `_configsvrDropCollection`, `_configsvrDropDatabase`, `_configsvrEnableSharding`, `_configsvrMoveChunk`, `_configsvrMovePrimary`, `_configsvrRemoveShard`, `_configsvrRemoveShardFromZone`, `_configsvrShardCollection`, `_configsvrUpdateZoneKeyRange`, `_flushDatabaseCacheUpdates`, `_flushRoutingTableCacheUpdates`, `_getNextSessionMods`, `_getUserCacheGeneration`, `_isSelf`, `_mergeAuthzCollections`, `_migrateClone`, `_movePrimary`, `_recvChunkAbort`, `_recvChunkCommit`, `_recvChunkStart`, `_recvChunkStatus`, `_shardsvrShardCollection`, and `_transferMods`.

Command availability requires both name recognition and an executable implementation. Check your provider's supported commands.

## Source correlation

All source paths mentioned in command pages refer to the revision above. Gateway paths are relative to `pg_documentdb_gw/documentdb_gateway_core/src/`.

| Source | Used to verify |
| --- | --- |
| [Gateway dispatch](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/pg_documentdb_gw/documentdb_gateway_core/src/processor/process.rs) and [command-name parsing](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/pg_documentdb_gw/documentdb_gateway_core/src/requests/request_type.rs) | Command dispatch, recognized names, and aliases. |
| [Default PostgreSQL adapter](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/pg_documentdb_gw/documentdb_gateway_core/src/postgres/documentdb_data_client.rs) | Fields forwarded to backend implementations and provider-dependent command entry points. |
| [Shared request parsing](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/pg_documentdb_gw/documentdb_gateway_core/src/requests/common_fields.rs) | Session, transaction, timeout, read-concern, and annotation metadata. |
| [Common backend fields](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/pg_documentdb/src/commands/commands_common.c) | Fallback parsing of shared options. |
| [Read parsers](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/pg_documentdb/src/aggregation/bson_aggregation_pipeline.c) and [metadata-query parsers](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/pg_documentdb/src/aggregation/bson_aggregation_metadata_queries.c) | Read and metadata command fields, types, and feature gates. |
| [Backend command implementations](https://github.com/documentdb/documentdb/tree/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/pg_documentdb/src/commands) | Write, index, administration, diagnostics, user, and role parsers. |

The scope is top-level wire commands and their option documents, not a new audit of every aggregation stage, expression operator, PostgreSQL API, or deployment-specific extension.

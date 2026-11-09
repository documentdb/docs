---
title: compact
description: Run collection maintenance, update statistics, or estimate reclaimable storage in DocumentDB.
type: commands
category: diagnostic
---

# compact

Runs collection maintenance or estimates reclaimable space.

```javascript
db.runCommand({ compact: "flights", dryRun: true })
db.runCommand({ compact: "flights", mode: "updateStats" })
```

| Parameter | Description |
| --- | --- |
| **`compact`** | (Required) A string specifying the collection name. |
| **`dryRun`** | (Optional) A boolean that returns `estimatedBytesFreed` without compacting; defaults to `false`. Cannot be combined with `mode: "updateStats"`. |
| **`freeSpaceTargetMB`** | (Optional) A number that skips `standard` or `full` compaction when the estimated reclaimable space is below this positive threshold in MB. Does not limit the amount reclaimed; has no effect on `updateStats`. |
| **`mode`** | (Optional) A string provided as a DocumentDB extension: `standard` (default) runs ordinary vacuum, `full` requests a blocking table rewrite, and `updateStats` refreshes planner statistics. |
| **`maxTimeMS`** | (Optional) A number specifying the command execution timeout in milliseconds. Set an explicit budget for long-running maintenance; the default and any enforced limits depend on the deployment. |

Full compaction requires the server's full-vacuum capability to be enabled. When disabled, `mode: "full"` returns success with `bytesFreed: 0` without rewriting the collection. Dry runs and standard maintenance do not require that capability. The operation must pass the collection's maintenance authorization checks.

## Examples

The examples use the `bookings.flights` collection from the [airline dataset](wire-protocol.md#airline-example-dataset). The command has no dedicated shell helper.

### Refresh planner statistics with updateStats

After a bulk load or substantial changes to the collection, refresh the statistics used for query planning:

```javascript
db.runCommand({ compact: "flights", mode: "updateStats" })
```

This mode runs PostgreSQL `ANALYZE` on the collection's backing table. It updates statistics used to estimate query selectivity and choose execution plans; it does not rewrite the collection or reclaim storage. A successful response is:

```json
{ "ok": 1, "bytesFreed": 0 }
```

Here, `bytesFreed: 0` is expected and does not mean that statistics were unchanged. Refreshing statistics does not guarantee a particular index or execution plan.

`updateStats` does not enable per-collection planner statistics. To enable that capability for a collection before refreshing its statistics, see [`collMod.enableStats`](../administration/collMod.md#enable-planner-statistics). It does not require the full-vacuum capability, but still requires maintenance authorization and can conflict with other maintenance operations.

Do not combine `updateStats` with `dryRun: true`: this combination is invalid. `freeSpaceTargetMB` does not apply because this mode does not compact storage.

### Reclaim storage with full

Full compaction rewrites the collection's backing table using PostgreSQL `VACUUM FULL`. Unlike the default `standard` mode, which primarily makes space from deleted or obsolete records reusable inside the table, a full rewrite can reduce the collection's storage footprint.

First estimate the reclaimable space without performing the rewrite:

```javascript
db.runCommand({ compact: "flights", mode: "full", dryRun: true })
```

The response contains `estimatedBytesFreed`. This is an estimate, not a promise of how much storage a later rewrite will reclaim. The dry run works even when the full-vacuum capability is disabled.

When that capability is enabled, request the rewrite:

```javascript
db.runCommand({ compact: "flights", mode: "full" })
```

Full compaction takes an exclusive lock that blocks reads and writes on the collection while the rewrite runs. Schedule it during a maintenance window. If a conflicting operation is already in progress, the command can fail rather than start the rewrite.

The response contains `bytesFreed`. If the full-vacuum capability is disabled, the command instead returns `{ ok: 1, bytesFreed: 0 }` without rewriting anything. A zero value alone therefore does not prove that a full rewrite ran.

Alternatively, request a rewrite only when at least 64 MB is estimated to be reclaimable:

```javascript
db.runCommand({
  compact: "flights",
  mode: "full",
  freeSpaceTargetMB: 64
})
```

The threshold is checked before compaction. If the estimate is unavailable or below the threshold, the command skips the rewrite and returns `bytesFreed: 0`. It is not a limit on the amount of space reclaimed.

### Allow more time for a full rewrite

A full rewrite of a large collection can take longer than two minutes. Instead of relying on the deployment's default timeout, allow up to 30 minutes in this alternative to the preceding full-compaction examples:

```javascript
db.runCommand({
  compact: "flights",
  mode: "full",
  maxTimeMS: 1800000
})
```

Choose the budget for the collection size and maintenance window; 30 minutes is an example, not a guarantee of completion or a new default. A longer timeout does not reduce the exclusive-lock duration. Client socket timeouts and provider gateway limits must also permit the requested duration.

### Monitor maintenance and investigate a timeout

From a **separate shell connection**, inspect maintenance for the same collection:

```javascript
db.currentOp({
  $all: true,
  ns: "bookings.flights",
  "command.compact": "flights"
})
```

Repeat the query while compact is running, or reconnect and run it after the original request times out. The result's `inprog` entries can include the command wrapper and the underlying maintenance operations. See [compact progress fields](currentOp.md#monitor-compact-and-statistics-maintenance) for `progress.phase`, block counters, `msg`, and lock-wait information.

Compact is synchronous, not an asynchronously queued job. A server-side timeout can cancel the command and its maintenance connection; a client-side timeout or lost connection alone does not establish whether the server has stopped. If a matching maintenance entry remains, monitor it rather than immediately submitting another compact request. If no entry remains, `currentOp` cannot tell whether the operation finished successfully or was cancelled, and cannot recover the original `bytesFreed` response. Use the original command response, when available, and provider diagnostics to determine the outcome.

## Implementation

`updateStats` runs `ANALYZE`, `standard` runs `VACUUM`, and `full` runs `VACUUM FULL`. On cluster version 1.0 or later, storage-reclamation reporting compares relation-file sizes before and after vacuum; older cluster versions use estimate-based reporting.

See [wire protocol and common fields](wire-protocol.md). Source: `pg_documentdb/src/commands/compact.c`, `ParseCompactCommandSpec`, `command_compact`, `PerformCompactOperation`, and `RunVacuumAndMeasureFreedSpace`.

Timeout handling: gateway `postgres/documentdb_data_client.rs`, `execute_compact`, and `postgres/conn_mgmt/query_dispatch.rs`; maintenance-connection cancellation: `pg_documentdb/src/infrastructure/connection_management.c`. Progress reporting: `pg_documentdb/src/commands/current_op.c`, `WriteMaintenanceProgressAndGetMessage`.

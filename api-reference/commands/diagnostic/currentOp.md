---
title: currentOp
description: Inspect active operations in DocumentDB using supported currentOp fields and filters.
type: commands
category: diagnostic
---

# currentOp

Returns operation information in `inprog`, including index-build and collection-maintenance progress.

```javascript
db.currentOp({ $all: true, active: true, ns: "bookings.flights" })
```

Mongosh implements `currentOp()` with an aggregation and returns the results in `inprog`. The option descriptions below cover the direct wire command; its `$all` handling is not the shell helper's aggregation-option translation.

| Parameter | Description |
| --- | --- |
| **`currentOp`** | (Required) A command marker, conventionally `1`. |
| **`$all`** | (Optional) A boolean-like value that sets the underlying `allUsers` selection; defaults to `false`. |
| **Other non-common fields** | (Optional) A query predicate that filters operation documents, for example `active: true` or `ns: "bookings.flights"`. |

The command constructs a current-operation query that includes idle connections and sessions before filtering. The [`$currentOp`](../../operators/aggregation/$currentop.md) aggregation stage has a separate option document.

## Monitor index builds

While an index is being built, run this from another shell connection:

```javascript
db.currentOp({
  $all: true,
  ns: "bookings.flights",
  "command.createIndexes": { $exists: true }
})
```

This includes index-creation operations for the [airline example dataset](wire-protocol.md#airline-example-dataset). Do not add `active: true` when investigating queued or retrying builds: those entries can report `active: false`. Use `command.indexes`, when present, to identify the index name and key pattern.

An illustrative excerpt from one entry in `inprog` is:

```json
{
  "ns": "bookings.flights",
  "active": true,
  "command": {
    "createIndexes": "flights",
    "indexes": [{ "key": { "route_no": 1 }, "name": "route_no_1" }],
    "$db": "bookings"
  },
  "progress": {
    "builds": [
      {
        "phase": "Scanning Table (building index).",
        "concurrent": true,
        "blocks_done": 400,
        "blocks_total": 1000,
        "Progress": 40,
        "terms_done": 0,
        "terms_total": 0
      }
    ]
  }
}
```

### Read the progress fields

| Field | Description |
| --- | --- |
| **`progress.builds`** | An array of reported build-progress records. Inspect each record; multiple records or operations can participate in a distributed build. |
| **`phase`** | The current stage, using the [translated phase messages](#index-build-phase-messages) below. Available phases depend on the index and build path. |
| **`concurrent`** | Whether the reported build uses concurrent index creation. This is not a completion indicator. |
| **`blocks_done`, `blocks_total`** | Completed and total blocks reported for the current phase. These are not document counts. |
| **`Progress`** | Percentage of the current phase's reported block work completed: `100 * blocks_done / blocks_total`. The capital `P` is significant. This field is omitted when `blocks_total` is zero. |
| **`terms_done`, `terms_total`** | Completed and total tuples/terms reported by the index build for the current phase, not necessarily the number of documents in the collection. |
| **`terms_progress`** | Percentage of the current phase's reported tuple/term work completed: `100 * terms_done / terms_total`. Omitted when `terms_total` is zero, as in the example above. |
| **`msg`** | An operation-level summary. It can describe build phases, a queued build, or a failed build awaiting retry. |
| **`waitingForLock`** | An operation-level indication that the operation is waiting on a lock or related synchronization event. Check this and `msg` when counters stop advancing. |

In the example, `400 / 1000` means the reported block work is **40% complete for that phase**, not that the whole index build is 40% complete. The zero term total means no term percentage is available.

Reaching `100%` means the reported work for that phase is complete, not that the entire index build is complete. The build may still need to sort, validate, or wait in subsequent phases. Counters and percentages can reset or change meaning between phases; do not sum percentages across records or use them as an overall completion estimate.

Compare `phase`, `msg`, and counters across repeated snapshots to follow the build through its stages. Detailed progress can be absent or empty when counters are unavailable; queued and failed/retrying entries can have only a status message. A waiting phase or a missing percentage does not by itself indicate failure.

An entry disappearing from `currentOp` is not proof of success. For [index-creation calls](../indexing/createIndexes.md) that wait for completion, use the call's final result to confirm that the build succeeded.

### Index-build phase messages

`progress.builds[].phase` contains a translated message, rather than the underlying phase name. These messages can also appear in the operation-level `msg`. The table follows an approximate chronological order, not a required sequence: stages depend on the index and build path, and some are skipped. Concurrent-command waits can recur between stages; aborting is an alternative path, not the final step of a successful build.

| Reported phase | Description |
| --- | --- |
| **`Initializing index.`**, **`Initializing index`** | Preparing the index build. Both punctuation variants are returned by the implementation for different initialization phases. |
| **`Index is waiting for other concurrent commands.`** | Waiting for concurrent activity before building or between later stages, including worker coordination. Counters can remain unchanged while waiting. |
| **`Building index.`** | Building the index; no more specific build stage is reported. |
| **`Scanning Table (building index).`** | Scanning the collection's underlying table as part of the build. |
| **`Concurrently Sorting tuples (building index).`** | Worker processes are sorting tuples concurrently. |
| **`Concurrently merging tuples on workers (building index).`** | Worker processes are merging tuples concurrently. |
| **`Sorting tuples (building index).`** | Sorting tuples for the index build. |
| **`Merging tuples into index (building index).`** | Merging tuples into the index. |
| **`Writing WAL files (building index).`** | Writing write-ahead log (WAL) files for the index build. |
| **`Finalizing index build.`** | Finalizing the index-building work, before validation where applicable; not yet a confirmation that the index-creation request succeeded. |
| **`Validating index.`** | Validating the index after earlier build phases have completed. |
| **`Index is waiting for commands to reach snapshot threshold.`** | After concurrent index validation, waiting for old snapshots held by other commands before the index can be marked valid. |
| **`Cleaning up.`** | Performing cleanup work. |
| **`Concurrent cleanup.`** | Performing concurrent cleanup work. |
| **`Finalizing.`** | Performing finalization work. |
| **`Aborting.`** | Aborting the operation; this is not a successful completion state. |

## Monitor compact and statistics maintenance

Use another shell connection to monitor [`compact`](compact.md), including `mode: "full"` and `mode: "updateStats"`:

```javascript
db.currentOp({
  $all: true,
  ns: "bookings.flights",
  "command.compact": "flights"
})
```

The filter includes underlying maintenance operations as well as the compact command wrapper. The wrapper can have no progress counters; inspect the maintenance entries for progress. Availability depends on the installed implementation and the caller's visibility into operations.

An illustrative excerpt from a full rewrite scanning the backing table is:

```json
{
  "ns": "bookings.flights",
  "active": true,
  "secs_running": 180,
  "op": "workerCommand",
  "command": { "compact": "flights", "mode": "full" },
  "waitingForLock": false,
  "progress": {
    "phase": "seq scanning heap",
    "blocks_done": 400,
    "blocks_total": 1000,
    "Progress": 40
  },
  "msg": "Vacuum progress: seq scanning heap."
}
```

Unlike index builds, maintenance counters are directly under `progress`, not in `progress.builds`.

| Field | Description |
| --- | --- |
| **`command.compact`** | The collection being maintained. The underlying vacuum and analyze operations are reported as compact operations. |
| **`command.mode`** | `full` for a recognized full rewrite or `updateStats` for analyze. Ordinary vacuum can omit this field; absence does not mean that no maintenance is running. |
| **`ns`** | The logical database and collection namespace. |
| **`opid`** | The operation identifier, when reported. Distinct wrapper or worker entries can have different identifiers. |
| **`active`** | Whether the operation is active. An active operation can be waiting; compare phases and counters across snapshots to check its progress. |
| **`secs_running`** | Elapsed execution time in seconds for this operation. |
| **`progress.phase`** | The current maintenance phase, using the underlying phase text. When no progress row is available yet, this can be `waiting to start`. |
| **`progress.blocks_done`** | Heap blocks scanned for vacuum/full rewrite, or sample blocks scanned for analyze. This is not a count of documents rewritten or deleted. |
| **`progress.blocks_total`** | Total heap blocks for vacuum/full rewrite, or total sample blocks for analyze. It can be zero before counters are available. |
| **`progress.Progress`** | Percentage of the scan work completed: `100 * blocks_done / blocks_total`, omitted when the total is zero. The capital `P` is significant. At `100%`, the scan phase is complete; other maintenance phases can remain. |
| **`msg`** | A summary such as `Vacuum progress: seq scanning heap.` or `Analyze progress: acquiring sample rows.` |
| **`waitingForLock`** | Whether the operation is waiting on a reported lock or related synchronization event. Inspect this when counters stop advancing. |
| **`locking_op_prefixes`** | Identifiers of blocking operations, when available for a waiting operation. |
| **`writeConflicts`** | The number of blocking processes reported with the lock-wait information, not a cumulative count of write failures. |

Phases depend on the maintenance mode. A full rewrite can proceed from scanning to writing the new heap, rebuilding indexes, and swapping relation files. Scan progress reaching `100%` means the scan phase is complete, not that the entire compact operation is complete; later phases may still be running. These counters provide neither an overall completion estimate nor an estimate of time remaining. Analyze reports sample-block progress rather than a full collection scan. Distributed maintenance can produce multiple entries; do not sum their percentages.

Use the compact command's successful final response to confirm completion of the entire operation. After a client timeout, reconnect and use the same query to see whether maintenance remains active. A timeout does not guarantee that maintenance continues: cancellation can stop the operation. An entry disappearing can mean success, failure, or cancellation; `currentOp` does not retain a completed compact result or its `bytesFreed` value. See [extending compact timeouts and investigating their outcome](compact.md#allow-more-time-for-a-full-rewrite).

## Implementation

Index-build counters come from PostgreSQL's index-creation progress reporting. The implementation translates phase messages and groups the reported counters into `progress.builds`; queued and failed/retrying background builds are also included in the operation output.

The generic translation for phase names containing `waiting for` runs before the worker-specific waiting cases. Consequently, worker join, start, finish, abort, finalize, and cleanup waits currently use `Index is waiting for other concurrent commands.`, rather than distinct worker-wait messages.

See [wire protocol and common fields](wire-protocol.md). Source: `pg_documentdb/src/commands/current_op.c`, `BuildAggregateSpecFromCommandSpec`, `WriteIndexBuildProgressAndGetMessage`, `PhaseToUserMessage`, and `AddIndexBuilds`.

Maintenance reporting uses `MaintenanceProgressQuery`, `WriteMaintenanceProgressAndGetMessage`, and the vacuum/analyze command mapping in the same source file. It reads `pg_stat_progress_vacuum` for ordinary vacuum, `pg_stat_progress_cluster` for full rewrites, and `pg_stat_progress_analyze` for statistics refreshes.

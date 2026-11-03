---
title: currentOp
description: Inspect active operations in DocumentDB using supported currentOp fields and filters.
type: commands
category: diagnostic
---

# currentOp

Returns operation information in `inprog`, including index-build progress.

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
| **`Progress`** | Block-based percentage: `100 * blocks_done / blocks_total`. The capital `P` is significant. This field is omitted when `blocks_total` is zero. |
| **`terms_done`, `terms_total`** | Completed and total tuples/terms reported by the index build for the current phase, not necessarily the number of documents in the collection. |
| **`terms_progress`** | Tuple/term-based percentage: `100 * terms_done / terms_total`. Omitted when `terms_total` is zero, as in the example above. |
| **`msg`** | An operation-level summary. It can describe build phases, a queued build, or a failed build awaiting retry. |
| **`waitingForLock`** | An operation-level indication that the operation is waiting on a lock or related synchronization event. Check this and `msg` when counters stop advancing. |

In the example, `400 / 1000` means the reported block work is **40% complete for that phase**, not that the whole index build is 40% complete. The zero term total means no term percentage is available.

Counters and percentages can reset or change meaning as the build moves between phases. A phase reaching `100%` can still be followed by sorting, validation, or waiting. Do not sum percentages across records or use them as an overall completion estimate.

Detailed progress can be absent or empty when counters are unavailable; queued and failed/retrying entries can have only a status message. A waiting phase or a missing percentage does not by itself indicate failure. Check `phase`, `msg`, and repeated snapshots together.

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
| **`Validating index.`** | Validating the index. This can follow earlier build work reaching `100%`. |
| **`Index is waiting for commands to reach snapshot threshold.`** | After concurrent index validation, waiting for old snapshots held by other commands before the index can be marked valid. |
| **`Cleaning up.`** | Performing cleanup work. |
| **`Concurrent cleanup.`** | Performing concurrent cleanup work. |
| **`Finalizing.`** | Performing finalization work. |
| **`Aborting.`** | Aborting the operation; this is not a successful completion state. |

## Implementation

Index-build counters come from PostgreSQL's index-creation progress reporting. The implementation translates phase messages and groups the reported counters into `progress.builds`; queued and failed/retrying background builds are also included in the operation output.

The generic translation for phase names containing `waiting for` runs before the worker-specific waiting cases. Consequently, worker join, start, finish, abort, finalize, and cleanup waits currently use `Index is waiting for other concurrent commands.`, rather than distinct worker-wait messages.

See [wire protocol and common fields](wire-protocol.md). Source: `pg_documentdb/src/commands/current_op.c`, `BuildAggregateSpecFromCommandSpec`, `WriteIndexBuildProgressAndGetMessage`, `PhaseToUserMessage`, and `AddIndexBuilds`.

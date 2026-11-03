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
| **`freeSpaceTargetMB`** | (Optional) A number that skips compaction when the estimated reclaimable space is below this positive threshold in MB. |
| **`mode`** | (Optional) A string provided as a DocumentDB extension: `standard` (default) runs ordinary vacuum, `full` requests a blocking table rewrite, and `updateStats` refreshes planner statistics. |

Full compaction requires the server's full-vacuum capability to be enabled. When disabled, `mode: "full"` returns success with `bytesFreed: 0` without rewriting the collection. Dry runs and standard maintenance do not require that capability. The operation must pass the collection's maintenance authorization checks.

See [wire protocol and common fields](wire-protocol.md). Source: `pg_documentdb/src/commands/compact.c`, `ParseCompactCommandSpec` and `command_compact`.

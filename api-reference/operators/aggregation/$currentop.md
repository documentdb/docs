---
title: $currentOp
description: The $currentOp stage returns information on active and queued operations for the database.
type: operators
category: aggregation
---

# $currentOp

The `$currentOp` stage returns a stream of documents containing information on active and queued operations for the database instance. This stage must be the first stage in the pipeline and is run on the `admin` database.

## Syntax

```javascript
db.adminCommand({
  aggregate: 1,
  pipeline: [
    {
      $currentOp: {
        allUsers: <boolean>,
        idleConnections: <boolean>,
        idleCursors: <boolean>,
        idleSessions: <boolean>,
        localOps: <boolean>
      }
    }
  ],
  cursor: {}
})
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`allUsers`** | Optional. Boolean. If `true`, reports operations for all users. Default: `false`. |
| **`idleConnections`** | Optional. Boolean. If `true`, reports on idle connections. Default: `false`. |
| **`idleCursors`** | Optional. Boolean. If `true`, reports on idle cursors. Default: `false`. |
| **`idleSessions`** | Optional. Boolean. If `true`, reports on idle sessions. Default: `false`. |
| **`localOps`** | Optional. Boolean. If `true`, reports operations running locally on the current instance. Default: `false`. |

## Examples

### Example 1: List active operations

Return all currently active operations:

```javascript
db.adminCommand({
  aggregate: 1,
  pipeline: [
    { $currentOp: { allUsers: true } }
  ],
  cursor: {}
})
```

### Example 2: Filter active operations

Return active operations for a specific database, combined with `$match`:

```javascript
db.adminCommand({
  aggregate: 1,
  pipeline: [
    { $currentOp: { allUsers: true } },
    { $match: { "ns": /^mydb\./ } }
  ],
  cursor: {}
})
```

## Key Takeaways

- **Must be first stage** — `$currentOp` must be the first stage in the aggregation pipeline
- **Admin database only** — this stage must be run against the `admin` database using `db.adminCommand()`
- **Collection-agnostic** — does not operate on a specific collection

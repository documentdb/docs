---
title: currentOp
description: The currentOp command returns information about the operations currently running on the server.
type: commands
category: administration
---

# currentOp

The `currentOp` command returns a single document that describes the operations
currently running on the DocumentDB server, including queries, updates, index
builds, and administrative commands. It is the primary tool for diagnosing slow
or stuck activity on the DocumentDB cluster, and the source of the operation identifiers
consumed by [`killOp`](killOp.md).

## Syntax

```javascript
db.getSiblingDB("admin").runCommand({
  currentOp: 1,
  // Optional flags
  $all: <boolean>,
  $ownOps: <boolean>,

  // Any other top-level field is treated as a filter applied
  // to the operations returned in `inprog`.
  <field>: <value>,
  ...
})
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`currentOp`** | Required. Always `1`. |
| **`$all`** | Optional boolean. If `true`, return operations from every user (equivalent to MongoDB's `allUsers`). Default is `false`. |
| **`$ownOps`** | Optional boolean. Reserved for a future release; the field is parsed and validated but currently has no effect on the result set. |
| Any other top-level field | Treated as a query filter on the `inprog` array. For example `{ currentOp: 1, op: "query" }` only returns running operations whose `op` is `"query"`. |

## Return value

```json
{
  "inprog": [
    {
      "type": "op",
      "host": "<host:port>",
      "shard": "<shardName>",
      "opid": "<shardId>:<opId>",
      "active": true,
      "secs_running": <int>,
      "microsecs_running": <int>,
      "op": "query|insert|update|remove|command|...",
      "ns": "<db>.<collection>",
      "command": { ... },
      "client": "<host:port>",
      "appName": "<applicationName>",
      "currentOpTime": "<ISO8601 timestamp>"
    }
  ],
  "ok": 1
}
```

## Examples

### Example 1: List active operations across the DocumentDB cluster

```javascript
db.getSiblingDB("admin").runCommand({ currentOp: 1 })
```

### Example 2: List long-running queries from all users

```javascript
db.getSiblingDB("admin").runCommand({
  currentOp: 1,
  $all: true
}).inprog.filter(op => op.secs_running > 30 && op.op === "query")
```

### Example 3: Pair with killOp to cancel a stuck operation

```javascript
const ops = db.getSiblingDB("admin").runCommand({
  currentOp: 1,
  $all: true
}).inprog;

const target = ops.find(o => o.ns === "shop.orders" && o.secs_running > 60);

if (target) {
  db.getSiblingDB("admin").runCommand({
    killOp: 1,
    op: target.opid
  });
}
```

## Related commands

- [killOp](killOp.md) — terminate a single operation by `opid`.
- [killSessions](killSessions.md) — terminate every operation that belongs to one or more sessions.

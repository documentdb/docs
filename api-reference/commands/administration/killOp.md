---
title: killOp
description: The killOp command terminates an in-flight operation identified by its operation id.
type: commands
category: administration
---

# killOp

> **Added in v0.109-0.**

The `killOp` command cancels a single operation that is currently running on the
server. The operation is identified by an opaque operation identifier that you
can obtain from [`currentOp`](currentOp.md).

## Syntax

```javascript
db.getSiblingDB("admin").runCommand({
  killOp: 1,
  op: "<shardId>:<opId>"
})
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`killOp`** | Required. Always set to `1`. |
| **`op`** | Required. String of the form `"<shardId>:<opId>"`. Both parts must contain only digits. The values are taken from the `opid` field returned by `currentOp`. |

## Behavior

- Must be issued against the `admin` database.
- Requires the calling user to have permission to cancel backend processes
  (`pg_signal_backend` membership or superuser).
- The command attempts to cancel the operation identified by `op`. If the
  operation has already finished, the command is a no-op.
- The server returns the shard that handled the cancellation request.

## Return value

```json
{
  "shard": "shardN",
  "shardid": <int>,
  "ok": 1
}
```

## Examples

### Example 1: Find a long-running query and terminate it

```javascript
// 1. List currently running operations
const ops = db.getSiblingDB("admin").runCommand({
  currentOp: 1,
  $all: true
}).inprog;

// 2. Pick the operation you want to cancel
const target = ops.find(o => o.secs_running > 60 && o.op === "query");

// 3. Terminate it
db.getSiblingDB("admin").runCommand({
  killOp: 1,
  op: target.opid
})
```

### Example 2: Terminate by explicit identifier

```javascript
db.getSiblingDB("admin").runCommand({
  killOp: 1,
  op: "10000004122:12345"
})
```

## Related commands

- [currentOp](currentOp.md) — list running operations and obtain `opid` values.
- [killSessions](killSessions.md) — terminate every operation that belongs to one or more sessions.

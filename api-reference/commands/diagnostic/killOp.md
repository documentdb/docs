---
title: killOp
description: Cancel a DocumentDB operation using the string identifier reported by currentOp.
type: commands
category: diagnostic
---

# killOp

Requests cancellation of a running operation. Run against `admin`.

```javascript
const flightOps = db.currentOp({
  active: true,
  ns: "bookings.flights"
}).inprog
if (flightOps.length > 0) {
  db.killOp(flightOps[0].opid)
}
```

| Parameter | Description |
| --- | --- |
| **`killOp`** | (Required) A command marker, conventionally `1`. |
| **`op`** | (Required) A string specifying the operation identifier reported by [`currentOp`](currentOp.md). Numeric identifiers are rejected by the gateway. |

Cancellation is subject to server authorization. See [wire protocol and common fields](wire-protocol.md).

Source: gateway `processor/data_management.rs`, `process_kill_op`, and `pg_documentdb/src/commands/kill_op.c`.

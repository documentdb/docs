---
title: killSessions
description: The killSessions command terminates one or more client sessions, killing their cursors and rolling back any in-flight transactions.
type: commands
category: administration
---

# killSessions

> **Added in v0.110-0.**

The `killSessions` command terminates the specified client sessions. For each
session it removes any open cursors and rolls back any in-flight transaction
state on the server. It is the explicit, server-driven counterpart to the
client-driven `endSessions` command.

## Syntax

```javascript
db.adminCommand({
  killSessions: [
    { id: <UUID> },
    ...
  ]
})
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`killSessions`** | Array of session descriptors to terminate. Each entry is a document with a single binary `id` field that holds the session UUID returned by `startSession`. |

## Behavior

- For every session id, all cursors that belong to the session are invalidated
  and any matching server-side `killCursors` work is performed.
- Any in-progress transaction owned by the session is removed from the
  transaction store on a best-effort basis.
- The command always returns success even if some session ids were unknown.

## Return value

```json
{ "ok": 1 }
```

## Examples

### Example 1: Terminate a single session

```javascript
const sess = db.getMongo().startSession();
const sessionId = sess.getSessionId();

db.adminCommand({
  killSessions: [ { id: sessionId.id } ]
})
```

### Example 2: Terminate multiple sessions at once

```javascript
db.adminCommand({
  killSessions: [
    { id: UUID("9b6a3b2c-3d49-4f7c-9f42-2a5fdc8a1d11") },
    { id: UUID("e21c1c8a-1f0e-49e7-9f02-b7ad9b3eafff") }
  ]
})
```

## Related commands

- [killOp](killOp.md) — terminate a single in-flight operation by operation id.
- [currentOp](currentOp.md) — list operations currently running on the server.

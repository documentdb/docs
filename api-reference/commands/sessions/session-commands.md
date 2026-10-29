---
title: Session commands
description: Terminate DocumentDB logical sessions.
type: commands
category: sessions
---

# Session commands

Drivers create logical session IDs and attach them as `lsid`. The default gateway does not implement the explicit `startSession` wire command; this does not prevent driver-managed sessions, including the shell's `startSession()` helper.

## endSessions and killSessions

The examples attach sessions to reads of the [airline dataset](../diagnostic/wire-protocol.md#airline-example-dataset):

```javascript
const bookingsSession = db.getMongo().startSession()
bookingsSession.getDatabase("bookings").flights.find().limit(1).toArray()
bookingsSession.endSession()

const dispatcherSession = db.getMongo().startSession()
dispatcherSession.getDatabase("bookings").flights.find().limit(1).toArray()
db.adminCommand({ killSessions: [dispatcherSession.id] })
dispatcherSession.endSession()
```

`endSession()` ends the client session; the driver manages server-session cleanup and can pool the underlying server session. It does not guarantee immediate dispatch of `endSessions`. There is no equivalent shell helper for explicitly sending `killSessions`, so that example uses an administrative request.

| Parameter | Description |
| --- | --- |
| **`endSessions`** | (Required) An array of documents specifying the session-ID list. Each document is `{ id: <BSON UUID> }`. |
| **`killSessions`** | (Required) An array of documents specifying the session-ID list. Each document is `{ id: <BSON UUID> }`. |

Both handlers invalidate the selected sessions' cursors and remove their transactions, subject to ownership authorization. With session argument validation enabled, extra fields inside an ID document are rejected, UUID binary subtype is required, and `null` array elements are ignored. An empty array selects no sessions.

See [transaction commands](transaction-commands.md) and [wire protocol and common fields](../diagnostic/wire-protocol.md).

Source: gateway `processor/session.rs`, `processor/process.rs`, and `requests/request_type.rs`.

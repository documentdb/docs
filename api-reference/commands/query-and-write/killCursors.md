---
title: killCursors
description: Close DocumentDB cursors by their BSON 64-bit identifiers.
type: commands
category: query-and-write
---

# killCursors

Closes the specified cursors and releases their resources.

```javascript
const flightCursor = db.flights.find().batchSize(1)
flightCursor.next()
flightCursor.close()
```

The [airline dataset](../diagnostic/wire-protocol.md#airline-example-dataset) contains more than one flight. `next()` executes the query and consumes one result; `close()` closes the remaining server cursor. The shell manages the cursor ID and session metadata.

| Parameter | Description |
| --- | --- |
| **`killCursors`** | (Required) A string specifying the collection name. |
| **`cursors`** | (Required) An array of BSON int64 values specifying the cursor IDs to close. Use IDs returned by the server, not JavaScript numbers. |

The response categorizes IDs in `cursorsKilled`, `cursorsNotFound`, `cursorsAlive`, and `cursorsUnknown`. See [wire protocol and common fields](../diagnostic/wire-protocol.md).

Source: gateway `processor/cursor.rs`, `process_kill_cursors`.

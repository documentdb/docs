---
title: Transaction commands
description: Commit or abort transactions using DocumentDB session and transaction metadata.
type: commands
category: sessions
---

# Transaction commands

Drivers start a transaction by adding transaction metadata to its first operation, not by sending a separate begin command.

## Transaction context

Use the driver's transaction API to maintain matching [session and transaction metadata](../diagnostic/wire-protocol.md#common-request-fields). Read isolation follows the [shared read-concern rules](../diagnostic/wire-protocol.md#read-concern).

## commitTransaction and abortTransaction

Using the [airline example dataset](../diagnostic/wire-protocol.md#airline-example-dataset), start a transaction with a read and commit it. The session helper maintains matching `lsid` and `txnNumber` values:

```javascript
const bookingsSession = db.getMongo().startSession({ causalConsistency: false })
const bookingsDb = bookingsSession.getDatabase("bookings")
bookingsSession.startTransaction()
bookingsDb.flights.findOne({ route_no: "PG0101" })
bookingsSession.commitTransaction()
bookingsSession.endSession()
```

Use `bookingsSession.abortTransaction()` instead of `commitTransaction()` to abort. The helpers do not return a raw command document; the wire handlers return `ok: 1` on success. A missing or nonexistent active transaction produces `NoSuchTransaction`; committing an already committed transaction is handled as an idempotent retry.

The handlers do not implement per-command `writeConcern` policies, `maxCommitTimeMS`, or recovery-token coordination.

Source: gateway `processor/transaction.rs` and `requests/common_fields.rs`.

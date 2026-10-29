---
title: getMore
description: The getMore command is used to retrieve extra batches of documents from an existing cursor.
type: commands
category: query-and-write
---

# getMore

The `getMore` command is used to retrieve extra batches of documents from an existing cursor. This command is useful when dealing with large datasets that can't be fetched in a single query due to size limitations. The command allows clients to paginate through the results in manageable chunks with commands that return a cursor. For example, [find](https://documentdb.io/docs/reference/commands/query-and-write/find/) and [aggregate](https://documentdb.io/docs/reference/commands/aggregation/aggregate/), to return subsequent batches of documents currently pointed to by the cursor.

Returns the next batch from an existing cursor in `cursor.nextBatch`. Use the cursor identifier returned by a preceding command; its BSON int64 type must be preserved.

## Parameters

| Parameter | Description |
| --- | --- |
| **`getMore`** | (Required) A BSON int64 specifying the cursor ID. A BSON int32 is rejected. |
| **`collection`** | (Conditional) A string specifying the collection associated with the cursor. Required when gateway argument validation is enabled. |
| **`batchSize`** | (Optional) A number or null specifying the requested batch limit. With argument validation enabled, values that coerce to a negative integer are rejected. |
| **`maxTimeMS`** | (Optional) A number specifying the timeout for the batch; for a tailable cursor, bounds the wait for new data. |
| **`comment`** | (Optional) A string carrying trace context for gateway request tracing; see [common request fields](../diagnostic/wire-protocol.md). |

Shell and driver cursors automatically maintain the [session and transaction metadata](../diagnostic/wire-protocol.md#common-request-fields) needed to continue a cursor. Requests must use the session and transaction that created the cursor. A cursor created without a session must remain sessionless.

An omitted `batchSize` does not use the initial-query default of `101`: paging is bounded by the remaining results and the response size limit, up to 16 MB. A short batch alone does not prove that the cursor is exhausted. Stop when the returned `cursor.id` is zero.

With argument validation enabled, unknown non-`$` fields are rejected. Legacy configurations validate only the cursor ID at this stage.

## Example: page through flights

Using the [airline example dataset](../diagnostic/wire-protocol.md#airline-example-dataset), let the shell continue the cursor:

```javascript
const flightCursor = db.flights.find({}, { _id: 0, route_no: 1 })
  .sort({ route_no: 1 }).batchSize(2)
flightCursor.next()
flightCursor.next()
flightCursor.next()
flightCursor.next()
flightCursor.close()
```

Successive `next()` calls return documents for routes `PG0101`, `PG0102`, `PG0103`, and `PG0104`. The shell requests another batch when the first two results are consumed, managing the cursor ID and session metadata automatically. The option descriptions above cover that underlying `getMore` request.

## Shell syntax

```javascript
db.runCommand({
   getMore: NumberLong("<cursor-id>"),
   collection: <collection-name>,
   batchSize: <number-of-documents>,
   maxTimeMS: <milliseconds>
})
```

## Detailed Examples

### Example 1: Retrieve more documents from a cursor

Assume you have a cursor with the ID `1234567890` from the `stores` collection. The following command retrieves up to five more documents:

```javascript
db.runCommand({
   getMore: NumberLong("1234567890"),
   collection: "stores",
   batchSize: 5
})
```

### Example 2: Drain the rest of the cursor

Omitting `batchSize` returns every document still held by the cursor in a single batch, up to the 16 MB limit:

```javascript
db.runCommand({
   getMore: NumberLong("1234567890"),
   collection: "stores"
})
```

A batch can come back smaller than requested, and an omitted `batchSize` does not guarantee the cursor was drained — the 16 MB batch limit can cut it short. Always keep calling `getMore` until the response reports a `cursor.id` of `0`, rather than stopping when a batch is shorter than `batchSize`.

## Implementation

Source: gateway `processor/cursor.rs`, `validate_get_more_arguments` and `process_get_more`. See [wire protocol and common fields](../diagnostic/wire-protocol.md) for the audited source revision and metadata behavior.

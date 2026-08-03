---
title: getMore
description: The getMore command is used to retrieve extra batches of documents from an existing cursor.
type: commands
category: query-and-write
---

# getMore

The `getMore` command is used to retrieve extra batches of documents from an existing cursor. This command is useful when dealing with large datasets that can't be fetched in a single query due to size limitations. The command allows clients to paginate through the results in manageable chunks with commands that return a cursor. For example, [find](https://documentdb.io/docs/reference/commands/query-and-write/find/) and [aggregate](https://documentdb.io/docs/reference/commands/aggregation/aggregate/), to return subsequent batches of documents currently pointed to by the cursor.

## Syntax

The syntax for the `getMore` command is as follows:

```javascript
db.runCommand({
   getMore: NumberLong("<cursor-id>"),
   collection: <collection-name>,
   batchSize: <number-of-documents>,
   maxTimeMS: <milliseconds>
})
```

- `getMore`: The unique identifier for the cursor from which to retrieve more documents, taken from the `cursor.id` field of the originating `find` or `aggregate` response. This field must be a BSON 64-bit integer — in `mongosh` write it as `NumberLong("...")`, and in Extended JSON as `{"$numberLong": "..."}`. A plain JavaScript number is serialized as a 32-bit integer and is rejected with `BadValue: getMore value should be an i64`.
- `collection`: The name of the collection associated with the cursor.
- `batchSize`: (Optional) The maximum number of documents to return in the batch. Unlike the first page of `find` or `aggregate`, which defaults to 101 documents, `getMore` has no small default — if `batchSize` is omitted the server returns everything remaining in the cursor, stopping only when the accumulated batch reaches 16 MB.
- `maxTimeMS`: (Optional) A statement timeout for this batch. On a tailable cursor such as a change stream it instead bounds how long the server waits for new data.

## Examples

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

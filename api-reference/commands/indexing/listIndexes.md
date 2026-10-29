---
title: listIndexes
description: List index specifications using a DocumentDB command cursor.
type: commands
category: indexing
---

# listIndexes

Returns index specifications for a collection in a cursor.

```javascript
db.flights.getIndexes()
```

The helper returns an array of index specifications and handles the command cursor.

| Parameter | Description |
| --- | --- |
| **`listIndexes`** | (Required) A string specifying the collection name. |
| **`cursor`** | (Optional) A document that supports numeric `batchSize`, converted to a non-negative integer, for the first batch; also supports boolean `singleBatch` to close the cursor after that batch. |
| **`maxTimeMS`** | (Optional) A number specifying the execution timeout in milliseconds. |
| **`cursor.batchSize`** | (Optional) A number specifying the first-batch limit, converted to a non-negative integer. |
| **`cursor.singleBatch`** | (Optional) A boolean that closes the cursor after the first batch when `true`. |

The default wire command does not offer `includeBuildUUIDs` or `includeIndexBuildInfo`. See [wire protocol and common fields](../diagnostic/wire-protocol.md) for shared metadata.

The backend parser also accepts the spelling `listindexes`, but the gateway name parser does not expose it as a wire-command alias.

Source: `pg_documentdb/src/aggregation/bson_aggregation_metadata_queries.c`, `GenerateListIndexesQuery`.

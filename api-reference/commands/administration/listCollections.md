---
title: listCollections
description: List collection and view metadata in DocumentDB with filtering and cursor options.
type: commands
category: administration
---

# listCollections

Returns collection and view metadata in a cursor.

```javascript
db.getCollectionInfos({ name: "flights" }, { batchSize: 20 })
```

The helper returns an array of metadata documents and handles the command cursor.

| Parameter | Description |
| --- | --- |
| **`listCollections`** | (Required) A command marker, conventionally `1`. |
| **`filter`** | (Optional) A document that filters the returned metadata documents; `null` means no filter. |
| **`cursor`** | (Optional) A document that supports numeric `batchSize`, converted to a non-negative integer, for the first batch; also supports boolean `singleBatch` to close the cursor after that batch. |
| **`nameOnly`** | (Optional) A boolean-like value that returns only collection names and types rather than full metadata; defaults to `false`. |
| **`addDistributedMetadata`** | (Optional) A boolean-like value provided as a DocumentDB extension including distributed collection metadata. |
| **`maxTimeMS`** | (Optional) A number specifying the execution timeout in milliseconds. |
| **`cursor.batchSize`** | (Optional) A number specifying the first-batch limit, converted to a non-negative integer. |
| **`cursor.singleBatch`** | (Optional) A boolean that closes the cursor after the first batch when `true`. |

See [wire protocol and common fields](../diagnostic/wire-protocol.md) for shared metadata.

Source: `pg_documentdb/src/aggregation/bson_aggregation_metadata_queries.c`, `GenerateListCollectionsQuery`.

---
title: dropIndexes
description: Remove DocumentDB indexes selected by name, key pattern, or a list of names.
type: commands
category: indexing
---

# dropIndexes

Removes indexes from a collection.

```javascript
db.flights.dropIndex("route_no_1")
```

First create `route_no_1` using the airline example in [`createIndexes`](createIndexes.md).

| Parameter | Description |
| --- | --- |
| **`dropIndexes`** | (Required) A string specifying the collection name. |
| **`index`** | (Required) A string, document, or array of strings specifying the index name, key pattern, or list of names. `"*"` drops eligible non-`_id` indexes. |
| **`comment`** | (Optional) A string carrying trace context for gateway request tracing; see [common request fields](../diagnostic/wire-protocol.md). |

The `_id` index cannot be dropped.

See [wire protocol and common fields](../diagnostic/wire-protocol.md).

## Implementation

The shell's `dropIndex()` helper sends `dropIndexes`.

Source: `pg_documentdb/src/commands/drop_indexes.c`, `ParseDropIndexesArg`, and gateway `processor/process.rs`.

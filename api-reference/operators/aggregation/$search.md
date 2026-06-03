---
title: $search
description: The $search stage performs full-text search on specified fields using text indexes.
type: operators
category: aggregation
---

# $search

The `$search` stage performs full-text search against fields covered by a text index. This stage must be the first stage in the aggregation pipeline and leverages text indexes for efficient search operations.

## Syntax

```javascript
{
  $search: {
    <operator>: {
      <specification>
    }
  }
}
```

## Parameters

The `$search` stage accepts search operator specifications. The exact parameters depend on the search configuration and the text index defined on the collection.

| Parameter | Description |
| --- | --- |
| **`operator`** | The search operator to use for the query. |
| **`specification`** | The operator-specific search criteria. |

## Examples

### Example 1: Basic text search

Perform a text search for stores matching a keyword:

```javascript
db.stores.aggregate([
  {
    $search: {
      text: {
        query: "Beverage",
        path: "name"
      }
    }
  },
  { $project: { name: 1, score: { $meta: "searchScore" } } },
  { $limit: 5 }
])
```

## Key Takeaways

- **Must be first stage** — `$search` must be the first stage in the aggregation pipeline
- **Requires a text index** — the collection must have an appropriate text index configured
- **Search scores** — use `{ $meta: "searchScore" }` in a `$project` stage to include relevance scores

---
title: $search
description: The $search stage in the aggregation pipeline runs a vector similarity search through the cosmosSearch or knnBeta operator.
type: operators
category: aggregation
---

# $search

The `$search` stage in the aggregation pipeline runs a vector similarity search over a vector index. It carries exactly one search operator — `cosmosSearch`, or the deprecated `knnBeta` — and returns the documents whose stored embedding is closest to a query vector.

`$search` must be the first stage in the pipeline, and the field named by `path` must be covered by a vector index.

> `$search` in DocumentDB is a vector search stage. It does not accept text-search operators: a spec such as `{ $search: { text: { ... } } }` is rejected with `Unrecognized $search option: text`. For full-text queries, use the `$text` query operator against a text index. For new work, prefer [`$vectorSearch`](./%24vectorsearch.md), which is the current stage for vector search and takes a clearer spec; `$search` remains for compatibility with existing `cosmosSearch` and `knnBeta` queries.

## Syntax

```javascript
{
  $search: {
    index: <string>,
    cosmosSearch: {
      path: <string>,
      vector: [<number>, ...],
      k: <positiveInteger>,
      filter: <document>,
      exact: <boolean>,
      oversampling: <number>
    }
  }
}
```

## Parameters

The stage takes exactly one search operator, plus a small number of options alongside it.

| Parameter | Description |
| --- | --- |
| **`cosmosSearch`** | The search operator, holding the query spec described below. Exactly one operator must be present — supplying both `cosmosSearch` and `knnBeta` fails with `The $search spec can only contain one search operator.`, and supplying neither fails with `Invalid search spec provided, must include one of the supported operators.` |
| **`knnBeta`** | Deprecated alias kept for backward compatibility. It takes the same spec as `cosmosSearch`, except that `filter` and `score` are rejected outright. Use `cosmosSearch` instead. |
| **`index`** | Optional. The name of the index to search, as a string. |
| **`returnStoredSource`** | Optional. Accepted for compatibility and ignored. |
| **`count`** | Optional. Parsed and validated, but the resulting count metadata is **not yet emitted in the results**, so the option currently has no observable effect. |

Any other key alongside the operator is rejected with `Unrecognized $search option: <name>`.

### Operator spec

| Field | Description |
| --- | --- |
| **`path`** | Required. The document field holding the stored embedding. Must be covered by a vector index. Omitting it fails with `$path is required field for using a vector index.` |
| **`vector`** | Required. The query embedding, as a non-empty array of numbers. Omitting it fails with `$vector is required field for using a vector index.` |
| **`k`** | Required. The number of documents to return, as a positive integer. |
| **`filter`** | Optional. A query document intersected with the vector search. Requires vector pre-filtering to be enabled on the server; when it is not, the query fails with `$filter is not supported for vector search yet.` Not supported at all with `knnBeta`. |
| **`exact`** | Optional. Boolean. Runs an exact search instead of an approximate one. |
| **`oversampling`** | Optional. Widens the candidate set considered during an approximate search, improving recall at the cost of latency. |
| **`score`** | Optional. Not supported with `knnBeta`, which rejects it with `$score is not supported for knnBeta queries.` |

## Examples

The examples on this page use the following documents in a `products` collection, with a vector index on `embedding`.

```json
[
  { "_id": 1, "name": "Espresso Machine", "category": "appliance", "embedding": [0.9, 0.1, 0.05] },
  { "_id": 2, "name": "Coffee Grinder", "category": "appliance", "embedding": [0.85, 0.15, 0.1] },
  { "_id": 3, "name": "Merlot Bottle", "category": "wine", "embedding": [0.05, 0.9, 0.2] },
  { "_id": 4, "name": "Chardonnay Bottle", "category": "wine", "embedding": [0.1, 0.85, 0.25] }
]
```

### Example 1: Nearest neighbors for a query vector

Return the three documents closest to a query embedding:

```javascript
db.products.aggregate([
  {
    $search: {
      cosmosSearch: {
        path: "embedding",
        vector: [0.88, 0.12, 0.07],
        k: 3
      }
    }
  },
  { $project: { name: 1, category: 1 } }
])
```

```json
[
  { "_id": 1, "name": "Espresso Machine", "category": "appliance" },
  { "_id": 2, "name": "Coffee Grinder", "category": "appliance" },
  { "_id": 4, "name": "Chardonnay Bottle", "category": "wine" }
]
```

### Example 2: Exact search

Set `exact` to `true` to compare against every indexed vector rather than an approximate candidate set. This is slower, and useful when you need a reference result to measure recall against:

```javascript
db.products.aggregate([
  {
    $search: {
      cosmosSearch: {
        path: "embedding",
        vector: [0.88, 0.12, 0.07],
        k: 3,
        exact: true
      }
    }
  },
  { $project: { name: 1 } }
])
```

## Behavior

- **`$search` must come first.** Placing it anywhere else fails with `$search must appear as the initial stage in the pipeline sequence.` The same applies when the pipeline already carries a limit ahead of it.
- **One operator per spec.** The stage rejects a spec carrying more than one search operator, and a spec carrying none.
- **`path` must be indexed.** The field named by `path` must be covered by a vector index; see [`$vectorSearch`](./%24vectorsearch.md) for creating one.

## Related content

- [`$vectorSearch`](./%24vectorsearch.md) — the current vector search stage, and the one to prefer for new queries.
- [`$project`](./%24project.md) — shape the documents returned by the search.
- [`$limit`](./%24limit.md) — narrow the result set further; `k` already bounds it.

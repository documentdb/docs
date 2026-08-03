---
title: $vectorSearch
description: The $vectorSearch stage in the aggregation pipeline runs an approximate nearest-neighbor search over a vector index.
type: operators
category: aggregation
---

# $vectorSearch

The `$vectorSearch` stage in the aggregation pipeline runs an approximate nearest-neighbor search over a vector index and returns the documents whose stored embedding is closest to a query vector. It lets you keep documents and their embeddings in one database — semantic search, retrieval-augmented generation, and recommendations run as an ordinary aggregation pipeline, with the results available to every stage that follows.

`$vectorSearch` must be the first stage in the pipeline, and the field named by `path` must be covered by a vector index.

## Syntax

```javascript
{
  $vectorSearch: {
    queryVector: [<number>, ...],
    path: <string>,
    limit: <positiveInteger>,
    numCandidates: <integer>,
    filter: <document>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`queryVector`** | Required. The query embedding, as an array of numbers. Cannot be empty, and cannot be longer than 16000 elements. Its length must match the `dimensions` of the index. |
| **`path`** | Required. The document field holding the stored embedding. Must be covered by a vector index. |
| **`limit`** | Required. The number of documents to return. Must be a positive 32-bit integer. |
| **`numCandidates`** | Optional. For HNSW indexes, the size of the candidate list explored during the search (`efSearch`). An integer between 1 and 1000; larger values improve recall at the cost of latency. When omitted, DocumentDB chooses a default from the index's `efConstruction` and the size of the collection. |
| **`filter`** | Optional. A query document applied as a pre-filter, so only matching documents are considered. Every field used in the filter must itself be indexed. |

Any other field is rejected with `BSON field '$vectorSearch.<name>' is an unknown field`.

## Examples

The examples on this page use the following documents in a `products` collection.

```json
[
  { "_id": 1, "name": "Espresso Machine", "category": "appliance", "embedding": [0.9, 0.1, 0.05] },
  { "_id": 2, "name": "Coffee Grinder", "category": "appliance", "embedding": [0.85, 0.15, 0.1] },
  { "_id": 3, "name": "Merlot Bottle", "category": "wine", "embedding": [0.05, 0.9, 0.2] },
  { "_id": 4, "name": "Chardonnay Bottle", "category": "wine", "embedding": [0.1, 0.85, 0.25] },
  { "_id": 5, "name": "Whiskey Tumbler", "category": "glassware", "embedding": [0.2, 0.3, 0.9] }
]
```

Three-element vectors keep the examples readable; real embeddings are typically hundreds or thousands of elements.

### Creating the vector index

`$vectorSearch` requires an index on the embedding field. Create one with `createIndexes` using the `cosmosSearch` index type:

```javascript
db.runCommand({
  createIndexes: "products",
  indexes: [{
    key: { embedding: "cosmosSearch" },
    name: "embedding_hnsw_idx",
    cosmosSearchOptions: {
      kind: "vector-hnsw",
      similarity: "COS",
      dimensions: 3,
      m: 16,
      efConstruction: 64
    }
  }]
})
```

`kind` accepts `vector-hnsw` and `vector-ivf`. `similarity` selects the distance metric — `COS` for cosine, `L2` for Euclidean, `IP` for inner product. `dimensions` must match the length of the vectors you store and query.

### Example 1: Finding the nearest documents

This query returns the three products closest to the query vector.

```javascript
db.products.aggregate([
  {
    $vectorSearch: {
      queryVector: [0.9, 0.1, 0.05],
      path: "embedding",
      limit: 3
    }
  },
  { $project: { _id: 0, name: 1, category: 1 } }
])
```

This query returns the following result:

```json
[
  { "name": "Espresso Machine", "category": "appliance" },
  { "name": "Coffee Grinder", "category": "appliance" },
  { "name": "Whiskey Tumbler", "category": "glassware" }
]
```

Results come back ordered from nearest to furthest.

### Example 2: Returning the similarity score

The score assigned by the search is available to later stages through `$meta: "searchScore"`.

```javascript
db.products.aggregate([
  {
    $vectorSearch: {
      queryVector: [0.9, 0.1, 0.05],
      path: "embedding",
      limit: 2
    }
  },
  { $project: { _id: 0, name: 1, score: { $meta: "searchScore" } } }
])
```

This query returns the following result:

```json
[
  { "name": "Espresso Machine", "score": 1 },
  { "name": "Coffee Grinder", "score": 0.9961580039320722 }
]
```

### Example 3: Restricting the search with a filter

`filter` narrows the candidate set before the vector comparison, so `limit` counts only documents that satisfy it.

**Every field used in `filter` must be indexed.** Index the filter path first:

```javascript
db.products.createIndex({ category: 1 }, { name: "category_idx" })
```

Then filter on it:

```javascript
db.products.aggregate([
  {
    $vectorSearch: {
      queryVector: [0.9, 0.1, 0.05],
      path: "embedding",
      limit: 2,
      filter: { category: "wine" }
    }
  },
  { $project: { _id: 0, name: 1, category: 1 } }
])
```

This query returns the following result:

```json
[
  { "name": "Chardonnay Bottle", "category": "wine" },
  { "name": "Merlot Bottle", "category": "wine" }
]
```

Without that index the same query fails:

```
The index for filter path 'category' was not found, please check whether the index is created.
```

### Example 4: Tuning recall with numCandidates

```javascript
db.products.aggregate([
  {
    $vectorSearch: {
      queryVector: [0.1, 0.88, 0.2],
      path: "embedding",
      limit: 2,
      numCandidates: 40
    }
  },
  { $project: { _id: 0, name: 1, category: 1 } }
])
```

This query returns the following result:

```json
[
  { "name": "Merlot Bottle", "category": "wine" },
  { "name": "Chardonnay Bottle", "category": "wine" }
]
```

Raise `numCandidates` when a search misses documents you expect it to find; the search explores a larger neighborhood before selecting the top `limit` results.

## Error cases

| Problem | Error |
| --- | --- |
| `path`, `queryVector`, or `limit` missing | `$path, $queryVector, and $limit are all required fields for using a vector index.` |
| `limit` is zero or negative | `$vectorSearch.limit must be provided as a positive integer value.` |
| `queryVector` is empty | `$vectorSearch.queryVector cannot be an empty array.` |
| `queryVector` is not an array of numbers | `$vectorSearch.queryVector must be an array of numbers.` |
| `numCandidates` above 1000 | `$vectorSearch.numCandidates must be less than or equal to 1000.` |
| No vector index on `path` | `Similarity index was not found for a vector similarity search query.` |
| Unindexed field in `filter` | `The index for filter path '<field>' was not found, please check whether the index is created.` |
| Unknown field | `BSON field '$vectorSearch.<name>' is an unknown field` |

## Notes

An `index` field is accepted for compatibility but currently ignored — the search uses the vector index defined on `path`.

## Related

- [`$limit`](../%24limit/) — `$vectorSearch` takes its own `limit`; a later `$limit` can narrow the result further.
- [`$project`](../%24project/) — shape the results, and surface the similarity score with `$meta`.

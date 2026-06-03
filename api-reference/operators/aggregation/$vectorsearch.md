---
title: $vectorSearch
description: The $vectorSearch stage performs approximate nearest neighbor (ANN) search using vector embeddings.
type: operators
category: aggregation
---

# $vectorSearch

The `$vectorSearch` stage performs an approximate nearest neighbor (ANN) search on vector embeddings stored in documents. This is used for similarity search applications such as semantic search, recommendation systems, and image search.

## Syntax

```javascript
{
  $vectorSearch: {
    queryVector: <array of numbers>,
    path: <string>,
    numCandidates: <number>,
    limit: <number>,
    index: <string>,
    filter: <document>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`queryVector`** | Required. An array of numbers representing the query vector to search with. |
| **`path`** | Required. The field path containing the stored vector embeddings to search against. |
| **`numCandidates`** | Optional. The number of nearest neighbor candidates to consider. Higher values improve accuracy but reduce performance. |
| **`limit`** | Required. The maximum number of results to return. |
| **`index`** | Required. The name of the vector index to use for the search. |
| **`filter`** | Optional. A pre-filter expression to narrow down the search space before performing vector search. |

## Examples

Consider a `products` collection with vector embeddings.

```json
{
  "_id": "prod-001",
  "name": "Wireless Headphones",
  "category": "Electronics",
  "embedding": [0.12, -0.34, 0.56, 0.78, -0.91]
}
```

### Example 1: Basic vector search

Find the 5 most similar products to a query vector:

```javascript
db.products.aggregate([
  {
    $vectorSearch: {
      queryVector: [0.15, -0.30, 0.50, 0.80, -0.85],
      path: "embedding",
      numCandidates: 100,
      limit: 5,
      index: "vector_index"
    }
  },
  {
    $project: {
      name: 1,
      category: 1,
      score: { $meta: "vectorSearchScore" }
    }
  }
])
```

### Example 2: Vector search with pre-filter

Search only within a specific category:

```javascript
db.products.aggregate([
  {
    $vectorSearch: {
      queryVector: [0.15, -0.30, 0.50, 0.80, -0.85],
      path: "embedding",
      numCandidates: 100,
      limit: 5,
      index: "vector_index",
      filter: { category: "Electronics" }
    }
  },
  {
    $project: {
      name: 1,
      score: { $meta: "vectorSearchScore" }
    }
  }
])
```

## Key Takeaways

- **Must be first stage** — `$vectorSearch` must be the first stage in the aggregation pipeline
- **Requires a vector index** — a vector index must be created on the field specified by `path`
- **Similarity scores** — use `{ $meta: "vectorSearchScore" }` to include similarity scores in the output
- **Pre-filtering** — use the `filter` parameter to restrict the search to a subset of documents before performing vector similarity comparison

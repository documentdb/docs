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
| **`queryVector`** | Required. The query embedding, as an array of numbers. Cannot be empty, and cannot be longer than 16000 elements. Its length must match the `dimensions` of the index — which is itself capped well below 16000 unless the index sets `compression`. See [Creating the vector index](#creating-the-vector-index). |
| **`path`** | Required. The document field holding the stored embedding. Must be covered by a vector index. |
| **`limit`** | Required. The number of documents to return. Must be a BSON 32-bit integer greater than zero. |
| **`numCandidates`** | Optional. For HNSW indexes, the size of the candidate list explored during the search (`efSearch`). Must be a BSON 32-bit integer between 1 and 1000; larger values improve recall at the cost of latency. **Ignored on `vector-ivf` indexes** — no error is raised and the query behaves as if it were omitted. When omitted, DocumentDB derives a default; see [Choosing numCandidates](#choosing-numcandidates). |
| **`filter`** | Optional. A query document intersected with the vector search, so only matching documents are returned. Every field used in the filter must itself be indexed, and `filter` is **not supported on sharded collections**. See [Example 3](#example-3-restricting-the-search-with-a-filter). |

Any other field is rejected with `BSON field '$vectorSearch.<name>' is an unknown field`.

`limit` and `numCandidates` are checked for the BSON int32 wire type with no numeric coercion, so a double or a 64-bit integer is rejected even when its value is in range:

```
The BSON field 'limit' has an incorrect type 'long'; it should be of type 'int'.
```

This matters for drivers that marshal bare integer literals as doubles, and for explicit `NumberLong(...)` values. Note that the index options below (`dimensions`, `m`, `efConstruction`, `numLists`) *do* accept any numeric type — the same literal can be valid in one block and fatal in the other.

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

`kind` and `similarity` are both required. `similarity` selects the distance metric — `COS` for cosine, `L2` for Euclidean, `IP` for inner product. `dimensions` must match the length of the vectors you store and query, and must be greater than 1.

**`dimensions` is capped by `compression`, not by the 16000 query-vector limit.** An uncompressed index tops out at 2000:

| `compression` | Maximum `dimensions` |
| --- | --- |
| omitted (uncompressed) | 2000 |
| `"half"` | 4000 |
| `"pq"` | 16000 |

Exceeding the cap fails index creation with `field cannot have more than 2000 dimensions for vector index`. This is the limit you hit first with common embedding models — `text-embedding-3-large` produces 3072 dimensions, which needs `compression: "half"` or `"pq"`. Both compression modes are gated behind server configuration and `half` additionally requires a pgvector build with half-vector support, so they may be unavailable on a given deployment:

```
Compression type 'half' is not enabled.
The compression type 'half' is currently unsupported.
```

**Tuning options are index-kind specific**, and the two HNSW options are coupled:

| Option | Kind | Range | Default |
| --- | --- | --- | --- |
| `m` | `vector-hnsw` | 2 – 100 | 16 |
| `efConstruction` | `vector-hnsw` | 4 – 1000 | 64 |
| `numLists` | `vector-ivf` | 1 – 32768 | 100 |

`efConstruction` must be at least `2 * m`, otherwise index creation fails:

```
efConstruction must be greater than or equal to 2 * m for vector-hnsw indexes
```

The example above uses `m: 16` with `efConstruction: 64`, which satisfies the rule — raising `m` to 48 without also raising `efConstruction` does not.

An IVF index is created the same way, with `numLists` in place of the HNSW options:

```javascript
db.runCommand({
  createIndexes: "products",
  indexes: [{
    key: { embedding: "cosmosSearch" },
    name: "embedding_ivf_idx",
    cosmosSearchOptions: {
      kind: "vector-ivf",
      similarity: "COS",
      dimensions: 3,
      numLists: 100
    }
  }]
})
```

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

The `$project` here is doing more than trimming output. `$vectorSearch` attaches an internal `__cosmos_meta__` field to every document it returns, so the same pipeline without a projection gives you:

```json
[
  {
    "_id": 1,
    "name": "Espresso Machine",
    "category": "appliance",
    "embedding": [0.9, 0.1, 0.05],
    "__cosmos_meta__": { "score": 1 }
  }
]
```

An *exclusion* projection such as `{ $project: { embedding: 0 } }` does not remove `__cosmos_meta__` — only an inclusion projection that omits it, as above, will. Read the score through `$meta` rather than by reaching into this field, and take care not to write documents straight back into a collection without stripping it.

### Example 2: Returning the similarity score

The score assigned by the search is available to later stages through `$meta: "searchScore"`. `$meta: "vectorSearchScore"` is accepted as an equivalent alias — it is the keyword MongoDB Atlas uses for `$vectorSearch`, so pipelines ported from Atlas work unchanged.

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

`filter` restricts results to documents matching a query document. DocumentDB runs the approximate nearest-neighbor scan and the filter query separately and intersects them, then applies `limit` to the intersection — so every returned document satisfies the filter, but **reaching `limit` is best-effort**: it depends on how many matching documents the ANN scan encounters before it stops.

The practical consequence is that a highly selective filter over a large collection can return fewer documents than `limit`, with no error and nothing to distinguish it from there genuinely being no more matches. On a million-document collection, `filter: { category: "wine" }` matching 0.1% of documents with `limit: 10` may return only a handful of rows. Raising `numCandidates` widens the scan and is the remedy. How far the scan can extend also depends on the deployed pgvector version — iterative scanning requires pgvector 0.8.0 or later and is silently skipped below that, which makes short results considerably more likely.

**`filter` is not supported on sharded collections.** The query is rejected outright, even though unfiltered `$vectorSearch` continues to work on the same collection:

```
Filter is not supported for vector search on sharded collection.
```

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
      numCandidates: 200
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

Raise `numCandidates` when a search misses documents you expect it to find; the search explores a larger neighborhood before selecting the top `limit` results. The five-document collection used here is small enough that the search is exhaustive at any setting, so the value above changes nothing — `numCandidates` only begins to matter once the index is large enough that the HNSW graph is traversed rather than scanned.

## Choosing numCandidates

When `numCandidates` is omitted on an HNSW index, the default depends on the size of the collection, with a hard threshold at 10,000 rows:

| Rows in the collection | Default `efSearch` |
| --- | --- |
| fewer than 10,000 | the index's `efConstruction` |
| 10,000 or more | 40 |

Above the threshold, `efConstruction` stops feeding the default entirely. An index built with `efConstruction: 512` for high recall searches at `efSearch: 40` on a 50,000-document collection unless `numCandidates` is set explicitly — a large drop from the configured value, which surfaces only as missing results. Set `numCandidates` by hand on any production-sized collection.

On `vector-ivf` indexes `numCandidates` has no effect at all. It is accepted and validated, but the IVF search path reads only `nProbes`, which `$vectorSearch` does not expose — so there is no recall knob for IVF in this stage, and raising `numCandidates` returns byte-identical results.

## Error cases

| Problem | Error |
| --- | --- |
| `path`, `queryVector`, or `limit` missing | `$path, $queryVector, and $limit are all required fields for using a vector index.` |
| `limit` is zero or negative | `$vectorSearch.limit must be provided as a positive integer value.` |
| `queryVector` is empty | `$vectorSearch.queryVector cannot be an empty array.` |
| `queryVector` is not an array of numbers | `$vectorSearch.queryVector must be an array of numbers.` |
| `queryVector` longer than 16000 elements | `Length of the query vector cannot exceed 16000` |
| `queryVector` length does not match the index | `expected <n> dimensions, not <m>` |
| `numCandidates` below 1 | `The vectorSearch.numCandidates should have a value that is not less than 1.` |
| `numCandidates` above 1000 | `$vectorSearch.numCandidates must be less than or equal to 1000.` |
| `limit` or `numCandidates` is not a BSON int32 | `The BSON field 'limit' has an incorrect type 'long'; it should be of type 'int'.` |
| `$vectorSearch` is not the first stage | `The $vectorSearch needs to appear as the initial stage in the processing pipeline.` |
| No vector index on `path` | `Similarity index was not found for a vector similarity search query.` |
| Unindexed field in `filter` | `The index for filter path '<field>' was not found, please check whether the index is created.` |
| `filter` on a sharded collection | `Filter is not supported for vector search on sharded collection.` |
| Unknown field | `BSON field '$vectorSearch.<name>' is an unknown field` |

The `numCandidates` lower-bound message omits the `$` prefix that the upper-bound message carries; both are reproduced above as the server emits them. Note also that `numCandidates: 0` is an error rather than a request for the default.

The first-stage check is stricter than it reads: it also rejects the stage when it runs against a view, or nested inside `$facet` or `$lookup`, even where `$vectorSearch` is demonstrably the first stage of that pipeline.

`expected <n> dimensions, not <m>` is raised by pgvector rather than by DocumentDB, so its exact wording can change with a pgvector upgrade.

## Notes

An `index` field is accepted for compatibility but currently ignored — the search uses the vector index defined on `path`.

Documents returned by `$vectorSearch` carry an internal `__cosmos_meta__` field holding the similarity score. It is not removed by an exclusion projection — use an inclusion `$project`, and read the score with `$meta: "searchScore"` rather than from the field directly.

## Related

- [`$limit`](https://documentdb.io/docs/reference/operators/aggregation/%24limit/) — `$vectorSearch` takes its own `limit`; a later `$limit` can narrow the result further.
- [`$project`](https://documentdb.io/docs/reference/operators/aggregation/%24project/) — shape the results, and surface the similarity score with `$meta`.
- [`$meta`](https://documentdb.io/docs/reference/operators/projection/%24meta/) — the metadata operator used to project `searchScore`.

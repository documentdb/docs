---
title: $inverseMatch
description: The $inverseMatch stage in the aggregation pipeline keeps documents whose stored query matches a supplied document, inverting the usual direction of a match.
type: operators
category: aggregation
---

# $inverseMatch

The `$inverseMatch` stage in the aggregation pipeline inverts the usual direction of a match. Where `$match` holds one query and tests it against many documents, `$inverseMatch` reads a query *out of* each document and tests it against one supplied input. A document survives the stage when the query it carries matches that input.

That makes it the stage for collections of stored queries — saved searches, alert rules, subscription filters, routing policies — where the question is not "which documents match this query" but "which stored queries match this document".

`$inverseMatch` is specific to DocumentDB and has no MongoDB equivalent.

## Syntax

The stage takes the input to test in one of two forms. Either supply it inline with `input`:

```javascript
{
  $inverseMatch: {
    path: <string>,
    input: <expression>,
    defaultResult: <boolean>
  }
}
```

Or read it from another collection with `from` and `pipeline`:

```javascript
{
  $inverseMatch: {
    path: <string>,
    from: <string>,
    pipeline: [ <stage>, ... ],
    defaultResult: <boolean>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`path`** | Required. The dotted path at which each document stores its query. Omitting it fails with `Path parameter is missing for the operator $inverseMatch`. |
| **`input`** | The document to test the stored queries against. Must be a constant value or a string path expression such as `"$payload"` — anything else fails with `$inverseMatch expects 'input' to be a constant value or a string path expression.` It has to resolve to a document or an array of documents. Mutually exclusive with `from`. |
| **`from`** | The collection to read input documents from, as a string. Requires `pipeline`. Mutually exclusive with `input`. |
| **`pipeline`** | Required when `from` is set. The pipeline run against `from` to produce the input documents. Only `$match`, `$project` and `$limit` may appear in it. |
| **`defaultResult`** | Optional boolean, default `false`. The result used for a document that has no value at `path` — `false` drops it, `true` keeps it. |

Exactly one of `input` or `from` must be present. Any other key is rejected with `Unrecognized parameter supplied to $inverseMatch: '<name>'`.

## Examples

The examples use an `alerts` collection, where each document stores the query that defines the alert:

```json
[
  { "_id": 1, "name": "High value orders", "criteria": { "total": { "$gt": 500 } } },
  { "_id": 2, "name": "Electronics", "criteria": { "category": "electronics" } },
  { "_id": 3, "name": "Bulk orders", "criteria": { "quantity": { "$gte": 10 } } },
  { "_id": 4, "name": "Unfiltered" }
]
```

### Example 1: Which stored queries match this document

Find the alerts that a single order would trigger:

```javascript
db.alerts.aggregate([
  {
    $inverseMatch: {
      path: "criteria",
      input: { total: 750, category: "electronics", quantity: 2 }
    }
  },
  { $project: { name: 1 } }
])
```

```json
[
  { "_id": 1, "name": "High value orders" },
  { "_id": 2, "name": "Electronics" }
]
```

Alert 3 is dropped because the order's `quantity` is below its threshold, and alert 4 because it has no `criteria` field at all — `defaultResult` is `false` by default.

### Example 2: Keeping documents that store no query

Set `defaultResult` to `true` to treat a missing query as a match, which is how you model a catch-all rule:

```javascript
db.alerts.aggregate([
  {
    $inverseMatch: {
      path: "criteria",
      input: { total: 750, category: "electronics", quantity: 2 },
      defaultResult: true
    }
  },
  { $project: { name: 1 } }
])
```

```json
[
  { "_id": 1, "name": "High value orders" },
  { "_id": 2, "name": "Electronics" },
  { "_id": 4, "name": "Unfiltered" }
]
```

### Example 3: Taking the input from another collection

Use `from` and `pipeline` to test the stored queries against a document held elsewhere, rather than one written into the stage:

```javascript
db.alerts.aggregate([
  {
    $inverseMatch: {
      path: "criteria",
      from: "orders",
      pipeline: [
        { $match: { _id: 4001 } },
        { $project: { total: 1, category: 1, quantity: 1, _id: 0 } },
        { $limit: 1 }
      ]
    }
  },
  { $project: { name: 1 } }
])
```

The pipeline is restricted to `$match`, `$project` and `$limit`. Any other stage fails with:

```
<stage> is not allowed to be used within an $inverseMatch stage, only $match, $project or $limit are allowed
```

## Behavior

- **The stored value must be a query document.** If the value at `path` is present but is not a document, the stage errors rather than skipping the document.
- **A missing path is not an error.** When a document has no value at `path`, the stage uses `defaultResult` — `false` unless you set it otherwise — so those documents are dropped silently by default.
- **`input` is an expression, not just a literal.** A string is treated as a path expression, so `input: "$payload"` tests each stored query against that document's own `payload` field. To pass a literal document, write it inline as in the examples above.
- **`input` may be an array.** An array of documents is accepted as well as a single document.

## Related content

- [`$match`](https://documentdb.io/docs/reference/operators/aggregation/%24match/) — the usual direction: one query, many documents.
- [`$project`](https://documentdb.io/docs/reference/operators/aggregation/%24project/) — shape the surviving documents.
- [`$limit`](https://documentdb.io/docs/reference/operators/aggregation/%24limit/) — one of the three stages permitted inside `pipeline`.

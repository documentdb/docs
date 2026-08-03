---
title: $limit
description: The $limit stage in the aggregation pipeline restricts the number of documents passed to the next stage.
type: operators
category: aggregation
---

# $limit

The `$limit` stage in the aggregation pipeline restricts the number of documents passed on to the next stage. It is most often combined with `$sort` to return a top-N result, or placed after `$match` to cap the amount of work later stages do.

## Syntax

```javascript
{
  $limit: <positiveInteger>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`positiveInteger`** | The maximum number of documents to pass to the next stage. Must be a number that can be represented as a 64-bit integer, and must be greater than zero. |

## Examples

The examples on this page use the following documents in the `stores` collection.

```json
[
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "sales": { "totalSales": 75670 }
  },
  {
    "_id": "2e2f4c9e-6a1c-4a3a-9b52-7f1a5d0c2f11",
    "name": "Fourth Coffee | Beverage Shop - Lake Ellenmouth",
    "sales": { "totalSales": 21200 }
  },
  {
    "_id": "9d2c1a77-8f52-4a1e-9a55-3d4e6b8c0a22",
    "name": "Contoso | Beverage Shop - Port Alina",
    "sales": { "totalSales": 112400 }
  }
]
```

### Example 1: Returning the top results

This query sorts stores by total sales and returns only the two highest.

```javascript
db.stores.aggregate([
  { $sort: { "sales.totalSales": -1 } },
  { $project: { _id: 0, name: 1, "sales.totalSales": 1 } },
  { $limit: 2 }
])
```

This query returns the following result:

```json
[
  {
    "name": "Contoso | Beverage Shop - Port Alina",
    "sales": { "totalSales": 112400 }
  },
  {
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "sales": { "totalSales": 75670 }
  }
]
```

Place `$limit` *after* `$sort` when you want a top-N result. A `$limit` that runs before `$sort` truncates the input first, so the sort only orders the documents that survived.

### Example 2: Limiting to a single document

```javascript
db.stores.aggregate([
  { $match: { "sales.totalSales": { $gt: 50000 } } },
  { $limit: 1 },
  { $project: { _id: 0, name: 1 } }
])
```

This query returns the following result:

```json
[
  { "name": "First Up Consultants | Beverage Shop - Satterfieldmouth" }
]
```

Without a preceding `$sort`, which document survives is not guaranteed.

## Behavior

When a pipeline contains more than one `$limit`, the smallest value wins — a `$limit: 10` followed later by `$limit: 3` yields at most three documents.

## Error cases

| Specification | Error |
| --- | --- |
| `{ $limit: 0 }` | `The specified limit value must always be positive` |
| `{ $limit: -1 }` | `Invalid argument passed to the $skip stage: a non-negative number was expected but received in $limit: -1` |
| `{ $limit: "2" }` | `the limit must be specified as a number` |

A value that cannot be represented as a 64-bit integer is also rejected.

## Related

- [`$skip`](../%24skip/) — skips documents instead of truncating the result.
- [`$sort`](../%24sort/) — order documents before limiting them.

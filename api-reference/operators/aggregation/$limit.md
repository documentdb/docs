---
title: $limit
description: The $limit stage restricts the number of documents passed to the next stage in the pipeline.
type: operators
category: aggregation
---

# $limit

The `$limit` stage constrains the number of documents passed to the next stage in the aggregation pipeline. It passes the first *n* documents unmodified to the pipeline where *n* is the specified limit.

## Syntax

```javascript
{
  $limit: <positive integer>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`positive integer`** | Required. A positive integer specifying the maximum number of documents to pass to the next stage. Must be greater than zero. |

## Examples

Consider this sample document from the stores collection.

```json
{
  "_id": "2cf3f885-9962-4b67-a172-aa9039e9ae2f",
  "name": "First Up Consultants | Bed and Bath Center - South Amir",
  "location": {
    "lat": 60.7954,
    "lon": -142.0012
  },
  "staff": {
    "totalStaff": {
      "fullTime": 18,
      "partTime": 17
    }
  },
  "sales": {
    "totalSales": 37701
  }
}
```

### Example 1: Basic limit

Return only the first 3 documents from the collection:

```javascript
db.stores.aggregate([
  { $limit: 3 }
])
```

### Example 2: Sort and limit (Top-N pattern)

Find the top 2 stores by total sales:

```javascript
db.stores.aggregate([
  { $sort: { "sales.totalSales": -1 } },
  { $limit: 2 },
  { $project: { name: 1, "sales.totalSales": 1 } }
])
```

### Example 3: Limit with filtering

Get the first 5 stores with more than 10 full-time staff:

```javascript
db.stores.aggregate([
  { $match: { "staff.totalStaff.fullTime": { $gt: 10 } } },
  { $limit: 5 }
])
```

## Key Takeaways

- **Simple syntax** — takes only a single positive integer
- **Order dependent** — without a preceding `$sort`, the order of returned documents is undefined
- **Combines with `$sort`** — place `$sort` before `$limit` for deterministic top-N queries
- **Combines with `$skip`** — use `$skip` then `$limit` for pagination
- **Consecutive limits** — when multiple `$limit` stages appear consecutively, the smallest value is used

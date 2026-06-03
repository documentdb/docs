---
title: $count
description: The $count stage returns a count of the number of documents at this stage of the aggregation pipeline.
type: operators
category: aggregation
---

# $count

The `$count` stage passes a document to the next stage that contains a count of the number of documents input to the stage. This is useful for getting the total number of documents that match earlier pipeline stages.

## Syntax

```javascript
{
  $count: <string>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`string`** | Required. The name of the output field which has the count as its value. Must be a non-empty string, must not start with `$`, and must not contain the `.` character. |

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

### Example 1: Count all documents

Count the total number of documents in the collection:

```javascript
db.stores.aggregate([
  { $count: "totalStores" }
])
```

This query returns:

```json
[
  { "totalStores": 5 }
]
```

### Example 2: Count documents after filtering

Count stores that have more than 10 full-time staff:

```javascript
db.stores.aggregate([
  { $match: { "staff.totalStaff.fullTime": { $gt: 10 } } },
  { $count: "highStaffStores" }
])
```

This query returns:

```json
[
  { "highStaffStores": 3 }
]
```

### Example 3: Count with $unwind

Count the total number of promotion events across all stores:

```javascript
db.stores.aggregate([
  { $unwind: "$promotionEvents" },
  { $count: "totalPromotionEvents" }
])
```

## Key Takeaways

- **Single output document** — `$count` always returns exactly one document with a single field
- **Equivalent to `$group`** — `{ $count: "total" }` is equivalent to `{ $group: { _id: null, total: { $sum: 1 } } }` followed by `{ $project: { _id: 0 } }`
- **Field name restrictions** — the output field name must be non-empty, cannot start with `$`, and cannot contain `.`

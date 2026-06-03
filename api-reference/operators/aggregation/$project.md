---
title: $project
description: The $project stage reshapes each document by including, excluding, or computing new fields.
type: operators
category: aggregation
---

# $project

The `$project` stage reshapes each document in the stream by including, excluding, or adding new fields. It can be used to rename fields, compute new values, and control which fields appear in the output documents.

## Syntax

```javascript
{
  $project: {
    <field1>: <1 or true>,
    <field2>: <0 or false>,
    <field3>: <expression>,
    ...
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`field: 1` or `field: true`** | Include the field in the output. |
| **`field: 0` or `field: false`** | Exclude the field from the output. |
| **`field: <expression>`** | Add a new field or reset the value of an existing field using an aggregation expression. |
| **`_id: 0`** | Suppress the `_id` field. By default, `_id` is always included. |

## Examples

Consider this sample document from the stores collection.

```json
{
  "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
  "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
  "location": {
    "lat": -89.2384,
    "lon": -46.4012
  },
  "staff": {
    "totalStaff": {
      "fullTime": 8,
      "partTime": 20
    }
  },
  "sales": {
    "totalSales": 75670,
    "salesByCategory": [
      { "categoryName": "Wine Accessories", "totalSales": 34440 },
      { "categoryName": "Bitters", "totalSales": 39496 },
      { "categoryName": "Rum", "totalSales": 1734 }
    ]
  }
}
```

### Example 1: Include specific fields

Return only the store name and total sales:

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      "sales.totalSales": 1
    }
  },
  { $limit: 2 }
])
```

This query returns:

```json
[
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "sales": { "totalSales": 75670 }
  }
]
```

### Example 2: Exclude specific fields

Return all fields except location and promotion events:

```javascript
db.stores.aggregate([
  {
    $project: {
      location: 0,
      promotionEvents: 0
    }
  },
  { $limit: 2 }
])
```

### Example 3: Compute new fields

Calculate total staff and staff ratio:

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      totalStaff: {
        $add: ["$staff.totalStaff.fullTime", "$staff.totalStaff.partTime"]
      },
      fullTimeRatio: {
        $round: [
          {
            $divide: [
              "$staff.totalStaff.fullTime",
              { $add: ["$staff.totalStaff.fullTime", "$staff.totalStaff.partTime"] }
            ]
          },
          2
        ]
      }
    }
  },
  { $limit: 2 }
])
```

This query returns:

```json
[
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "totalStaff": 28,
    "fullTimeRatio": 0.29
  }
]
```

### Example 4: Suppress `_id` and rename fields

```javascript
db.stores.aggregate([
  {
    $project: {
      _id: 0,
      storeName: "$name",
      revenue: "$sales.totalSales"
    }
  },
  { $limit: 2 }
])
```

## Key Takeaways

- **Cannot mix inclusion and exclusion** — you cannot combine include (`1`) and exclude (`0`) specifications in the same `$project` stage, except for `_id: 0`
- **`_id` included by default** — explicitly set `_id: 0` to suppress it
- **Supports expressions** — use any aggregation expression to compute new field values
- **Supports dot notation** — include or exclude nested fields using dot notation (e.g., `"sales.totalSales": 1`)
- **Supports `let` variables** — pipeline variables defined with `let` can be referenced in expressions

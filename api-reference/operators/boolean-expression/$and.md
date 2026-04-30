---
title: $and
description: The $and aggregation operator evaluates one or more expressions and returns true if all evaluate to true.
type: operators
category: boolean-expression
---

# $and

The `$and` aggregation expression operator evaluates one or more expressions and returns `true` only if all of the expressions evaluate to `true`. Otherwise, it returns `false`. This operator is different from the `$and` query operator; it is used within aggregation expressions such as `$project`, `$addFields`, and `$match` with `$expr`.

## Syntax

```javascript
{
  $and: [ <expression1>, <expression2>, ... ]
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`expression`** | An array of valid expressions. Each expression can be any valid aggregation expression that resolves to a boolean value. |

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
        "totalSales": 47510,
        "salesByCategory": [
            {
                "categoryName": "Wine Accessories",
                "totalSales": 34450
            },
            {
                "categoryName": "Bitters",
                "totalSales": 39496
            },
            {
                "categoryName": "Rum",
                "totalSales": 1734
            }
        ]
    },
    "promotionEvents": [
        {
            "eventName": "Summer Sizzler",
            "promotionalDates": {
                "startDate": {
                    "$date": "2024-08-01T00:00:00Z"
                },
                "endDate": {
                    "$date": "2024-08-31T00:00:00Z"
                }
            },
            "discountPercentage": 15
        },
        {
            "eventName": "Oktoberfest Specials",
            "promotionalDates": {
                "startDate": {
                    "$date": "2024-10-01T00:00:00Z"
                },
                "endDate": {
                    "$date": "2024-10-31T00:00:00Z"
                }
            },
            "discountPercentage": 20
        }
    ]
}
```

### Example 1: Using `$and` in `$project` to evaluate multiple conditions

This query adds a computed field that checks whether a store has both high sales (greater than 40000) and a large full-time staff (greater than 5).

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      isHighPerformer: {
        $and: [
          { $gt: ["$sales.totalSales", 40000] },
          { $gt: ["$staff.totalStaff.fullTime", 5] }
        ]
      }
    }
  },
  { $limit: 3 }
])
```

This query returns the following results:

```json
[
    {
        "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
        "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
        "isHighPerformer": true
    },
    {
        "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74",
        "name": "Proseware, Inc. | Home Entertainment Hub - East Linwoodbury",
        "isHighPerformer": false
    }
]
```

### Example 2: Using `$and` in `$addFields` with three conditions

This query adds a field that evaluates whether a store meets all three criteria: sales above 30000, more than 5 full-time staff, and more than 10 part-time staff.

```javascript
db.stores.aggregate([
  {
    $addFields: {
      isFullyStaffedAndProfitable: {
        $and: [
          { $gt: ["$sales.totalSales", 30000] },
          { $gt: ["$staff.totalStaff.fullTime", 5] },
          { $gt: ["$staff.totalStaff.partTime", 10] }
        ]
      }
    }
  },
  {
    $project: {
      name: 1,
      isFullyStaffedAndProfitable: 1
    }
  },
  { $limit: 3 }
])
```

This query returns the following results:

```json
[
    {
        "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
        "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
        "isFullyStaffedAndProfitable": true
    },
    {
        "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74",
        "name": "Proseware, Inc. | Home Entertainment Hub - East Linwoodbury",
        "isFullyStaffedAndProfitable": false
    }
]
```

---
title: $not
description: The $not aggregation operator returns the boolean opposite of its argument expression.
type: operators
category: boolean-expression
---

# $not

The `$not` aggregation expression operator returns the boolean opposite of its argument expression. It accepts a single-element array containing the expression to evaluate. This operator is different from the `$not` query operator; it is used within aggregation expressions such as `$project`, `$addFields`, and `$match` with `$expr`.

## Syntax

```javascript
{
  $not: [ <expression> ]
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`expression`** | A valid aggregation expression that resolves to a boolean value. Must be provided as a single-element array. |

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

### Example 1: Using `$not` in `$project` to negate a condition

This query adds a computed field that identifies stores that do not have high sales (i.e., sales not greater than 50000).

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      isNotHighSales: {
        $not: [
          { $gt: ["$sales.totalSales", 50000] }
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
        "isNotHighSales": true
    },
    {
        "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74",
        "name": "Proseware, Inc. | Home Entertainment Hub - East Linwoodbury",
        "isNotHighSales": true
    }
]
```

### Example 2: Using `$not` with `$and` for combined logic

This query adds a field to flag stores that are not both high-sales and well-staffed. It combines `$not` with `$and` to negate a compound condition.

```javascript
db.stores.aggregate([
  {
    $addFields: {
      isNotTopPerformer: {
        $not: [
          {
            $and: [
              { $gt: ["$sales.totalSales", 40000] },
              { $gt: ["$staff.totalStaff.fullTime", 10] }
            ]
          }
        ]
      }
    }
  },
  {
    $project: {
      name: 1,
      "sales.totalSales": 1,
      "staff.totalStaff.fullTime": 1,
      isNotTopPerformer: 1
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
        "staff": {
            "totalStaff": {
                "fullTime": 8
            }
        },
        "sales": {
            "totalSales": 47510
        },
        "isNotTopPerformer": true
    },
    {
        "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74",
        "name": "Proseware, Inc. | Home Entertainment Hub - East Linwoodbury",
        "staff": {
            "totalStaff": {
                "fullTime": 14
            }
        },
        "sales": {
            "totalSales": 35150
        },
        "isNotTopPerformer": true
    }
]
```

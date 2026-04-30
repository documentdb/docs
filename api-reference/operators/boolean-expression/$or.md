---
title: $or
description: The $or aggregation operator evaluates one or more expressions and returns true if any evaluate to true.
type: operators
category: boolean-expression
---

# $or

The `$or` aggregation expression operator evaluates one or more expressions and returns `true` if any of the expressions evaluate to `true`. Otherwise, it returns `false`. This operator is different from the `$or` query operator; it is used within aggregation expressions such as `$project`, `$addFields`, and `$match` with `$expr`.

## Syntax

```javascript
{
  $or: [ <expression1>, <expression2>, ... ]
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

### Example 1: Using `$or` in `$project` to evaluate multiple conditions

This query adds a computed field that checks whether a store has either high sales (greater than 100000) or a large staff (more than 15 full-time employees).

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      isNotable: {
        $or: [
          { $gt: ["$sales.totalSales", 100000] },
          { $gt: ["$staff.totalStaff.fullTime", 15] }
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
        "isNotable": false
    },
    {
        "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74",
        "name": "Proseware, Inc. | Home Entertainment Hub - East Linwoodbury",
        "isNotable": false
    }
]
```

### Example 2: Using `$or` in `$addFields` to categorize documents

This query adds a field that flags stores needing attention if they have either very low sales (less than 10000) or very few full-time staff (less than 3).

```javascript
db.stores.aggregate([
  {
    $addFields: {
      needsAttention: {
        $or: [
          { $lt: ["$sales.totalSales", 10000] },
          { $lt: ["$staff.totalStaff.fullTime", 3] }
        ]
      }
    }
  },
  {
    $project: {
      name: 1,
      "sales.totalSales": 1,
      "staff.totalStaff.fullTime": 1,
      needsAttention: 1
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
        "needsAttention": false
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
        "needsAttention": false
    }
]
```

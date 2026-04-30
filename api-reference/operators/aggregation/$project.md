---
title: $project
description: The $project stage in the aggregation pipeline is used to reshape documents by including, excluding, or computing new fields.
type: operators
category: aggregation
---

# $project

The `$project` stage in the aggregation pipeline is used to reshape documents by including, excluding, or adding computed fields. It controls which fields appear in the output documents, making it essential for transforming document structure and reducing the amount of data passed to subsequent pipeline stages.

## Syntax

```javascript
{
  $project: {
    <field1>: <1 or 0 or expression>,
    <field2>: <1 or 0 or expression>,
    ...
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`field`** | The name of the field to include, exclude, or compute. |
| **`1`** | Includes the field in the output documents. |
| **`0`** | Excludes the field from the output documents. |
| **`expression`** | An aggregation expression that computes a new value for the field. |

> **Note:** The `_id` field is included by default unless explicitly excluded with `_id: 0`. You cannot mix inclusion and exclusion in the same `$project` stage, except for the `_id` field.

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
            {
                "categoryName": "Wine Accessories",
                "totalSales": 34440
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
            "eventName": "Unbeatable Bargain Bash",
            "promotionalDates": {
                "startDate": {
                    "Year": 2024,
                    "Month": 6,
                    "Day": 23
                },
                "endDate": {
                    "Year": 2024,
                    "Month": 7,
                    "Day": 2
                }
            },
            "discounts": [
                {
                    "categoryName": "Whiskey",
                    "discountPercentage": 7
                },
                {
                    "categoryName": "Bitters",
                    "discountPercentage": 15
                },
                {
                    "categoryName": "Brandy",
                    "discountPercentage": 8
                },
                {
                    "categoryName": "Sports Drinks",
                    "discountPercentage": 22
                },
                {
                    "categoryName": "Vodka",
                    "discountPercentage": 19
                }
            ]
        },
        {
            "eventName": "Steal of a Deal Days",
            "promotionalDates": {
                "startDate": {
                    "Year": 2024,
                    "Month": 9,
                    "Day": 21
                },
                "endDate": {
                    "Year": 2024,
                    "Month": 9,
                    "Day": 29
                }
            },
            "discounts": [
                {
                    "categoryName": "Organic Wine",
                    "discountPercentage": 19
                },
                {
                    "categoryName": "White Wine",
                    "discountPercentage": 20
                },
                {
                    "categoryName": "Sparkling Wine",
                    "discountPercentage": 19
                },
                {
                    "categoryName": "Whiskey",
                    "discountPercentage": 17
                },
                {
                    "categoryName": "Vodka",
                    "discountPercentage": 23
                }
            ]
        }
    ]
}
```

### Example 1: Including specific fields

To return only the store name and total sales while excluding the `_id` field:

```javascript
db.stores.aggregate([
  {
    $project: {
      _id: 0,
      name: 1,
      totalSales: "$sales.totalSales"
    }
  },
  { $limit: 2 }
])
```

The first two results returned by this query are:

```json
[
  {
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "totalSales": 75670
  },
  {
    "name": "Lakeshore Retail | DJ Equipment Stop - Port Cecile",
    "totalSales": 47510
  }
]
```

### Example 2: Computing new fields with expressions

To compute the total number of staff and the number of sales categories for each store:

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      totalStaff: {
        $add: ["$staff.totalStaff.fullTime", "$staff.totalStaff.partTime"]
      },
      numberOfCategories: {
        $size: "$sales.salesByCategory"
      },
      totalSales: "$sales.totalSales"
    }
  },
  { $limit: 2 }
])
```

The first two results returned by this query are:

```json
[
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "totalStaff": 28,
    "numberOfCategories": 3,
    "totalSales": 75670
  },
  {
    "_id": "7954bd5c-9ac2-4c10-bb7a-2b79bd0963c5",
    "name": "Lakeshore Retail | DJ Equipment Stop - Port Cecile",
    "totalStaff": 15,
    "numberOfCategories": 2,
    "totalSales": 47510
  }
]
```

### Example 3: Excluding specific fields

To return all fields except the `location` and `promotionEvents` fields:

```javascript
db.stores.aggregate([
  {
    $project: {
      location: 0,
      promotionEvents: 0
    }
  },
  { $limit: 1 }
])
```

This query returns the following result:

```json
[
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "staff": {
      "totalStaff": {
        "fullTime": 8,
        "partTime": 20
      }
    },
    "sales": {
      "totalSales": 75670,
      "salesByCategory": [
        {
          "categoryName": "Wine Accessories",
          "totalSales": 34440
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
    }
  }
]
```

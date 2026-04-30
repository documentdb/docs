---
title: $concat
description: The $concat operator concatenates two or more strings and returns the resulting string.
type: operators
category: string-expression
---

# $concat

The `$concat` operator concatenates two or more strings and returns the resulting string. If any argument resolves to `null`, `$concat` returns `null`.

## Syntax

```javascript
{
  $concat: [ <expression1>, <expression2>, ... ]
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<expression1>, <expression2>, ...`** | An array of expressions that resolve to strings. The expressions are concatenated in the order they appear. If any expression resolves to `null`, the result is `null`. |

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

### Example 1 - Concatenate store name with a label

Use `$concat` to create a formatted label that combines the store name with additional descriptive text.

```javascript
db.stores.aggregate([
  {
    $project: {
      storeLabel: {
        $concat: [ "Store: ", "$name" ]
      }
    }
  }
])
```

This query returns the following result:

```json
[
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "storeLabel": "Store: First Up Consultants | Beverage Shop - Satterfieldmouth"
  }
]
```

### Example 2 - Concatenate multiple fields into a summary

Use `$concat` with `$toString` to build a summary string combining the store name and total sales.

```javascript
db.stores.aggregate([
  {
    $project: {
      summary: {
        $concat: [
          "$name",
          " - Total Sales: $",
          { $toString: "$sales.totalSales" }
        ]
      }
    }
  }
])
```

This query returns the following result:

```json
[
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "summary": "First Up Consultants | Beverage Shop - Satterfieldmouth - Total Sales: $47510"
  }
]
```

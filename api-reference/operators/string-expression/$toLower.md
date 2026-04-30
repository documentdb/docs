---
title: $toLower
description: The $toLower operator converts a string to lowercase and returns the resulting string.
type: operators
category: string-expression
---

# $toLower

The `$toLower` operator converts a string to lowercase and returns the resulting string. If the argument resolves to `null`, `$toLower` returns an empty string.

## Syntax

```javascript
{
  $toLower: <expression>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<expression>`** | Any expression that resolves to a string. If the expression resolves to `null`, `$toLower` returns an empty string. |

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

### Example 1 - Convert store name to lowercase

Use `$toLower` to normalize the store name to lowercase for consistent comparison or display.

```javascript
db.stores.aggregate([
  {
    $project: {
      lowercaseName: {
        $toLower: "$name"
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
    "lowercaseName": "first up consultants | beverage shop - satterfieldmouth"
  }
]
```

### Example 2 - Convert category names to lowercase

Use `$toLower` within a `$map` expression to convert all category names to lowercase.

```javascript
db.stores.aggregate([
  {
    $project: {
      normalizedCategories: {
        $map: {
          input: "$sales.salesByCategory",
          as: "category",
          in: {
            categoryName: {
              $toLower: "$$category.categoryName"
            },
            totalSales: "$$category.totalSales"
          }
        }
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
    "normalizedCategories": [
      {
        "categoryName": "wine accessories",
        "totalSales": 34450
      },
      {
        "categoryName": "bitters",
        "totalSales": 39496
      },
      {
        "categoryName": "rum",
        "totalSales": 1734
      }
    ]
  }
]
```

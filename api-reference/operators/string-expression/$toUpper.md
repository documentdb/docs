---
title: $toUpper
description: The $toUpper operator converts a string to uppercase and returns the resulting string.
type: operators
category: string-expression
---

# $toUpper

The `$toUpper` operator converts a string to uppercase and returns the resulting string. If the argument resolves to `null`, `$toUpper` returns an empty string.

## Syntax

```javascript
{
  $toUpper: <expression>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<expression>`** | Any expression that resolves to a string. If the expression resolves to `null`, `$toUpper` returns an empty string. |

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

### Example 1 - Convert store name to uppercase

Use `$toUpper` to convert the store name to uppercase for display or reporting.

```javascript
db.stores.aggregate([
  {
    $project: {
      uppercaseName: {
        $toUpper: "$name"
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
    "uppercaseName": "FIRST UP CONSULTANTS | BEVERAGE SHOP - SATTERFIELDMOUTH"
  }
]
```

### Example 2 - Convert event names to uppercase

Use `$toUpper` within a `$map` expression to convert all promotion event names to uppercase.

```javascript
db.stores.aggregate([
  {
    $project: {
      uppercaseEvents: {
        $map: {
          input: "$promotionEvents",
          as: "event",
          in: {
            eventName: {
              $toUpper: "$$event.eventName"
            },
            discountPercentage: "$$event.discountPercentage"
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
    "uppercaseEvents": [
      {
        "eventName": "SUMMER SIZZLER",
        "discountPercentage": 15
      },
      {
        "eventName": "OKTOBERFEST SPECIALS",
        "discountPercentage": 20
      }
    ]
  }
]
```

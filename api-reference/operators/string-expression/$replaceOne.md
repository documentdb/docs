---
title: $replaceOne
description: The $replaceOne operator replaces the first occurrence of a search string with a replacement string.
type: operators
category: string-expression
---

# $replaceOne

The `$replaceOne` operator replaces the first instance of a search string in an input string with a replacement string. If the search string is not found, `$replaceOne` returns the input string unchanged.

## Syntax

```javascript
{
  $replaceOne: {
    input: <string>,
    find: <string>,
    replacement: <string>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`input`** | The string on which to perform the replacement. Can be any expression that resolves to a string. |
| **`find`** | The substring to search for within the input string. Can be any expression that resolves to a string. |
| **`replacement`** | The string to substitute in place of the first occurrence of `find`. Can be any expression that resolves to a string. |

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

### Example 1 - Replace the pipe separator in the store name

Use `$replaceOne` to replace the pipe separator with a dash in the store name.

```javascript
db.stores.aggregate([
  {
    $project: {
      updatedName: {
        $replaceOne: {
          input: "$name",
          find: " | ",
          replacement: " - "
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
    "updatedName": "First Up Consultants - Beverage Shop - Satterfieldmouth"
  }
]
```

### Example 2 - Replace a word in event names

Use `$replaceOne` within a `$map` to replace "Specials" with "Deals" in event names.

```javascript
db.stores.aggregate([
  {
    $project: {
      updatedEvents: {
        $map: {
          input: "$promotionEvents",
          as: "event",
          in: {
            eventName: {
              $replaceOne: {
                input: "$$event.eventName",
                find: "Specials",
                replacement: "Deals"
              }
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
    "updatedEvents": [
      {
        "eventName": "Summer Sizzler",
        "discountPercentage": 15
      },
      {
        "eventName": "Oktoberfest Deals",
        "discountPercentage": 20
      }
    ]
  }
]
```

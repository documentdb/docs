---
title: $substrCP
description: The $substrCP operator returns a substring of a string based on the specified UTF-8 code point index and length.
type: operators
category: string-expression
---

# $substrCP

The `$substrCP` operator returns a substring of a string, starting at a specified UTF-8 code point index position for a specified number of code points. The index is zero-based.

## Syntax

```javascript
{
  $substrCP: [ <string>, <start>, <length> ]
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<string>`** | The string from which to extract the substring. Can be any expression that resolves to a string. |
| **`<start>`** | The zero-based code point index position at which to start the substring. Can be any expression that resolves to a non-negative integer. |
| **`<length>`** | The number of code points to include in the substring. Can be any expression that resolves to a non-negative integer or number. |

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

### Example 1 - Extract the first word of the store name

Use `$substrCP` to extract the first 5 code points from the store name.

```javascript
db.stores.aggregate([
  {
    $project: {
      firstWord: {
        $substrCP: [ "$name", 0, 5 ]
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
    "firstWord": "First"
  }
]
```

### Example 2 - Extract a portion of category names

Use `$substrCP` within a `$map` to extract the first 4 code points of each category name.

```javascript
db.stores.aggregate([
  {
    $project: {
      categoryPrefixes: {
        $map: {
          input: "$sales.salesByCategory",
          as: "category",
          in: {
            prefix: {
              $substrCP: [ "$$category.categoryName", 0, 4 ]
            },
            categoryName: "$$category.categoryName"
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
    "categoryPrefixes": [
      {
        "prefix": "Wine",
        "categoryName": "Wine Accessories"
      },
      {
        "prefix": "Bitt",
        "categoryName": "Bitters"
      },
      {
        "prefix": "Rum",
        "categoryName": "Rum"
      }
    ]
  }
]
```

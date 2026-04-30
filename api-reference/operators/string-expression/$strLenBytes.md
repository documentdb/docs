---
title: $strLenBytes
description: The $strLenBytes operator returns the number of UTF-8 encoded bytes in a string.
type: operators
category: string-expression
---

# $strLenBytes

The `$strLenBytes` operator returns the number of UTF-8 encoded bytes in the specified string.

## Syntax

```javascript
{
  $strLenBytes: <expression>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<expression>`** | Any expression that resolves to a string. If the argument resolves to a value of `null` or refers to a missing field, `$strLenBytes` returns an error. |

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

### Example 1 - Get the byte length of the store name

Use `$strLenBytes` to determine the number of UTF-8 bytes in the store name.

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      nameByteLength: {
        $strLenBytes: "$name"
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
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "nameByteLength": 55
  }
]
```

### Example 2 - Get the byte length of each category name

Use `$strLenBytes` within a `$map` to calculate the byte length of each sales category name.

```javascript
db.stores.aggregate([
  {
    $project: {
      categoryLengths: {
        $map: {
          input: "$sales.salesByCategory",
          as: "category",
          in: {
            categoryName: "$$category.categoryName",
            byteLength: {
              $strLenBytes: "$$category.categoryName"
            }
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
    "categoryLengths": [
      {
        "categoryName": "Wine Accessories",
        "byteLength": 16
      },
      {
        "categoryName": "Bitters",
        "byteLength": 7
      },
      {
        "categoryName": "Rum",
        "byteLength": 3
      }
    ]
  }
]
```

---
title: $strLenCP
description: The $strLenCP operator returns the number of UTF-8 code points in a string.
type: operators
category: string-expression
---

# $strLenCP

The `$strLenCP` operator returns the number of UTF-8 code points in the specified string.

## Syntax

```javascript
{
  $strLenCP: <expression>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<expression>`** | Any expression that resolves to a string. If the argument resolves to a value of `null` or refers to a missing field, `$strLenCP` returns an error. |

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

### Example 1 - Get the code point length of the store name

Use `$strLenCP` to determine the number of code points in the store name.

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      nameCodePointLength: {
        $strLenCP: "$name"
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
    "nameCodePointLength": 55
  }
]
```

### Example 2 - Get the code point length of each event name

Use `$strLenCP` within a `$map` to calculate the code point length of each promotion event name.

```javascript
db.stores.aggregate([
  {
    $project: {
      eventNameLengths: {
        $map: {
          input: "$promotionEvents",
          as: "event",
          in: {
            eventName: "$$event.eventName",
            codePointLength: {
              $strLenCP: "$$event.eventName"
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
    "eventNameLengths": [
      {
        "eventName": "Summer Sizzler",
        "codePointLength": 14
      },
      {
        "eventName": "Oktoberfest Specials",
        "codePointLength": 20
      }
    ]
  }
]
```

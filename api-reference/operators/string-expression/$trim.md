---
title: $trim
description: The $trim operator removes whitespace or specified characters from the beginning and end of a string.
type: operators
category: string-expression
---

# $trim

The `$trim` operator removes whitespace or the specified characters from the beginning and end of a string. If no characters are specified, `$trim` removes whitespace characters including the null character.

## Syntax

```javascript
{
  $trim: {
    input: <string>,
    chars: <string>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`input`** | The string to trim. Can be any expression that resolves to a string. |
| **`chars`** | Optional. The characters to remove from the beginning and end of the `input` string. If omitted, `$trim` removes whitespace characters. |

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

### Example 1 - Trim whitespace from a constructed string

Use `$trim` to remove leading and trailing whitespace from a string created by `$concat`.

```javascript
db.stores.aggregate([
  {
    $project: {
      trimmedName: {
        $trim: {
          input: {
            $concat: [ "  ", "$name", "  " ]
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
    "trimmedName": "First Up Consultants | Beverage Shop - Satterfieldmouth"
  }
]
```

### Example 2 - Trim specific characters from a string

Use `$trim` with the `chars` parameter to remove specific characters from the store name.

```javascript
db.stores.aggregate([
  {
    $project: {
      eventName: {
        $trim: {
          input: { $arrayElemAt: [ "$promotionEvents.eventName", 0 ] },
          chars: "Sumer"
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
    "eventName": " Sizzl"
  }
]
```

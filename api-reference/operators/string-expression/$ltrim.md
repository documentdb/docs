---
title: $ltrim
description: The $ltrim operator removes whitespace or specified characters from the beginning of a string.
type: operators
category: string-expression
---

# $ltrim

The `$ltrim` operator removes whitespace or the specified characters from the beginning (left side) of a string. If no characters are specified, `$ltrim` removes whitespace characters including the null character.

## Syntax

```javascript
{
  $ltrim: {
    input: <string>,
    chars: <string>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`input`** | The string to trim. Can be any expression that resolves to a string. |
| **`chars`** | Optional. The characters to remove from the beginning of the `input` string. If omitted, `$ltrim` removes whitespace characters. |

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

### Example 1 - Remove leading whitespace from a string

Use `$ltrim` to remove leading whitespace from a string that has been padded.

```javascript
db.stores.aggregate([
  {
    $project: {
      trimmedName: {
        $ltrim: {
          input: {
            $concat: [ "   ", "$name" ]
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

### Example 2 - Remove specific leading characters

Use `$ltrim` with the `chars` parameter to strip specific leading characters from event names.

```javascript
db.stores.aggregate([
  {
    $project: {
      events: {
        $map: {
          input: "$promotionEvents",
          as: "event",
          in: {
            trimmedName: {
              $ltrim: {
                input: "$$event.eventName",
                chars: "SOk"
              }
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
    "events": [
      {
        "trimmedName": "ummer Sizzler"
      },
      {
        "trimmedName": "toberfest Specials"
      }
    ]
  }
]
```

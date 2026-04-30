---
title: $regexMatch
description: The $regexMatch operator applies a regular expression to a string and returns a boolean indicating if the pattern matches.
type: operators
category: string-expression
---

# $regexMatch

The `$regexMatch` operator applies a regular expression pattern to a string and returns `true` if the pattern matches, or `false` otherwise.

## Syntax

```javascript
{
  $regexMatch: {
    input: <string>,
    regex: <pattern>,
    options: <string>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`input`** | The string on which to apply the regex pattern. Can be any expression that resolves to a string. |
| **`regex`** | The regex pattern to apply. Can be a string or a regex object. |
| **`options`** | Optional. Regex options such as `"i"` for case-insensitive, `"m"` for multiline, `"x"` for extended, and `"s"` for dot-all. |

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

### Example 1 - Check if the store name contains "Beverage"

Use `$regexMatch` to test whether the store name contains the word "Beverage".

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      isBeverageShop: {
        $regexMatch: {
          input: "$name",
          regex: "Beverage",
          options: "i"
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
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "isBeverageShop": true
  }
]
```

### Example 2 - Filter events by name pattern

Use `$regexMatch` within a `$filter` to find promotion events whose names start with "S".

```javascript
db.stores.aggregate([
  {
    $project: {
      summerEvents: {
        $filter: {
          input: "$promotionEvents",
          as: "event",
          cond: {
            $regexMatch: {
              input: "$$event.eventName",
              regex: "^S"
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
    "summerEvents": [
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
      }
    ]
  }
]
```

---
title: $regexFind
description: The $regexFind operator applies a regular expression to a string and returns the first match.
type: operators
category: string-expression
---

# $regexFind

The `$regexFind` operator applies a regular expression pattern to a string and returns information on the first matched substring. If no match is found, it returns `null`.

## Syntax

```javascript
{
  $regexFind: {
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

### Example 1 - Find the first word in the store name

Use `$regexFind` to extract the first word from the store name using a regex pattern.

```javascript
db.stores.aggregate([
  {
    $project: {
      firstWord: {
        $regexFind: {
          input: "$name",
          regex: "^\\w+"
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
    "firstWord": {
      "match": "First",
      "idx": 0,
      "captures": []
    }
  }
]
```

### Example 2 - Find a pattern with capture groups

Use `$regexFind` with capture groups to extract the company name and shop type from the store name.

```javascript
db.stores.aggregate([
  {
    $project: {
      nameComponents: {
        $regexFind: {
          input: "$name",
          regex: "^(.+?) \\| (.+?) -",
          options: ""
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
    "nameComponents": {
      "match": "First Up Consultants | Beverage Shop -",
      "idx": 0,
      "captures": [
        "First Up Consultants",
        "Beverage Shop"
      ]
    }
  }
]
```

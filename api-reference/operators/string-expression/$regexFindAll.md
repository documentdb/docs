---
title: $regexFindAll
description: The $regexFindAll operator applies a regular expression to a string and returns all matches.
type: operators
category: string-expression
---

# $regexFindAll

The `$regexFindAll` operator applies a regular expression pattern to a string and returns an array of information for all matched substrings. If no match is found, it returns an empty array.

## Syntax

```javascript
{
  $regexFindAll: {
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

### Example 1 - Find all words in the store name

Use `$regexFindAll` to extract all individual words from the store name.

```javascript
db.stores.aggregate([
  {
    $project: {
      words: {
        $regexFindAll: {
          input: "$name",
          regex: "\\w+"
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
    "words": [
      { "match": "First", "idx": 0, "captures": [] },
      { "match": "Up", "idx": 6, "captures": [] },
      { "match": "Consultants", "idx": 9, "captures": [] },
      { "match": "Beverage", "idx": 23, "captures": [] },
      { "match": "Shop", "idx": 32, "captures": [] },
      { "match": "Satterfieldmouth", "idx": 39, "captures": [] }
    ]
  }
]
```

### Example 2 - Find all capitalized words in event names

Use `$regexFindAll` to extract all words that start with an uppercase letter from event names.

```javascript
db.stores.aggregate([
  {
    $project: {
      eventCapitalizedWords: {
        $map: {
          input: "$promotionEvents",
          as: "event",
          in: {
            eventName: "$$event.eventName",
            capitalizedWords: {
              $regexFindAll: {
                input: "$$event.eventName",
                regex: "[A-Z][a-z]+"
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
    "eventCapitalizedWords": [
      {
        "eventName": "Summer Sizzler",
        "capitalizedWords": [
          { "match": "Summer", "idx": 0, "captures": [] },
          { "match": "Sizzler", "idx": 7, "captures": [] }
        ]
      },
      {
        "eventName": "Oktoberfest Specials",
        "capitalizedWords": [
          { "match": "Oktoberfest", "idx": 0, "captures": [] },
          { "match": "Specials", "idx": 12, "captures": [] }
        ]
      }
    ]
  }
]
```

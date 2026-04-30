---
title: $strcasecmp
description: The $strcasecmp operator performs a case-insensitive comparison of two strings and returns an integer.
type: operators
category: string-expression
---

# $strcasecmp

The `$strcasecmp` operator performs a case-insensitive comparison of two strings. It returns `0` if the strings are equal, `1` if the first string is greater, and `-1` if the second string is greater.

## Syntax

```javascript
{
  $strcasecmp: [ <expression1>, <expression2> ]
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<expression1>`** | The first string expression to compare. |
| **`<expression2>`** | The second string expression to compare. |

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

### Example 1 - Compare event name with a target string

Use `$strcasecmp` to perform a case-insensitive comparison of event names against a target string.

```javascript
db.stores.aggregate([
  {
    $project: {
      eventComparisons: {
        $map: {
          input: "$promotionEvents",
          as: "event",
          in: {
            eventName: "$$event.eventName",
            comparedToSummer: {
              $strcasecmp: [ "$$event.eventName", "SUMMER SIZZLER" ]
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
    "eventComparisons": [
      {
        "eventName": "Summer Sizzler",
        "comparedToSummer": 0
      },
      {
        "eventName": "Oktoberfest Specials",
        "comparedToSummer": -1
      }
    ]
  }
]
```

### Example 2 - Compare category names case-insensitively

Use `$strcasecmp` to check if a category name matches a target value regardless of case.

```javascript
db.stores.aggregate([
  {
    $project: {
      categories: {
        $map: {
          input: "$sales.salesByCategory",
          as: "category",
          in: {
            categoryName: "$$category.categoryName",
            isRum: {
              $eq: [
                { $strcasecmp: [ "$$category.categoryName", "rum" ] },
                0
              ]
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
    "categories": [
      {
        "categoryName": "Wine Accessories",
        "isRum": false
      },
      {
        "categoryName": "Bitters",
        "isRum": false
      },
      {
        "categoryName": "Rum",
        "isRum": true
      }
    ]
  }
]
```

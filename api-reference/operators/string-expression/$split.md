---
title: $split
description: The $split operator divides a string into an array of substrings based on a delimiter.
type: operators
category: string-expression
---

# $split

The `$split` operator divides a string into an array of substrings based on a specified delimiter. The operator returns an array of strings.

## Syntax

```javascript
{
  $split: [ <string>, <delimiter> ]
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<string>`** | The string to split. Can be any expression that resolves to a string. |
| **`<delimiter>`** | The delimiter to use for splitting the string. Can be any expression that resolves to a string. |

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

### Example 1 - Split the store name by the pipe delimiter

Use `$split` to separate the company name from the shop name using the pipe character as a delimiter.

```javascript
db.stores.aggregate([
  {
    $project: {
      nameParts: {
        $split: [ "$name", " | " ]
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
    "nameParts": [
      "First Up Consultants",
      "Beverage Shop - Satterfieldmouth"
    ]
  }
]
```

### Example 2 - Split and extract the shop location

Use `$split` to separate the shop portion of the name and extract the location after the dash.

```javascript
db.stores.aggregate([
  {
    $project: {
      shopAndLocation: {
        $split: [
          { $arrayElemAt: [ { $split: [ "$name", " | " ] }, 1 ] },
          " - "
        ]
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
    "shopAndLocation": [
      "Beverage Shop",
      "Satterfieldmouth"
    ]
  }
]
```

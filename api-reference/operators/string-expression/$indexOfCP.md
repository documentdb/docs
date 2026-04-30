---
title: $indexOfCP
description: The $indexOfCP operator searches a string for the first occurrence of a substring and returns the UTF-8 code point index.
type: operators
category: string-expression
---

# $indexOfCP

The `$indexOfCP` operator searches a string for the first occurrence of a substring and returns the UTF-8 code point index of the first occurrence. If the substring is not found, it returns `-1`.

## Syntax

```javascript
{
  $indexOfCP: [ <string>, <substring>, <start>, <end> ]
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<string>`** | The string to search. Can be any expression that resolves to a string. |
| **`<substring>`** | The substring to search for within the string. Can be any expression that resolves to a string. |
| **`<start>`** | Optional. An integer that specifies the starting code point index position for the search. Defaults to `0`. |
| **`<end>`** | Optional. An integer that specifies the ending code point index position for the search. If specified, the search stops before this index. |

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

### Example 1 - Find the code point index of a substring

Use `$indexOfCP` to find the code point position of "Beverage" in the store name.

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      beveragePosition: {
        $indexOfCP: [ "$name", "Beverage" ]
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
    "beveragePosition": 23
  }
]
```

### Example 2 - Search for "Shop" in the store name with a start index

Use `$indexOfCP` with a start index to find the position of "Shop" starting from code point 20.

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      shopPosition: {
        $indexOfCP: [ "$name", "Shop", 20 ]
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
    "shopPosition": 32
  }
]
```

---
title: $substrBytes
description: The $substrBytes operator returns a substring of a string based on the specified UTF-8 byte index and length.
type: operators
category: string-expression
---

# $substrBytes

The `$substrBytes` operator returns a substring of a string, starting at a specified UTF-8 byte index position for a specified number of bytes. The index is zero-based.

## Syntax

```javascript
{
  $substrBytes: [ <string>, <start>, <length> ]
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<string>`** | The string from which to extract the substring. Can be any expression that resolves to a string. |
| **`<start>`** | The zero-based byte index position at which to start the substring. Can be any expression that resolves to a non-negative integer. |
| **`<length>`** | The number of bytes to include in the substring. Can be any expression that resolves to a non-negative integer or number. |

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

### Example 1 - Extract the company name prefix

Use `$substrBytes` to extract the first 22 bytes of the store name, which corresponds to the company name portion.

```javascript
db.stores.aggregate([
  {
    $project: {
      companyName: {
        $substrBytes: [ "$name", 0, 22 ]
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
    "companyName": "First Up Consultants |"
  }
]
```

### Example 2 - Extract a portion of each event name

Use `$substrBytes` within a `$map` to extract the first 6 bytes of each promotion event name.

```javascript
db.stores.aggregate([
  {
    $project: {
      eventPrefixes: {
        $map: {
          input: "$promotionEvents",
          as: "event",
          in: {
            prefix: {
              $substrBytes: [ "$$event.eventName", 0, 6 ]
            },
            eventName: "$$event.eventName"
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
    "eventPrefixes": [
      {
        "prefix": "Summer",
        "eventName": "Summer Sizzler"
      },
      {
        "prefix": "Oktofe",
        "eventName": "Oktoberfest Specials"
      }
    ]
  }
]
```

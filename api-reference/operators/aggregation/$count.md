---
title: $count
description: The $count stage in the aggregation pipeline counts the number of documents entering the stage and outputs a document with the count.
type: operators
category: aggregation
---

# $count

The `$count` stage in the aggregation pipeline counts the number of documents that enter the stage and outputs a single document containing the count. This stage is useful for getting the total number of documents that match preceding pipeline stages, such as `$match` or `$group`.

## Syntax

```javascript
{
  $count: <string>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`string`** | The name of the output field that will contain the count. The field name must be a non-empty string and must not start with `$`. |

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
        "totalSales": 75670,
        "salesByCategory": [
            {
                "categoryName": "Wine Accessories",
                "totalSales": 34440
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
            "eventName": "Unbeatable Bargain Bash",
            "promotionalDates": {
                "startDate": {
                    "Year": 2024,
                    "Month": 6,
                    "Day": 23
                },
                "endDate": {
                    "Year": 2024,
                    "Month": 7,
                    "Day": 2
                }
            },
            "discounts": [
                {
                    "categoryName": "Whiskey",
                    "discountPercentage": 7
                },
                {
                    "categoryName": "Bitters",
                    "discountPercentage": 15
                },
                {
                    "categoryName": "Brandy",
                    "discountPercentage": 8
                },
                {
                    "categoryName": "Sports Drinks",
                    "discountPercentage": 22
                },
                {
                    "categoryName": "Vodka",
                    "discountPercentage": 19
                }
            ]
        },
        {
            "eventName": "Steal of a Deal Days",
            "promotionalDates": {
                "startDate": {
                    "Year": 2024,
                    "Month": 9,
                    "Day": 21
                },
                "endDate": {
                    "Year": 2024,
                    "Month": 9,
                    "Day": 29
                }
            },
            "discounts": [
                {
                    "categoryName": "Organic Wine",
                    "discountPercentage": 19
                },
                {
                    "categoryName": "White Wine",
                    "discountPercentage": 20
                },
                {
                    "categoryName": "Sparkling Wine",
                    "discountPercentage": 19
                },
                {
                    "categoryName": "Whiskey",
                    "discountPercentage": 17
                },
                {
                    "categoryName": "Vodka",
                    "discountPercentage": 23
                }
            ]
        }
    ]
}
```

### Example 1: Counting all documents in a collection

To count the total number of documents in the stores collection:

```javascript
db.stores.aggregate([
  {
    $count: "totalStores"
  }
])
```

This query returns the following result:

```json
[
  {
    "totalStores": 41506
  }
]
```

### Example 2: Counting documents that match a condition

To count the number of stores with total sales greater than 50000:

```javascript
db.stores.aggregate([
  {
    $match: {
      "sales.totalSales": { $gt: 50000 }
    }
  },
  {
    $count: "highSalesStores"
  }
])
```

This query returns the following result:

```json
[
  {
    "highSalesStores": 8924
  }
]
```

### Example 3: Counting documents after unwinding an array

To count the total number of sales categories across all stores:

```javascript
db.stores.aggregate([
  {
    $unwind: "$sales.salesByCategory"
  },
  {
    $count: "totalCategories"
  }
])
```

This query returns the following result:

```json
[
  {
    "totalCategories": 62150
  }
]
```

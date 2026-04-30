---
title: $push
description: The $push accumulator operator returns an array of values for each group.
type: operators
category: accumulators
---

# $push

The `$push` accumulator operator returns an array of all values that result from applying an expression to each document in a group. Unlike `$addToSet`, `$push` does not remove duplicate values and preserves the order of elements.

## Syntax

```javascript
{
  $push: <expression>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<expression>`** | The field or expression to evaluate for each document in the group. All results, including duplicates, are collected into an array. |

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

### Example 1: Using `$push` in `$group` to collect all category sales

This query unwinds the `salesByCategory` array and groups by store name, collecting all category sales figures into an array.

```javascript
db.stores.aggregate([
    {
        $unwind: "$sales.salesByCategory"
    },
    {
        $group: {
            _id: "$name",
            allCategorySales: {
                $push: {
                    category: "$sales.salesByCategory.categoryName",
                    sales: "$sales.salesByCategory.totalSales"
                }
            }
        }
    },
    {
        $limit: 3
    }
])
```

This query returns the following results:

```json
[
    {
        "_id": "First Up Consultants | Beverage Shop - Satterfieldmouth",
        "allCategorySales": [
            {
                "category": "Wine Accessories",
                "sales": 34450
            },
            {
                "category": "Bitters",
                "sales": 39496
            },
            {
                "category": "Rum",
                "sales": 1734
            }
        ]
    }
]
```

### Example 2: Using `$push` to collect promotion event details

This query groups all stores and pushes all promotion event names and discount percentages into an array per store.

```javascript
db.stores.aggregate([
    {
        $unwind: "$promotionEvents"
    },
    {
        $group: {
            _id: "$name",
            promotions: {
                $push: {
                    event: "$promotionEvents.eventName",
                    discount: "$promotionEvents.discountPercentage"
                }
            }
        }
    },
    {
        $limit: 3
    }
])
```

This query returns the following results:

```json
[
    {
        "_id": "First Up Consultants | Beverage Shop - Satterfieldmouth",
        "promotions": [
            {
                "event": "Summer Sizzler",
                "discount": 15
            },
            {
                "event": "Oktoberfest Specials",
                "discount": 20
            }
        ]
    }
]
```

### Example 3: Using `$push` to collect values including duplicates

This query collects all discount percentages across all stores. Unlike `$addToSet`, duplicate values are preserved.

```javascript
db.stores.aggregate([
    {
        $unwind: "$promotionEvents"
    },
    {
        $group: {
            _id: null,
            allDiscounts: {
                $push: "$promotionEvents.discountPercentage"
            }
        }
    },
    {
        $project: {
            _id: 0,
            allDiscounts: 1
        }
    }
])
```

This query returns the following results:

```json
[
    {
        "allDiscounts": [
            15,
            20,
            15,
            10,
            25,
            20
        ]
    }
]
```

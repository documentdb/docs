---
title: $addToSet
description: The $addToSet accumulator operator returns an array of unique values for each group.
type: operators
category: accumulators
---

# $addToSet

The `$addToSet` accumulator operator returns an array of all unique values that result from applying an expression to each document in a group. The order of elements in the output array is undefined. If the value is already present in the set, `$addToSet` does not add a duplicate.

## Syntax

```javascript
{
  $addToSet: <expression>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`<expression>`** | The field or expression to evaluate for each document in the group. The unique results are collected into a set. |

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

### Example 1: Using `$addToSet` in `$group` to collect unique category names

This query unwinds the `salesByCategory` array and groups by store name, collecting all unique category names sold at each store.

```javascript
db.stores.aggregate([
    {
        $unwind: "$sales.salesByCategory"
    },
    {
        $group: {
            _id: "$name",
            uniqueCategories: {
                $addToSet: "$sales.salesByCategory.categoryName"
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
        "uniqueCategories": [
            "Wine Accessories",
            "Bitters",
            "Rum"
        ]
    }
]
```

### Example 2: Using `$addToSet` to collect unique promotion event names

This query groups all stores and collects the unique promotion event names across documents.

```javascript
db.stores.aggregate([
    {
        $unwind: "$promotionEvents"
    },
    {
        $group: {
            _id: null,
            uniqueEventNames: {
                $addToSet: "$promotionEvents.eventName"
            }
        }
    },
    {
        $project: {
            _id: 0,
            uniqueEventNames: 1
        }
    }
])
```

This query returns the following results:

```json
[
    {
        "uniqueEventNames": [
            "Summer Sizzler",
            "Oktoberfest Specials",
            "Holiday Blowout",
            "Spring Fling Sale"
        ]
    }
]
```

### Example 3: Using `$addToSet` to collect unique discount percentages per store

This query groups by store and collects all unique discount percentages offered across promotion events.

```javascript
db.stores.aggregate([
    {
        $unwind: "$promotionEvents"
    },
    {
        $group: {
            _id: "$name",
            uniqueDiscounts: {
                $addToSet: "$promotionEvents.discountPercentage"
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
        "uniqueDiscounts": [
            15,
            20
        ]
    }
]
```

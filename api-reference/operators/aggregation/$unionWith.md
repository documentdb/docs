---
title: $unionWith
description: The $unionWith stage in the aggregation pipeline performs a union of two collections, combining documents from both into the pipeline.
type: operators
category: aggregation
---

# $unionWith

The `$unionWith` stage in the aggregation pipeline performs a union of two collections, combining documents from both collections into a single result set. It is similar to the SQL `UNION ALL` operation and does not remove duplicate documents. This stage is useful for merging data from multiple collections that share similar structures.

## Syntax

The full syntax with an optional pipeline:

```javascript
{
  $unionWith: {
    coll: <string>,
    pipeline: [ <stage1>, <stage2>, ... ]
  }
}
```

The shorthand syntax when no pipeline is needed:

```javascript
{
  $unionWith: <string>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`coll`** | Required. The name of the collection to perform the union with. |
| **`pipeline`** | Optional. An array of aggregation pipeline stages to apply to the specified collection before performing the union. |

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

Let's say we also have an `archivedStores` collection with similar documents:

```json
{
    "_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Contoso, Ltd. | Vintage Wine Cellar - Oldtown",
    "location": {
        "lat": 40.7128,
        "lon": -74.0060
    },
    "sales": {
        "totalSales": 25300
    }
}
```

### Example 1: Simple union of two collections

To combine all documents from the `stores` and `archivedStores` collections and count the total:

```javascript
db.stores.aggregate([
  {
    $project: { name: 1, totalSales: "$sales.totalSales" }
  },
  {
    $unionWith: {
      coll: "archivedStores",
      pipeline: [
        { $project: { name: 1, totalSales: "$sales.totalSales" } }
      ]
    }
  },
  {
    $count: "totalDocuments"
  }
])
```

This query returns the following result:

```json
[
  {
    "totalDocuments": 41510
  }
]
```

### Example 2: Union with a filtered pipeline

To combine active stores with archived stores that had total sales above 20000, then sort by sales:

```javascript
db.stores.aggregate([
  {
    $project: {
      name: 1,
      totalSales: "$sales.totalSales",
      source: { $literal: "active" }
    }
  },
  {
    $unionWith: {
      coll: "archivedStores",
      pipeline: [
        {
          $match: { "sales.totalSales": { $gt: 20000 } }
        },
        {
          $project: {
            name: 1,
            totalSales: "$sales.totalSales",
            source: { $literal: "archived" }
          }
        }
      ]
    }
  },
  {
    $sort: { totalSales: -1 }
  },
  { $limit: 3 }
])
```

The first three results returned by this query are:

```json
[
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "totalSales": 75670,
    "source": "active"
  },
  {
    "_id": "7954bd5c-9ac2-4c10-bb7a-2b79bd0963c5",
    "name": "Lakeshore Retail | DJ Equipment Stop - Port Cecile",
    "totalSales": 47510,
    "source": "active"
  },
  {
    "_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Contoso, Ltd. | Vintage Wine Cellar - Oldtown",
    "totalSales": 25300,
    "source": "archived"
  }
]
```

### Example 3: Shorthand syntax

To combine all documents from `stores` and `archivedStores` using the shorthand syntax:

```javascript
db.stores.aggregate([
  {
    $unionWith: "archivedStores"
  },
  {
    $group: {
      _id: null,
      totalSalesAcrossAll: { $sum: "$sales.totalSales" },
      totalDocuments: { $count: {} }
    }
  }
])
```

This query returns the following result:

```json
[
  {
    "_id": null,
    "totalSalesAcrossAll": 1524897250,
    "totalDocuments": 41510
  }
]
```

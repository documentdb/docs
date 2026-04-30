---
title: $limit
description: The $limit stage in the aggregation pipeline is used to limit the number of documents passed to the next stage.
type: operators
category: aggregation
---

# $limit

The `$limit` stage in the aggregation pipeline is used to restrict the number of documents passed to the next stage in the pipeline. This stage is useful for returning only a specified number of results, implementing pagination, or optimizing performance by reducing the volume of data processed in subsequent stages.

## Syntax

```javascript
{
  $limit: <positive integer>
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`positive integer`** | The maximum number of documents to pass to the next stage. Must be a positive integer. |

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

### Example 1: Limiting the number of documents

To return only the first 3 documents from the stores collection:

```javascript
db.stores.aggregate([
  { $limit: 3 }
])
```

The first two results returned by this query are:

```json
[
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
    }
  },
  {
    "_id": "7954bd5c-9ac2-4c10-bb7a-2b79bd0963c5",
    "name": "Lakeshore Retail | DJ Equipment Stop - Port Cecile",
    "location": {
      "lat": 45.1234,
      "lon": -93.5678
    },
    "staff": {
      "totalStaff": {
        "fullTime": 5,
        "partTime": 10
      }
    },
    "sales": {
      "totalSales": 47510,
      "salesByCategory": [
        {
          "categoryName": "DJ Speakers",
          "totalSales": 25000
        },
        {
          "categoryName": "Turntables",
          "totalSales": 22510
        }
      ]
    }
  }
]
```

### Example 2: Combining $sort with $limit to find top stores

To find the top 2 stores by total sales:

```javascript
db.stores.aggregate([
  {
    $sort: { "sales.totalSales": -1 }
  },
  {
    $limit: 2
  },
  {
    $project: {
      name: 1,
      totalSales: "$sales.totalSales"
    }
  }
])
```

The results returned by this query are:

```json
[
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "totalSales": 75670
  },
  {
    "_id": "7954bd5c-9ac2-4c10-bb7a-2b79bd0963c5",
    "name": "Lakeshore Retail | DJ Equipment Stop - Port Cecile",
    "totalSales": 47510
  }
]
```

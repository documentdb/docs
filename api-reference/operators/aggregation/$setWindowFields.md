---
title: $setWindowFields
description: The $setWindowFields stage in the aggregation pipeline performs window function calculations across a specified partition of documents.
type: operators
category: aggregation
---

# $setWindowFields

The `$setWindowFields` stage in the aggregation pipeline performs window function calculations across a partition of documents. It allows you to compute values such as running totals, moving averages, rankings, and other window-based operations without collapsing the documents into groups. Each document retains its original structure and gains additional computed fields.

## Syntax

```javascript
{
  $setWindowFields: {
    partitionBy: <expression>,
    sortBy: {
      <field1>: <sort order>,
      <field2>: <sort order>,
      ...
    },
    output: {
      <outputField1>: {
        <windowFunction>: <expression>,
        window: {
          documents: [ <lower boundary>, <upper boundary> ],
          range: [ <lower boundary>, <upper boundary> ],
          unit: <time unit>
        }
      },
      ...
    }
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`partitionBy`** | Optional. The expression to partition the documents by. If omitted, all documents are treated as a single partition. |
| **`sortBy`** | Optional. The field(s) to sort by within each partition. Specify `1` for ascending or `-1` for descending order. Required for certain window functions such as `$rank` and `$denseRank`. |
| **`output`** | Required. An object specifying the output fields and the window functions to compute. Each output field contains a window function expression and an optional `window` specification. |
| **`window`** | Optional. Defines the range of documents to include in the window calculation. Can use `documents` for position-based or `range` for value-based boundaries. |
| **`documents`** | A position-based window specified as `[lower, upper]` where values are relative to the current document. Use `"unbounded"` for the start or end of the partition. |
| **`range`** | A value-based window specified as `[lower, upper]` relative to the current document's `sortBy` field value. Use `"unbounded"` for the start or end of the partition. |

For the list of available window functions, see the [window operators documentation](../window-operators/).

## Limitations

- `$setWindowFields` does not support collation in DocumentDB `v0.110-0`.
- In DocumentDB `v0.110-0`, you can enable optimized `$min` and `$max` accumulators in `$setWindowFields` by setting the `enableNewMinMaxAccumulators` GUC parameter.

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

### Example 1: Ranking stores by total sales

To rank each store by its total sales in descending order:

```javascript
db.stores.aggregate([
  {
    $setWindowFields: {
      sortBy: { "sales.totalSales": -1 },
      output: {
        salesRank: {
          $rank: {}
        }
      }
    }
  },
  {
    $project: {
      name: 1,
      totalSales: "$sales.totalSales",
      salesRank: 1
    }
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
    "salesRank": 1
  },
  {
    "_id": "7954bd5c-9ac2-4c10-bb7a-2b79bd0963c5",
    "name": "Lakeshore Retail | DJ Equipment Stop - Port Cecile",
    "totalSales": 47510,
    "salesRank": 2
  },
  {
    "_id": "2cf3f885-9962-4b67-a172-aa9039e9ae2f",
    "name": "First Up Consultants | Bed and Bath Center - South Amir",
    "totalSales": 37701,
    "salesRank": 3
  }
]
```

### Example 2: Computing a running total of sales within a partition

To compute a cumulative sum of total sales for each store, partitioned by company name and sorted by total sales:

```javascript
db.stores.aggregate([
  {
    $setWindowFields: {
      partitionBy: "$company",
      sortBy: { "sales.totalSales": 1 },
      output: {
        cumulativeSales: {
          $sum: "$sales.totalSales",
          window: {
            documents: ["unbounded", "current"]
          }
        }
      }
    }
  },
  {
    $project: {
      name: 1,
      company: 1,
      totalSales: "$sales.totalSales",
      cumulativeSales: 1
    }
  },
  { $limit: 2 }
])
```

The first two results returned by this query are:

```json
[
  {
    "_id": "2cf3f885-9962-4b67-a172-aa9039e9ae2f",
    "name": "First Up Consultants | Bed and Bath Center - South Amir",
    "company": "First Up Consultants",
    "totalSales": 37701,
    "cumulativeSales": 37701
  },
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "company": "First Up Consultants",
    "totalSales": 75670,
    "cumulativeSales": 113371
  }
]
```

### Example 3: Computing a moving average over a document window

To compute a moving average of total sales over a sliding window of the current document and its two nearest neighbors:

```javascript
db.stores.aggregate([
  {
    $setWindowFields: {
      sortBy: { "sales.totalSales": 1 },
      output: {
        movingAvgSales: {
          $avg: "$sales.totalSales",
          window: {
            documents: [-1, 1]
          }
        }
      }
    }
  },
  {
    $project: {
      name: 1,
      totalSales: "$sales.totalSales",
      movingAvgSales: { $round: ["$movingAvgSales", 2] }
    }
  },
  { $limit: 2 }
])
```

The first two results returned by this query are:

```json
[
  {
    "_id": "44fdb9b9-df83-4492-8f71-b6ef648aa312",
    "name": "Fourth Coffee | Storage Solution Gallery - Port Camilla",
    "totalSales": 2236,
    "movingAvgSales": 4872.5
  },
  {
    "_id": "728c068a-638c-40af-9172-8ccfa7dddb49",
    "name": "Contoso, Ltd. | Book Store - Lake Myron",
    "totalSales": 7509,
    "movingAvgSales": 5148.33
  }
]
```

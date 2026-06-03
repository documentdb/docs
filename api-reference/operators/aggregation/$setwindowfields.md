---
title: $setWindowFields
description: The $setWindowFields stage performs operations on a specified span of documents (a window) and returns results based on the chosen window operator.
type: operators
category: aggregation
---

# $setWindowFields

The `$setWindowFields` stage groups documents into partitions, applies window functions over a defined span of documents within each partition, and outputs a new field with the result for each document. This is useful for running calculations (like running averages, ranks, and cumulative sums) without collapsing documents into groups.

## Syntax

```javascript
{
  $setWindowFields: {
    partitionBy: <expression>,
    sortBy: {
      <field1>: <1 or -1>,
      ...
    },
    output: {
      <outputField1>: {
        <windowOperator>: <specification>,
        window: {
          documents: [<lower>, <upper>],
          range: [<lower>, <upper>],
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
| **`partitionBy`** | Optional. An expression to group documents into partitions. Similar to `$group`'s `_id`. If omitted, all documents belong to a single partition. |
| **`sortBy`** | Optional. A document specifying the sort order within each partition. Each field value must be `1` (ascending) or `-1` (descending). |
| **`output`** | Required. A document with one or more fields. Each field specifies a window operator and optionally a window frame. |

### Window Frame Options

| Option | Description |
| --- | --- |
| **`documents`** | Row-based window bounds. Array of `[lower, upper]` where values can be `"unbounded"`, `"current"`, or an integer offset. |
| **`range`** | Range-based window bounds. Array of `[lower, upper]` using sort key values. |
| **`unit`** | Time unit for range-based windows. Values: `"year"`, `"quarter"`, `"month"`, `"week"`, `"day"`, `"hour"`, `"minute"`, `"second"`, `"millisecond"`. |

### Supported Window Operators

`$addToSet`, `$avg`, `$bottom`, `$bottomN`, `$count`, `$covariancePop`, `$covarianceSamp`, `$denseRank`, `$derivative`, `$documentNumber`, `$expMovingAvg`, `$first`, `$firstN`, `$integral`, `$last`, `$lastN`, `$linearFill`, `$locf`, `$max`, `$maxN`, `$min`, `$minN`, `$push`, `$rank`, `$shift`, `$stdDevPop`, `$stdDevSamp`, `$sum`, `$top`, `$topN`

## Examples

Consider this sample document from the stores collection.

```json
{
  "_id": "2cf3f885-9962-4b67-a172-aa9039e9ae2f",
  "name": "First Up Consultants | Bed and Bath Center - South Amir",
  "city": "South Amir",
  "staff": {
    "totalStaff": {
      "fullTime": 18,
      "partTime": 17
    }
  },
  "sales": {
    "totalSales": 37701
  }
}
```

### Example 1: Rank stores by sales

Rank all stores by total sales:

```javascript
db.stores.aggregate([
  {
    $setWindowFields: {
      sortBy: { "sales.totalSales": -1 },
      output: {
        salesRank: { $rank: {} }
      }
    }
  },
  { $project: { name: 1, "sales.totalSales": 1, salesRank: 1 } },
  { $limit: 3 }
])
```

### Example 2: Running average partitioned by city

Calculate a running average of sales within each city, using a 3-document window:

```javascript
db.stores.aggregate([
  {
    $setWindowFields: {
      partitionBy: "$city",
      sortBy: { "sales.totalSales": 1 },
      output: {
        movingAvgSales: {
          $avg: "$sales.totalSales",
          window: { documents: [-1, 1] }
        }
      }
    }
  },
  { $project: { name: 1, city: 1, "sales.totalSales": 1, movingAvgSales: 1 } },
  { $limit: 3 }
])
```

### Example 3: Cumulative sum

Calculate a cumulative sum of sales ordered by total sales:

```javascript
db.stores.aggregate([
  {
    $setWindowFields: {
      sortBy: { "sales.totalSales": 1 },
      output: {
        cumulativeSales: {
          $sum: "$sales.totalSales",
          window: { documents: ["unbounded", "current"] }
        }
      }
    }
  },
  { $project: { name: 1, "sales.totalSales": 1, cumulativeSales: 1 } },
  { $limit: 3 }
])
```

### Example 4: Document number

Assign a sequential number to each document:

```javascript
db.stores.aggregate([
  {
    $setWindowFields: {
      sortBy: { name: 1 },
      output: {
        docNumber: { $documentNumber: {} }
      }
    }
  },
  { $project: { name: 1, docNumber: 1 } },
  { $limit: 3 }
])
```

## Limitations

- Collation is not supported in `$setWindowFields`
- `$median` and `$percentile` window operators are not yet supported
- Cannot use both `documents` and `range` in the same window specification

## Key Takeaways

- **Non-destructive** — unlike `$group`, `$setWindowFields` adds computed fields to each document without collapsing rows
- **Partitioning** — use `partitionBy` to calculate window functions within groups independently
- **Flexible windows** — use `documents` for row-based windows or `range` for value-based windows
- **Ranking** — `$rank`, `$denseRank`, and `$documentNumber` are commonly used for ordering and pagination

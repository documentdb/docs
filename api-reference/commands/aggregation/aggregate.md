---
title: Aggregate
description: The aggregate command is used to process data records and return computed results.
type: commands
category: aggregation
---

# aggregate

The `aggregate` command is used to process data records and return computed results. It performs operations on the data, such as filtering, grouping, and sorting, and can transform the data in various ways. The `aggregate` command is highly versatile and is commonly used for data analysis and reporting.

## Parameters

| Parameter | Description |
| --- | --- |
| **`aggregate`** | (Required) A string or number specifying the collection name, or `1` for a collection-independent pipeline. |
| **`pipeline`** | (Required) An array of aggregation-stage documents. |
| **`cursor`** | (Required unless explaining the command) A document that accepts `batchSize` and the DocumentDB extension `singleBatch`. |
| **`hint`** | (Optional) A string or document specifying the index name or key pattern for the input collection. |
| **`let`** | (Optional) A document specifying the command-scoped variables referenced as `$$name`. |
| **`maxTimeMS`** | (Optional) A number specifying the execution timeout in milliseconds. |
| **`collation`** | (Optional) A document specifying the comparison rules; requires `documentdb_core.enableCollation`. Rejected when that capability is disabled. |
| **`collectionUUID`** | (Optional) A BSON UUID that must match the input collection's UUID; a mismatch fails the command. |
| **`explain`** | (Optional) A boolean that requests an explanation instead of normal execution. The [`explain`](../diagnostic/explain.md) wrapper provides verbosity selection. |
| **`cursor.batchSize`** | (Optional) A number limiting the initial batch, converted to an integer that must be non-negative after conversion. Defaults to `101`; `0` returns an empty first batch. Response size limits still apply. |
| **`cursor.singleBatch`** | (Optional) A boolean that closes the cursor after the initial batch when `true`. |

`cursor.batchSize` is converted to an integer and must be non-negative after conversion. The initial batch defaults to `101`; `0` returns an empty first batch. `cursor.singleBatch: true` closes the cursor after that batch. Response size limits still apply.

## Example: summarize departures by airport

Load the [airline example dataset](../diagnostic/wire-protocol.md#airline-example-dataset), then run:

```javascript
db.flights.aggregate(
  [
    { $group: {
      _id: "$departure_airport",
      departures: { $sum: 1 },
      total_seats: { $sum: "$seat_capacity" }
    } },
    { $sort: { _id: 1 } }
  ],
  { batchSize: 10, maxTimeMS: 5000 }
).toArray()
```

The returned array is:

```json
[
  { "_id": "LED", "departures": 1, "total_seats": 100 },
  { "_id": "SVO", "departures": 3, "total_seats": 440 }
]
```

The shell cursor retrieves additional batches automatically. The [`getMore`](../query-and-write/getMore.md) reference describes the underlying continuation request.

## Shell syntax

```javascript
db.collection.aggregate(pipeline, options)
```

## Detailed Examples

### Example 1: Calculate total sales by category

This example demonstrates how to calculate the total sales for each category in the `stores` collection.

```javascript
db.stores.aggregate([
  {
    $unwind: "$sales.salesByCategory"
  },
  {
    $group: {
      _id: "$sales.salesByCategory.categoryName",
      totalSales: { $sum: "$sales.salesByCategory.totalSales" }
    }
  }
])
```

#### Sample output

```json
[
  { "_id": "Christmas Trees", "totalSales": 3147281 },
  { "_id": "Nuts", "totalSales": 3002332 },
  { "_id": "Camping Tables", "totalSales": 4431667 }
]
```

### Example 2: Find stores with full-time staff greater than 10

This example shows how to filter stores where the number of full-time staff is greater than 10.

```javascript
db.stores.aggregate([
  {
    $match: {
      "staff.totalStaff.fullTime": { $gt: 10 }
    }
  }
])
```

#### Sample output

```json
[
  {
    "_id": "7954bd5c-9ac2-4c10-bb7a-2b79bd0963c5",
    "name": "Lenore's DJ Equipment Store",
    "location": { "lat": -9.9399, "lon": -0.334 },
    "staff": { "totalStaff": { "fullTime": 18, "partTime": 7 } },
    "sales": {
      "totalSales": 35911,
      "salesByCategory": [ { "categoryName": "DJ Headphones", "totalSales": 35911 } ]
    },
    "promotionEvents": [
      {
        "discounts": [
          { "categoryName": "DJ Turntables", "discountPercentage": 18 },
          { "categoryName": "DJ Mixers", "discountPercentage": 15 }
        ]
      }
    ],
    "tag": [ "#SeasonalSale", "#FreeShipping", "#MembershipDeals" ]
  }
]
```

### Example 3: List all promotion events with discounts greater than 15%

This example lists all promotion events where any discount is greater than 15%.

```javascript
db.stores.aggregate([
  {
    $unwind: "$promotionEvents"
  },
  {
    $unwind: "$promotionEvents.discounts"
  },
  {
    $match: {
      "promotionEvents.discounts.discountPercentage": { $gt: 15 }
    }
  },
  {
    $group: {
      _id: "$promotionEvents.eventName",
      discounts: { $push: "$promotionEvents.discounts" }
    }
  }
])
```

#### Sample output

```json
[
  {
    "discounts": [
      { "categoryName": "Basketball Gear", "discountPercentage": 23 },
      { "categoryName": "Wool Carpets", "discountPercentage": 22 },
      {
        "categoryName": "Portable Bluetooth Speakers",
        "discountPercentage": 24
      }
    ]
  }
]
```

## Implementation

Runs a pipeline against a collection or a collection-independent input. The backend parses the pipeline into a PostgreSQL query; the gateway returns results through a cursor.

Source: `pg_documentdb/src/aggregation/bson_aggregation_pipeline.c`, `ParseAggregationQueryAndLookupCollection` and `ParseCursorDocument`. Source revision and shared metadata are documented in [wire protocol and common fields](../diagnostic/wire-protocol.md).

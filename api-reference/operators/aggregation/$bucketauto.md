---
title: $bucketAuto
description: The $bucketAuto stage automatically distributes documents into a specified number of buckets based on a grouping expression.
type: operators
category: aggregation
---

# $bucketAuto

The `$bucketAuto` stage categorizes documents into a specified number of buckets, attempting to evenly distribute the documents based on the values of a `groupBy` expression. Unlike [`$bucket`](https://documentdb.io/docs/reference/operators/aggregation/%24bucket/), you do not have to provide boundaries — DocumentDB computes them for you.

Supported since `v0.105-0`.

## Syntax

```javascript
{
  $bucketAuto: {
    groupBy: <expression>,
    buckets: <number>,
    output: {
      <outputField1>: { <accumulator1> },
      ...
    },
    granularity: <granularity-string>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`groupBy`** | An expression to group documents by. The expression must resolve to a value that can be compared. |
| **`buckets`** | A positive 32-bit integer that specifies the number of buckets into which the input documents should be grouped. |
| **`output`** | Optional. A document that specifies the fields to compute for each bucket. Each field is an accumulator expression such as `$sum`, `$avg`, `$min`, `$max`, `$push`, or `$addToSet`. If omitted, `$bucketAuto` outputs a `count` field by default. |
| **`granularity`** | Optional. A string that specifies the [preferred number series](https://en.wikipedia.org/wiki/Preferred_number) used to round the bucket boundaries. Supported values: `POWERSOF2`, `1-2-5`, `R5`, `R10`, `R20`, `R40`, `R80`, `E6`, `E12`, `E24`, `E48`, `E96`, `E192`. The `groupBy` values must be numeric and non-negative when `granularity` is used. |

## Behavior

- `$bucketAuto` outputs documents with an `_id` of the form `{ "min": <lower>, "max": <upper> }`, representing the bucket's lower and upper boundary. The upper boundary is exclusive for all buckets except the last, which includes its upper boundary.
- Without `granularity`, a non-last bucket's `max` is the first `groupBy` value of the *next* bucket, so adjacent buckets share a boundary. The last bucket's `max` is its own largest value.
- Documents are distributed as evenly as the input allows: with `n` documents and `b` buckets each bucket takes `floor(n / b)` documents, and the first `n mod b` buckets take one extra. A bucket is then extended to absorb any following documents that tie with its largest value, so that equal values never straddle a boundary.
- The stage can produce fewer buckets than requested — when the number of distinct `groupBy` values is less than `buckets`, and also whenever `granularity` rounding absorbs documents (see below).
- When `granularity` is specified, the first bucket's `min` is rounded down to the nearest series value strictly below it, and every bucket's `max` is rounded up to the nearest series value strictly above it; each later bucket's `min` is simply the previous bucket's `max`. Because a rounded-up `max` can exceed values that were assigned to later buckets, those documents are pulled into the current bucket, which is why the result often has fewer buckets than requested. Boundaries produced this way are doubles.
- With `granularity`, every `groupBy` value must be numeric and non-negative; a non-numeric value fails with `$bucketAuto only allows specifying a 'granularity' with numeric boundaries`.

## Examples

Consider this sample `sales` collection:

```json
[
  { "_id": 1, "category": "Books",       "price":   8 },
  { "_id": 2, "category": "Books",       "price":  12 },
  { "_id": 3, "category": "Electronics", "price":  45 },
  { "_id": 4, "category": "Electronics", "price": 230 },
  { "_id": 5, "category": "Toys",        "price":   3 },
  { "_id": 6, "category": "Toys",        "price":  18 },
  { "_id": 7, "category": "Clothing",    "price":  35 },
  { "_id": 8, "category": "Clothing",    "price":  60 }
]
```

### Example 1: Three even price buckets

Group the sales into three buckets that contain roughly the same number of documents, and compute a count and average price per bucket:

```javascript
db.sales.aggregate([
  {
    $bucketAuto: {
      groupBy: "$price",
      buckets: 3,
      output: {
        count: { $sum: 1 },
        avgPrice: { $avg: "$price" }
      }
    }
  }
])
```

Sample output:

```json
[
  { "_id": { "min": 3,  "max": 18  }, "count": 3, "avgPrice": 7.666666666666667 },
  { "_id": { "min": 18, "max": 60  }, "count": 3, "avgPrice": 32.666666666666664 },
  { "_id": { "min": 60, "max": 230 }, "count": 2, "avgPrice": 145 }
]
```

Eight documents into three buckets gives sizes 3, 3, 2. The buckets hold prices `3, 8, 12`, then `18, 35, 45`, then `60, 230`. Each non-last bucket reports the next bucket's first price as its `max`, so the first bucket ends at `18` and the second at `60`.

### Example 2: Buckets with rounded boundaries via `granularity`

Request four buckets rounded to a power-of-two series:

```javascript
db.sales.aggregate([
  {
    $bucketAuto: {
      groupBy: "$price",
      buckets: 4,
      granularity: "POWERSOF2"
    }
  }
])
```

Sample output:

```json
[
  { "_id": { "min": 2,  "max": 16  }, "count": 3 },
  { "_id": { "min": 16, "max": 64  }, "count": 4 },
  { "_id": { "min": 64, "max": 256 }, "count": 1 }
]
```

Four buckets were requested but three are returned. The even split would have put `3, 8` in the first bucket, but rounding its `max` up from `8` to `16` pulls in `12` as well. The second bucket starts at `18`, and rounding its `max` up from `35` to `64` absorbs `45` and `60`, leaving only `230` for the third bucket.

## See Also

- [`$bucket`](https://documentdb.io/docs/reference/operators/aggregation/%24bucket/) — fixed-boundary bucketing.
- [`$group`](https://documentdb.io/docs/reference/operators/aggregation/%24group/) — generic grouping by an expression.

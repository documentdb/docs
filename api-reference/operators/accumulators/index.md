# Accumulator Operators

Accumulator operators maintain their state as documents progress through the pipeline, allowing for operations that require tracking data across multiple documents.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$bottom](bottom.md) | Returns the bottom element within a group according to the specified sort order. |
| [$bottomN](bottomN.md) | Returns an array of the bottom N elements within a group according to the specified sort order. |
| [$first](first.md) | Returns the first value in a group. |
| [$firstN](firstN.md) | Returns an array of the first N values in a group. |
| [$last](last.md) | Returns the last value in a group. |
| [$lastN](lastN.md) | Returns an array of the last N values in a group. |
| [$top](top.md) | Returns the top element within a group according to the specified sort order. |
| [$topN](topN.md) | Returns an array of the top N elements within a group according to the specified sort order. |

## Usage in Aggregation Pipeline

Accumulator operators can be used in the following stages:
- `$group`
- `$bucket`
- `$bucketAuto`
- `$setWindowFields`

## Example

```javascript
db.sales.aggregate([
  {
    $group: {
      _id: "$item",
      avgPrice: { $avg: "$price" },
      firstSale: { $first: "$date" },
      lastSale: { $last: "$date" },
      topThreePrices: { 
        $topN: {
          n: 3,
          sortBy: { price: -1 }
        }
      }
    }
  }
])
```

## Notes

- Accumulator operators maintain their state (e.g., running totals, extremes) throughout the aggregation pipeline stage.
- When used in a `$group` stage, they accumulate values from multiple documents.
- When used in a `$project` or `$addFields` stage without a `$group`, they accumulate values from array elements.
- Some operators like `$topN` and `$bottomN` require sorting criteria to determine order. 
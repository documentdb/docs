# $count

Returns a count of the number of documents at the current stage of the aggregation pipeline.

## Syntax

```javascript
{ $count: <string> }
```

The `<string>` parameter specifies the name of the output field that will contain the count value.

## Description

The `$count` stage creates a new document with a single field containing the count of documents at that stage in the pipeline. It is equivalent to, but more efficient than, using `$group` with a `$sum` expression.

## Examples

### Basic Count

```javascript
db.inventory.aggregate([
  { $match: { status: "active" } },
  { $count: "activeCount" }
])
```
Result:
```javascript
{ "activeCount": 123 }
```

### Count with Multiple Pipeline Stages

```javascript
db.sales.aggregate([
  { $match: { date: { $gte: new Date('2024-01-01') } } },
  { $group: { _id: "$product" } },
  { $count: "uniqueProducts" }
])
```

### Using Count in Facets

```javascript
db.orders.aggregate([
  {
    $facet: {
      "categories": [
        { $group: { _id: "$category" } },
        { $count: "totalCategories" }
      ],
      "statuses": [
        { $group: { _id: "$status" } },
        { $count: "totalStatuses" }
      ]
    }
  }
])
```

### Count with Complex Conditions

```javascript
db.inventory.aggregate([
  {
    $match: {
      $and: [
        { price: { $gt: 100 } },
        { quantity: { $gt: 0 } },
        { "reviews.rating": { $gte: 4 } }
      ]
    }
  },
  { $count: "premiumInStock" }
])
```

## Behavior

- Returns a single document with one field
- The count field name must be a non-empty string
- The count field name cannot start with '$'
- Returns an empty array if there are no matching documents
- Cannot be used with `$out` or `$merge` stages

## Use Cases

- Getting total number of matching documents
- Counting unique values
- Validating data consistency
- Generating statistics and reports
- Monitoring system metrics

## Performance Considerations

1. **Index Usage**
   - `$count` benefits from indexes used in preceding `$match` stages
   - Place `$match` before `$count` for better performance

2. **Memory Usage**
   - Very memory efficient as it only needs to maintain a counter
   - Does not store documents in memory

3. **Optimization**
   - More efficient than `{ $group: { _id: null, count: { $sum: 1 } } }`
   - Can use index count optimization for simple queries

## Related Operators

- [$group](../accumulators/group.md) - Groups documents by specified expression
- [$match](match.md) - Filters documents in the pipeline
- [$facet](facet.md) - Processes multiple aggregation pipelines 
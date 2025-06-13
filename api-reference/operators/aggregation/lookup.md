# $lookup

Performs a left outer join with another collection in the same database to filter in documents from the "joined" collection for processing.

## Syntax

### Simple Lookup (Equality Match)
```javascript
{
  $lookup: {
    from: <collection>,
    localField: <field>,
    foreignField: <field>,
    as: <output array field>
  }
}
```

### Advanced Lookup (Pipeline)
```javascript
{
  $lookup: {
    from: <collection>,
    let: { <var1>: <expression>, ... },
    pipeline: [ <pipeline stages> ],
    as: <output array field>
  }
}
```

## Parameters

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `from` | string | Target collection for the join | Yes |
| `localField` | string | Field from input documents | Yes* |
| `foreignField` | string | Field from documents of the "from" collection | Yes* |
| `let` | document | Variables for use in the pipeline | No |
| `pipeline` | array | Pipeline to run on the joined collection | No |
| `as` | string | Output array field | Yes |

*Required for equality match syntax only

## Description

The `$lookup` stage performs a left outer join to another collection in the same database. For each input document, it adds a new array field containing the matching documents from the "joined" collection.

## Examples

### Simple Equality Match

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customerInfo"
    }
  }
])
```

### Advanced Pipeline Lookup

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "inventory",
      let: { order_item: "$item", order_qty: "$quantity" },
      pipeline: [
        {
          $match: {
            $expr: {
              $and: [
                { $eq: ["$item", "$$order_item"] },
                { $gte: ["$instock", "$$order_qty"] }
              ]
            }
          }
        },
        {
          $project: {
            item: 1,
            stockLocation: 1,
            instock: 1
          }
        }
      ],
      as: "inventoryDocs"
    }
  }
])
```

### Multiple Joins with Uncorrelated Subqueries

```javascript
db.restaurants.aggregate([
  {
    $lookup: {
      from: "reviews",
      pipeline: [
        { $match: { rating: { $gte: 4 } } },
        { $group: {
            _id: "$restaurantId",
            avgRating: { $avg: "$rating" },
            reviewCount: { $sum: 1 }
          }
        }
      ],
      as: "highRatings"
    }
  },
  {
    $lookup: {
      from: "menu",
      localField: "_id",
      foreignField: "restaurantId",
      as: "menuItems"
    }
  },
  {
    $project: {
      name: 1,
      cuisine: 1,
      avgRating: { $arrayElemAt: ["$highRatings.avgRating", 0] },
      reviewCount: { $arrayElemAt: ["$highRatings.reviewCount", 0] },
      menuCount: { $size: "$menuItems" }
    }
  }
])
```

### Correlated Subquery with Date Range

```javascript
db.sales.aggregate([
  {
    $lookup: {
      from: "events",
      let: { sale_date: "$date", store_id: "$storeId" },
      pipeline: [
        {
          $match: {
            $expr: {
              $and: [
                { $eq: ["$storeId", "$$store_id"] },
                { $gte: ["$date", { $dateSubtract: { startDate: "$$sale_date", unit: "day", amount: 7 } }] },
                { $lte: ["$date", "$$sale_date"] }
              ]
            }
          }
        },
        {
          $sort: { date: -1 }
        }
      ],
      as: "recentEvents"
    }
  }
])
```

## Behavior

1. **Join Types**
   - Performs a left outer join
   - Returns all documents from the left side
   - Matches are returned in an array
   - Empty array if no matches found

2. **Performance**
   - Foreign collection is read for each input document
   - Benefits from indexes on join fields
   - Pipeline lookups can use additional indexes

3. **Limitations**
   - Cannot join across databases
   - Maximum 100MB per document result
   - Cannot use in `$facet` stage

## Use Cases

1. **Data Enrichment**
   - Adding customer details to orders
   - Enriching products with category information
   - Including user profiles in activity logs

2. **Complex Relationships**
   - Many-to-many relationships
   - Hierarchical data structures
   - Time-based relationships

3. **Analytics**
   - Sales reporting with product details
   - User activity analysis
   - Inventory tracking

## Performance Considerations

1. **Index Usage**
   - Create indexes on join fields
   - Index foreign collection lookup fields
   - Consider compound indexes for pipeline matches

2. **Memory Management**
   - Monitor document size after join
   - Use projection to limit fields
   - Consider batching for large datasets

3. **Query Optimization**
   - Place `$match` stages before `$lookup`
   - Use pipeline lookups for complex conditions
   - Consider denormalization for frequent joins

## Related Operators

- [$unwind](unwind.md) - Deconstructs an array field
- [$group](group.md) - Groups documents by expression
- [$match](match.md) - Filters documents in pipeline
- [$project](project.md) - Reshapes documents in pipeline 
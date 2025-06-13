# Aggregation Operators

Aggregation operators are used in aggregation pipelines to process documents and return computed results. These operators transform, group, analyze, and perform calculations on data within the pipeline stages.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$count](count.md) | Returns a count of the number of documents in a stage. |
| [$facet](facet.md) | Processes multiple aggregation pipelines within a single stage. |
| [$geoNear](geoNear.md) | Returns documents ordered by their proximity to a geospatial point. |
| [$lookup](lookup.md) | Performs a left outer join with another collection. |
| [$match](match.md) | Filters documents to pass only those that match specified conditions. |

## Pipeline Stages

Aggregation operators are used as stages in an aggregation pipeline. A pipeline can consist of multiple stages, where each stage transforms the documents as they pass through. The stages are processed in sequence, with the output of one stage serving as the input for the next.

## Example Pipeline

```javascript
db.sales.aggregate([
  {
    $match: {
      date: { $gte: new Date('2024-01-01') }
    }
  },
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customerInfo"
    }
  },
  {
    $facet: {
      "byProduct": [
        { $group: { _id: "$product", total: { $sum: "$amount" } } }
      ],
      "byCustomer": [
        { $group: { _id: "$customerInfo.name", count: { $sum: 1 } } }
      ]
    }
  }
])
```

## Common Use Cases

- Data transformation and reshaping
- Complex data aggregations
- Report generation
- Data analysis and statistics
- Geospatial queries
- Document relationships and joins

## Best Practices

1. **Pipeline Order**
   - Place `$match` and `$limit` stages early in the pipeline
   - Use `$project` to reduce document size when possible
   - Consider index usage in early stages

2. **Performance**
   - Use indexes to support early pipeline stages
   - Minimize memory usage with appropriate stage ordering
   - Consider batch processing for large datasets

3. **Memory Limits**
   - Be aware of the 100MB memory limit for pipeline stages
   - Use `allowDiskUse` for large datasets
   - Break complex pipelines into smaller steps

4. **Error Handling**
   - Validate data types before processing
   - Handle null values appropriately
   - Consider using `$facet` for parallel processing 